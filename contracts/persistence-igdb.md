# Persistencia, errores y entornos

Este documento es la revisión IGDB vigente de `GB-002.06` para la línea base
[`GB-002-CONTRACTS-v3`](contract-baseline-v3.md). Define la frontera mínima de Neon/Prisma,
las conexiones runtime y de migración, el contrato de errores y la matriz de
variables por nombre. No contiene valores de conexión, dominios reales,
claves, API keys ni migraciones ejecutables.

## 1. Neon y límites de esquema

Hay una base PostgreSQL de Neon con dos esquemas independientes:

| Esquema | Dueño | Puede leer/escribir | No puede hacer |
| --- | --- | --- | --- |
| `auth` | `GameBook.Microservice.AuthUser` | Sus tablas de cuentas, hashes y versión de sesión. | Crear FK, tabla o migración en `game`; exponer hashes o claves. |
| `game` | `GameBook.Microservice.Game` | Sus tablas de favoritos y plataformas. | Consultar tablas de `auth`, crear FK entre esquemas o aceptar un `userId` del navegador como autoridad. |

Cada repositorio tiene su propio `schema.prisma`, `PrismaClient` y directorio
de migraciones. El `search_path`/esquema por defecto de cada conexión se fija
al esquema del servicio y las credenciales se limitan a ese ámbito. Game guarda
el UUID de `sub` como referencia lógica, no como una relación PostgreSQL hacia
`auth`.

## 2. Forma mínima de datos

### 2.1 Esquema `auth`

`User` es propiedad exclusiva de AuthUser:

| Campo lógico | Tipo mínimo | Regla |
| --- | --- | --- |
| `id` | UUID | Clave primaria y valor de `sub`. |
| `fullName` | texto | Nombre completo validado en frontera HTTP. |
| `email` | texto | Normalizado antes de aplicar índice único. |
| `passwordHash` | texto | Hash con sal; nunca contraseña en claro, respuesta o log. |
| `sessionVersion` | entero positivo | Versión comparada con el claim `ver`; inicia en 1 y aumenta atómicamente al cambiar contraseña. |

El contrato HTTP de estos datos está en
[`authuser.openapi.yaml`](authuser.openapi.yaml). No se añade información de
perfil que el MVP no solicita.

### 2.2 Esquema `game`

`Favorite` pertenece a una cuenta por su identidad lógica `(userId, igdbId)`.
No existen datos de favoritos de aplicación que haya que migrar desde RAWG:
los esquemas Neon preparados aún no contienen tablas de producto.

| Campo lógico | Tipo mínimo | Regla |
| --- | --- | --- |
| `userId` | UUID | Copia del `sub` validado; no es una FK a `auth`. |
| `igdbId` | entero positivo | ID externo IGDB; junto con `userId` es único. |
| `name` | texto | Nombre recibido de IGDB, no traducido. |
| `released` | fecha nullable | Fecha UTC derivada de `first_release_date` o `null`; el año se deriva para filtrar. El modal no presentará una fecha completa como exacta si IGDB solo conoce una fecha parcial. |
| `imageUrl` | URI nullable | URL HTTPS formada desde `cover.image_id` de IGDB o `null`. |
| `rating` | decimal nullable | `total_rating` combinado IGDB en escala 0–100 o `null`. |

`FavoritePlatform` mantiene la forma mínima de plataforma necesaria para las
tarjetas y el filtro:

| Campo lógico | Tipo mínimo | Regla |
| --- | --- | --- |
| `userId` + `igdbId` | clave de favorito | Permite asociar la plataforma al favorito propio. |
| `platformId` | entero positivo | ID IGDB; no se sustituye por el nombre. |
| `name` | texto | Nombre mostrado recibido de IGDB. |

La implementación puede usar una FK interna dentro de `game` o una clave
compuesta equivalente, pero debe preservar unicidad por favorito y plataforma.
No se almacenan descripción, géneros, desarrolladores ni la respuesta completa
de IGDB. La actualización de snapshot reemplaza solo estos campos básicos.

## 3. Conexiones y migraciones

