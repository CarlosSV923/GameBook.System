# Contrato de identidad JWT y revocación

Este documento cierra la identidad compartida por AuthUser y Game para
`GB-002.03`. No contiene claves ni tokens reales. Los valores concretos de las
variables se configurarán por entorno en `GB-003`; el algoritmo, los claims y
el comportamiento de revocación son comunes a los tres consumidores.

## 1. Token emitido por AuthUser

AuthUser firma cada `accessToken` con RS256. El encabezado debe incluir:

| Campo | Tipo | Regla contractual |
| --- | --- | --- |
| `alg` | string | Exactamente `RS256`; se rechaza cualquier otro algoritmo. |
| `typ` | string | `JWT`. |

El payload debe incluir todos estos claims:

| Claim | Tipo | Regla contractual |
| --- | --- | --- |
| `sub` | UUID string | Identidad de la cuenta. Game usa este valor como `userId`; nunca acepta un `userId` equivalente desde el navegador. |
| `ver` | integer positivo | Versión de sesión persistida por AuthUser. El token solo es vigente si coincide con la versión actual de la cuenta. |
| `iat` | NumericDate | Instante UTC de emisión. |
| `exp` | NumericDate | Caducidad UTC; AuthUser emite una vigencia nominal de 3600 segundos (`exp = iat + 3600`). Se rechaza cuando la hora actual alcanza `exp`. |
| `iss` | string | Coincide exactamente con el valor configurado en `JWT_ISSUER`; no se acepta un emisor alternativo. |
| `aud` | string | Coincide exactamente con el valor configurado en `JWT_AUDIENCE`; AuthUser y Game usan el mismo valor por entorno. |

El cliente trata el token como opaco: no lo renueva automáticamente ni usa
claims decodificados para autorizar. La decisión de autorización corresponde a
AuthUser y Game. No se emite refresh token en el MVP.

## 2. Claves y configuración

| Servicio | Variable | Propiedad y alcance |
| --- | --- | --- |
| AuthUser | `JWT_PRIVATE_KEY` | Clave privada PEM para firmar; solo existe en el entorno de AuthUser. |
| AuthUser | `JWT_ISSUER` | Emisor esperado y emitido en `iss`; valor obligatorio, sin valor por defecto. |
| AuthUser | `JWT_AUDIENCE` | Audiencia emitida en `aud`; valor obligatorio, sin valor por defecto. |
| Game | `JWT_PUBLIC_KEY` | Clave pública PEM para verificar RS256; nunca recibe la privada. |
| Game | `JWT_ISSUER` | Mismo valor efectivo que AuthUser para validar `iss`. |
| Game | `JWT_AUDIENCE` | Mismo valor efectivo que AuthUser para validar `aud`. |
| Game | `AUTHUSER_URL` | URL base privada para consultar `GET /v1/auth/session`; no se obtiene del token ni del navegador. |

Las variables se documentan solo por nombre. No se incluyen valores, claves,
conexiones ni tokens en repositorios, ejemplos, logs, prompts o respuestas.
AuthUser y Game fallan al arrancar si falta una variable necesaria o si la
configuración criptográfica no permite verificar la firma.

## 3. Verificación en AuthUser

`GET /v1/auth/session` con `Authorization: Bearer <token>` ejecuta, en este
orden, las comprobaciones siguientes:

1. Existe exactamente un esquema Bearer y el JWT se puede parsear.
2. La firma verifica con la clave privada/pública correspondiente y el
   algoritmo permitido es `RS256`.
3. `sub` es un UUID, `ver` es un entero positivo, `iat`/`exp` son NumericDate,
   `exp` no ha vencido, y `iss`/`aud` coinciden con la configuración.
4. La cuenta identificada por `sub` existe, no está deshabilitada (`isDisabled =
   false`) y su versión persistida coincide con `ver`.

Una respuesta `200` devuelve la identidad pública descrita en
[`authuser.openapi.yaml`](authuser.openapi.yaml). Nunca devuelve contraseña,
hash, claves ni el token recibido.

### Respuestas `401` de AuthUser

Todos los casos siguientes son `401` y usan el cuerpo `ErrorResponse` del
contrato OpenAPI. El frontend puede traducir el `code`; `message` no contiene
detalles sensibles.

| Código estable | Situación |
| --- | --- |
| `TOKEN_MISSING` | No existe la cabecera Bearer. |
| `TOKEN_INVALID` | Formato, firma, algoritmo, `sub`, `ver`, `iss` o `aud` inválidos. |
| `TOKEN_EXPIRED` | `exp` ya venció. |
| `SESSION_REVOKED` | `ver` no coincide con la versión actual de la cuenta, o la cuenta ya no es válida. |
| `ACCOUNT_DISABLED` | La cuenta existe, pero fue deshabilitada lógicamente. |

## 4. Emisión, cambio de contraseña y revocación

