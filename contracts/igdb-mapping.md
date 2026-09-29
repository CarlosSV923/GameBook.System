# Mapeo y límites de IGDB v4 — contrato vigente

Este contrato reemplaza para implementación a [`rawg-mapping.md`](rawg-mapping.md),
que se conserva únicamente como evidencia histórica. La fuente es la
[documentación oficial de IGDB](https://api-docs.igdb.com/). No hay código de
aplicación ni prueba en vivo con las credenciales del autor en esta revisión.

## 1. Autenticación y frontera

IGDB v4 usa credenciales de **aplicación**, no el JWT de usuario de GameBook:

1. Solo el servidor de Next.js lee `IGDB_CLIENT_ID` e `IGDB_CLIENT_SECRET`
   desde variables privadas de Vercel (o un almacén local ignorado por Git).
2. Solicita un token con `POST https://id.twitch.tv/oauth2/token` y
   `grant_type=client_credentials`. Usa `expires_in` para renovarlo antes de
   caducar; un token de aplicación expirado puede renovarse y reintentarse
   como máximo una vez. Nunca se registra ni devuelve el secreto o el token.
3. Consulta `POST https://api.igdb.com/v4/{endpoint}` con cuerpo APICalypse,
   cabeceras `Client-ID` y `Authorization: Bearer <token de aplicación>`.
   Los endpoints disponibles se fijan en el servidor, nunca vienen como URL
   arbitraria del navegador. IGDB no habilita CORS para llamadas directas
   desde JavaScript del navegador.
4. El navegador consume rutas propias de Next.js sin credenciales IGDB. Las
   llamadas públicas no necesitan JWT de AuthUser. Game tampoco necesita
   credenciales IGDB: recibe la instantánea básica del frontend autenticado.

## 2. Cobertura de los flujos del MVP

| Flujo | Consulta IGDB propuesta | Resultado que normaliza Next.js |
| --- | --- | --- |
| Catálogo inicial | `/games`, `fields` de tarjeta, `where total_rating != null`, `sort total_rating desc`, `limit`/`offset` | Juegos mejor puntuados por valoración combinada; puntuación 0–100. |
| Filtros combinados | `/games`, `search` para nombre y condiciones `where` con `&` para plataforma y `first_release_date` | Intersección de nombre, plataforma y año/rango; la búsqueda textual puede ordenar por relevancia. |
| Sugerencias de juegos | `/games` con `search`, `fields id,name`, `limit` acotado | Opciones de nombre; seleccionar aplica el nombre/ID acordado en la interfaz. |
| Sugerencias de plataformas | `/platforms` con `search`, `fields id,name`, `limit` acotado | Opciones `{ id, name }` para el filtro; sin precargar todo el catálogo. |
| Detalle | `/games` con `where id = <igdbId>` y campos de detalle | Una ficha por ID o resultado ausente; `summary`, géneros, desarrolladores, fecha e imágenes. |
| Imágenes | `cover.image_id` y, si la vista lo requiere, `screenshots.image_id` | URL HTTPS formada solo con el `image_id` validado y tamaños admitidos por IGDB. |

Los campos necesarios están documentados en la entidad Game de IGDB:
`id`, `name`, `first_release_date`, `platforms.name`, `cover.image_id`,
`total_rating`, `summary`, `genres.name`, `involved_companies.developer`,
`involved_companies.company.name` y `release_dates`. Las plataformas se
identifican por sus IDs IGDB, no por sus nombres.

## 3. Filtros, clasificación y paginación

- `name`: texto recortado y escapado antes de construir APICalypse; nunca se
  interpola como sintaxis libre. La consulta `search` puede devolver resultados
  por similitud, por lo que **solo el catálogo sin búsqueda** promete orden
  descendente estricto por `total_rating`. No se añaden filtros de producto.
- `platformId`: entero positivo de una sugerencia de `/platforms`; condición
  `platforms = <id>` combinada con las demás mediante `&`.
- Año: rango UTC semiabierto de `first_release_date` en segundos Unix,
  `[añoInicial-01-01, (añoFinal+1)-01-01)`, para incluir todo el año final.
  Un año único usa los mismos extremos lógicos. Validar año, orden y límites
  antes de consultar. Si falta `first_release_date`, el juego no coincide con
  un filtro anual.
- `total_rating`: promedio combinado IGDB de valoración de usuarios y
  críticos, en escala 0–100. Se excluye `null` del listado inicial; no se
  sustituye por `rating` ni `aggregated_rating`. Si el usuario busca por
  nombre, una ficha sin puntuación puede aparecer con valor ausente.
- `limit`/`offset`: tamaño de página propio acotado (20 resultados visibles),
  con una entrada adicional para calcular `hasNext` sin requerir el endpoint
  `/games/count` en cada desplazamiento. IGDB admite hasta 500 resultados
  por solicitud; GameBook no expondrá ese máximo al navegador. Al cambiar un
  filtro, se reinicia `offset`; el cliente descarta respuestas obsoletas y
  elimina posibles duplicados por `igdbId` entre páginas. Los resultados
  remotos pueden cambiar durante el scroll; no se promete snapshot estable.

Las sugerencias se acotan a 10 juegos y 20 plataformas; el frontend emplea
debounce y cancelación de respuestas obsoletas. Las consultas del catálogo y
de favoritos se paginan de manera independiente.

## 4. Mapeo de tarjeta, favorito y modal

| Campo GameBook | Fuente IGDB | Regla |
| --- | --- | --- |
| `igdbId` | `games.id` | Entero positivo, identidad externa del favorito. |
| `name` | `games.name` | Título original recibido; no se traduce. |
| `released` | `games.first_release_date` | Convertir segundos Unix a fecha UTC `YYYY-MM-DD` o `null`; el año de tarjeta/filtro local se deriva de esta fecha. |
| `imageUrl` | `games.cover.image_id` | Construir URL `https://images.igdb.com/igdb/image/upload/t_cover_big/{image_id}.jpg` tras validar `image_id`; `null` si falta. |
| `rating` | `games.total_rating` | Número 0–100 o `null`, sin convertir a la escala 0–5 de RAWG. |
| `platforms[].id/name` | `games.platforms.id/name` | Conservar pares `{ id, name }` válidos, sin traducir. |

La instantánea de Game conserva solo esos campos básicos. No almacena
descripción, géneros, desarrolladores, capturas ni respuesta IGDB completa.
Se mantiene unicidad `(userId, igdbId)` y se actualiza la instantánea solo al
abrir satisfactoriamente el detalle de un favorito.

El modal usa `summary` como descripción (y muestra ausencia si no existe),
`genres.name` para géneros y solo las compañías con
`involved_companies.developer = true` para desarrolladores, mostrando su
`company.name`. Puede mostrar imágenes de `cover`/`screenshots` si existen.
Para fecha completa se contrastan `first_release_date` y `release_dates`:
**solo se presenta día/mes/año como exacto cuando IGDB indique precisión de
día**; si la fecha es aproximada, parcial o ausente, se muestra la precisión
disponible (por ejemplo, solo año) sin inventar un día. El contenido externo
se trata como texto no confiable y nunca como HTML ejecutable. Los datos IGDB
se muestran tal como llegan; solo la interfaz propia se traduce EN/ES.

## 5. Límites, fallos y uso permitido

| Situación | Respuesta interna de Next.js |
| --- | --- |
| Credenciales ausentes | `IGDB_NOT_CONFIGURED`; no llamar a Twitch/IGDB. |
| Token Twitch inválido, expiración no recuperable o `401`/`403` | `IGDB_AUTH_FAILED`; reintentar una sola renovación cuando proceda, sin revelar secretos. |
| IGDB `429` | `IGDB_RATE_LIMITED`; respetar pausa y reintento acotado, mostrar estado temporal. |
| Red, timeout o `5xx` | `IGDB_UNAVAILABLE`; conservar favorito local y omitir sincronización si falla el detalle. |
| Respuesta incompatible, ID/imagen inválidos | `IGDB_INVALID_RESPONSE`; registrar solo contexto seguro y `requestId`. |
| Juego de detalle inexistente | Estado de detalle no disponible; no eliminar favorito por inferencia. |

La documentación de IGDB publica **4 solicitudes por segundo y 8 abiertas
simultáneamente**. El proxy limita consultas y concurrencia, usa caché breve
cuando sea apropiado para contenido público y evita petición remota por cada
tarjeta de favoritos. En Vercel la caché de proceso no garantiza coordinación
global entre instancias: las respuestas `429` son un estado esperado y se
prueban. Nunca se mezclan datos privados entre cuentas ni se cachean
respuestas privadas compartidas. No se fija aquí una cuota mensual RAWG.

IGDB declara uso no comercial gratuito sujeto al acuerdo de Twitch y pide
atribución visible a IGDB.com en productos integrados. GameBook mostrará un
enlace de atribución en vistas que presentan sus datos/imágenes y revisará
los términos vigentes antes del lanzamiento y ante cualquier monetización.
Las pruebas unitarias usan respuestas simuladas; la credencial real del autor
no es necesaria para documentar ni congelar este contrato.

## 6. Fuentes y alcance

- [IGDB API: autenticación, solicitudes, CORS, límites, búsqueda, filtros y
  paginación](https://api-docs.igdb.com/).
- [IGDB API: entidades Game, Platform, Involved Company, Release Date e
  imágenes](https://api-docs.igdb.com/).

Las rutas propias de Next.js, los códigos `IGDB_*`, límites de UI, caché y
normalización son **decisiones de GameBook**; no se atribuyen como respuestas
ni garantías de IGDB. La verificación aquí es documental. La prueba real de
integración, incluyendo búsqueda con tres filtros simultáneos, token
renovado y comportamiento `429`, corresponde a `GB-009` y `GB-012`.