| Uso | AuthUser | Game | Política |
| --- | --- | --- | --- |
| Runtime serverless | `AUTH_DATABASE_URL` | `GAME_DATABASE_URL` | Conexión compatible con Neon/Prisma para funciones serverless; solo el esquema del servicio. |
| Migraciones | `AUTH_DATABASE_DIRECT_URL` | `GAME_DATABASE_DIRECT_URL` | Conexión directa, fuera del pool, usada por el proceso controlado de migración. |

Las variables de runtime y migración son distintas aunque apunten al mismo
entorno lógico. El patrón exacto de driver/adapter y la versión compatible de
Prisma/NestJS se comprobarán en `GB-003`; este contrato no presupone un
proveedor de pool concreto.

Reglas de ejecución:

1. AuthUser aplica únicamente sus migraciones al esquema `auth`; Game aplica
   únicamente las suyas al esquema `game`.
2. Las migraciones se ejecutan antes del despliegue que necesite el cambio,
   mediante el proceso controlado del servicio y su `*_DATABASE_DIRECT_URL`.
3. Ningún build, arranque de Vercel o handler runtime ejecuta migraciones
   implícitas contra producción.
4. Las pruebas de integración usan una base/rama Neon de prueba y esquemas
   aislados; nunca usan producción ni comparten el historial de migraciones.
5. Un rollback de aplicación no ejecuta una migración destructiva automática.

## 4. Contrato uniforme de errores

Ambos microservicios responden JSON con la forma común:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Request validation failed.",
  "requestId": "req_gb206_fictitious",
  "details": []
}
```

`code` es estable y traducible por el frontend; `message` es seguro y no
contiene SQL, stack traces, contraseñas, JWT, API keys ni datos de otra cuenta.
`requestId` y `details` son opcionales según el error.

| HTTP | Códigos estables | Servicios | Significado |
| --- | --- | --- | --- |
| `400` | `VALIDATION_ERROR` | AuthUser, Game | Cuerpo, query, path o rango inválido. |
| `401` | `TOKEN_MISSING`, `TOKEN_INVALID`, `TOKEN_EXPIRED`, `SESSION_REVOKED`, `INVALID_CREDENTIALS` | AuthUser, Game | Bearer ausente/incorrecto, sesión revocada o credenciales no válidas. |
| `404` | `FAVORITE_NOT_FOUND` | Game | El favorito propio no existe; no revela si existe en otra cuenta. |
| `409` | `EMAIL_ALREADY_REGISTERED`, `FAVORITE_ALREADY_EXISTS` | AuthUser, Game | Unicidad violada después de normalización/clave lógica. |
| `503` | `AUTHUSER_UNAVAILABLE` | Game | Game no pudo confirmar la sesión; falla cerrado y no lee ni modifica favoritos. |
| `500` | `INTERNAL_ERROR` | AuthUser, Game | Falla inesperada sin detalles internos. |

Los cuerpos y ejemplos de cada operación están en
[`authuser.openapi.yaml`](authuser.openapi.yaml) y
[`game-igdb.openapi.yaml`](game-igdb.openapi.yaml). Game realiza primero la verificación
local del JWT y después la consulta de sesión de AuthUser descrita en
[`jwt-revocation.md`](jwt-revocation.md).

## 5. URLs, CORS y versiones

Las variables de URL contienen el origen/base sin el sufijo `/v1`; cada cliente
añade la ruta versionada del contrato.

| Flujo | URL base | Exposición |
| --- | --- | --- |
| Navegador → AuthUser | `NEXT_PUBLIC_AUTHUSER_URL` | URL pública del entorno; no es un secreto. |
| Navegador → Game | `NEXT_PUBLIC_GAME_URL` | URL pública del entorno; no es un secreto. |
| Game → AuthUser | `AUTHUSER_URL` | URL configurada en Game para `GET /v1/auth/session`; no se acepta desde el navegador. |
| Next.js → Twitch/IGDB | `https://id.twitch.tv/oauth2/token` y `https://api.igdb.com/v4` | Solo servidor; Client Secret y token de aplicación nunca se reenvían. |

Cada backend usa `CORS_ALLOWED_ORIGINS` con una lista explícita de orígenes del
frontend de ese mismo entorno:

- No se usa `*` en preview ni producción.
- Se permiten `GET`, `POST`, `PATCH`, `DELETE` y `OPTIONS` donde el contrato
  los requiere.
- Se permiten las cabeceras `Authorization` y `Content-Type`, y se expone
  `requestId` si la respuesta lo incluye.