- Al crear una cuenta, AuthUser inicializa su versión de sesión en `1`.
- El login firma el JWT con la versión vigente y devuelve `expiresIn: 3600`.
- Un cambio de contraseña exige el token vigente, comprueba la contraseña
  actual y valida la nueva con la política del contrato AuthUser.
- En una única operación persistente, AuthUser guarda el nuevo hash e
  incrementa la versión (`ver = ver + 1`). No se acepta una actualización
  parcial.
- La respuesta correcta del cambio es `204`; el token usado queda revocado de
  inmediato, incluido antes de su `exp`. El frontend elimina el token y exige
  un nuevo login.
- Todos los tokens anteriores quedan revocados porque conservan la versión
  anterior. No se mantiene una lista de tokens ni se acepta revocación solo por
  esperar a la expiración.
- `DELETE /v1/users/me` marca `isDisabled = true` e incrementa `sessionVersion`
  atómicamente. Conserva la cuenta y sus favoritos indefinidamente, no permite
  reactivación en el MVP y deja inutilizables todos los JWT emitidos.

## 5. Verificación cerrada en Game

Cada operación protegida de Game ejecuta ambas capas:

1. Verificación local con `JWT_PUBLIC_KEY`, `RS256`, `iss`, `aud`, `sub`, `ver`,
   `iat` y `exp`.
2. Consulta de `GET {AUTHUSER_URL}/v1/auth/session` reenviando exactamente el
   mismo Bearer JWT.

Game solo usa el `sub` validado para delimitar datos. No consulta tablas del
   esquema `auth`, no recibe un token de servicio separado y no confía en un
   identificador de usuario enviado en la ruta, query o cuerpo.

| Resultado de la comprobación | Respuesta de Game | Efecto |
| --- | --- | --- |
| Firma/claims/exp inválidos localmente | `401` con `TOKEN_INVALID` o `TOKEN_EXPIRED` | No lee ni modifica favoritos. |
| AuthUser devuelve `401` (`TOKEN_*`, `SESSION_REVOKED` o `ACCOUNT_DISABLED`) | `401` con el mismo código estable | No lee ni modifica favoritos. |
| Timeout, DNS, conexión rechazada o respuesta `5xx` de AuthUser | `503` con `AUTHUSER_UNAVAILABLE` | Fallo cerrado: no lee ni modifica favoritos. |
| AuthUser responde `200` | Continúa la operación para el `sub` validado | Nunca amplía el alcance a otra cuenta. |

El `503` no se traduce como credenciales inválidas en el frontend; permite
mostrar un estado temporal y reintentar sin borrar una sesión potencialmente
válida. Un `401` sí limpia `sessionStorage` y solicita un nuevo inicio de
sesión, conforme al comportamiento del MVP.

## 6. Fixtures ficticios de contrato

Payload decodificado de ejemplo (no es un token utilizable):

```json
{
  "sub": "7b7f3d2e-6d8d-4e8c-9e0c-2a96f2fb11aa",
  "ver": 3,
  "iat": 1790100000,
  "exp": 1790103600,
  "iss": "configured-authuser-issuer",
  "aud": "configured-gamebook-audience"
}
```

Error de sesión revocada:

```json
{
  "code": "SESSION_REVOKED",
  "message": "Authentication is no longer valid.",
  "requestId": "req_gb00203_fictitious"
}
```

Error de fallo cerrado en Game:

```json
{
  "code": "AUTHUSER_UNAVAILABLE",
  "message": "Authentication service is temporarily unavailable.",
  "requestId": "req_gb00203_game_fictitious"
}
```

## 7. Casos mínimos de prueba para las tareas de aplicación

| Caso | Resultado esperado |
| --- | --- |
| JWT RS256 válido, `ver` vigente y claims correctos | AuthUser `200`; Game permite la operación del `sub`. |
| Algoritmo cambiado, firma manipulada, `iss`/`aud` incorrectos o `sub` no UUID | `401 TOKEN_INVALID`; no hay acceso a datos. |
| `exp` alcanzado | `401 TOKEN_EXPIRED`; no se renueva automáticamente. |
| `ver` anterior después de cambiar contraseña | AuthUser `401 SESSION_REVOKED`; Game `401 SESSION_REVOKED`. |
| Cuenta con `isDisabled = true`, incluso con JWT no vencido | AuthUser `401 ACCOUNT_DISABLED`; Game `401 ACCOUNT_DISABLED`; no accede a favoritos. |
| AuthUser sin respuesta o con `5xx` durante una petición protegida de Game | Game `503 AUTHUSER_UNAVAILABLE`; operación abortada sin efectos parciales. |
| Cambio de contraseña concurrente o repetido | Hash y aumento de versión atómicos; no queda una versión intermedia aceptada. |

Estos casos son fixtures y criterios de integración; no autorizan aún la
implementación de código en esta subtarea.