- La autenticación es por Bearer, no por cookies; no se habilitan credenciales
  de navegador de forma global.
- Un origen no incluido recibe rechazo CORS sin consultar la base ni AuthUser.

Las rutas de ambos microservicios usan `/v1`. Swagger UI puede ser público para
consulta, pero sus operaciones protegidas conservan el esquema Bearer y las
validaciones del contrato.

## 6. Matriz de variables por proyecto y entorno

Los mismos nombres se configuran con valores separados para `local`, `preview`
y `production`. Esta matriz no contiene valores reales.

| Proyecto | Variable | Alcance | Sensible | Propósito |
| --- | --- | --- | --- | --- |
| AuthUser | `AUTH_DATABASE_URL` | runtime | Sí | Conexión serverless al esquema `auth`. |
| AuthUser | `AUTH_DATABASE_DIRECT_URL` | migración | Sí | Conexión directa para migraciones `auth`. |
| AuthUser | `JWT_PRIVATE_KEY` | runtime | Sí | Firma RS256; nunca sale de AuthUser. |
| AuthUser | `JWT_ISSUER` / `JWT_AUDIENCE` | runtime | Configuración privada | Validación y emisión de `iss`/`aud`. |
| AuthUser | `CORS_ALLOWED_ORIGINS` | runtime | Configuración | Orígenes frontend permitidos. |
| Game | `GAME_DATABASE_URL` | runtime | Sí | Conexión serverless al esquema `game`. |
| Game | `GAME_DATABASE_DIRECT_URL` | migración | Sí | Conexión directa para migraciones `game`. |
| Game | `JWT_PUBLIC_KEY` | runtime | Configuración privada | Verificación RS256; no contiene la clave privada. |
| Game | `JWT_ISSUER` / `JWT_AUDIENCE` | runtime | Configuración privada | Valores efectivos iguales a AuthUser por entorno. |
| Game | `AUTHUSER_URL` | runtime | Configuración | Base para comprobar sesión y revocación. |
| Game | `CORS_ALLOWED_ORIGINS` | runtime | Configuración | Orígenes frontend permitidos. |
| Frontend | `NEXT_PUBLIC_AUTHUSER_URL` | navegador | Público | Base pública de AuthUser para el cliente. |
| Frontend | `NEXT_PUBLIC_GAME_URL` | navegador | Público | Base pública de Game para el cliente. |
| Frontend | `IGDB_CLIENT_ID` | servidor Next.js | Sí | Client ID para token de aplicación y cabecera IGDB; sin prefijo `NEXT_PUBLIC_`. |
| Frontend | `IGDB_CLIENT_SECRET` | servidor Next.js | Sí | Secreto Twitch para `client_credentials`; sin prefijo `NEXT_PUBLIC_`, nunca al navegador. |

Los valores reales se cargan en el proveedor de despliegue y en almacenes
locales ignorados por Git. No se copian a Markdown, fixtures, logs, capturas,
PR ni respuestas HTTP. `GB-003` comprobará presencia, permisos, URLs y
conectividad sin revelar los valores.

## 7. Entornos de trabajo

| Entorno | Base y conexiones | Orígenes | Criterio de uso |
| --- | --- | --- | --- |
| Local | `http://localhost:3001` AuthUser, `http://localhost:3002` Game, `http://localhost:3000` Frontend; URLs Neon solo por variables locales. | Solo `http://localhost:3000`. | Desarrollo y pruebas aisladas; nunca credenciales de producción. |
| Preview | Proyectos Vercel de cada aplicación y recurso/rama Neon de prueba. | Origen exacto de la preview correspondiente. | Contratos, migraciones controladas, CI y humo sin datos de producción. |
| Production | Proyectos Vercel publicados desde `main` y conexión Neon de producción. | Dominios finales exactos, definidos antes de `GB-012`. | Solo después de migraciones, variables y pruebas de aceptación. |

No se fija aquí ningún dominio de preview/producción: se determinará al crear
los recursos de `GB-003` y se actualizará la matriz sin introducir secretos.

## 8. Fuera de alcance

Esta subtarea no crea proyectos Neon/Vercel, no ejecuta migraciones, no cambia
permisos, no despliega servicios y no añade campos fuera de la forma mínima.
