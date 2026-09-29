# GB-002-CONTRACTS-v3 — línea base IGDB vigente

Estado: revisión documental completada el 2026-09-23 para `GB-002.08` y
`GB-002.09`. **Esta es la única línea base de contratos vigente para
implementación.** Sustituye el proveedor RAWG por IGDB v4 antes de que exista
código de producto, migración de favoritos o despliegue de aplicación. El
autor eligió la métrica combinada `total_rating` de IGDB para catálogo inicial
y tarjetas. No se utilizaron ni almacenaron sus credenciales reales.

## 1. Precedencia y alcance

1. [`../specs/mvp.md`](../specs/mvp.md) manda sobre el comportamiento del
   producto; [`../plan/mvp.md`](../plan/mvp.md) y [`../tasks/mvp.md`](../tasks/mvp.md)
   organizan la ejecución.
2. Esta v3 gobierna los contratos de AuthUser, Game, Frontend↔IGDB y Neon.
   `contract-baseline.md` (v1), `contract-baseline-v2.md` y sus artefactos
   RAWG se conservan **solo como historial** y no son fuentes de implementación.
3. El contrato AuthUser y JWT no cambian. En Game, `rawgId` se reemplaza por
   `igdbId` en rutas, payloads, unicidad y datos persistidos. El campo público
   `rating` de la instantánea representa `games.total_rating` IGDB (0–100), no
   la puntuación 0–5 anterior.
4. El aislamiento runtime Neon acreditado por `GB-003.02` permanece. La
   enmienda histórica [`persistence-access-amendment.md`](persistence-access-amendment.md)
   conserva el riesgo de los migradores antiguos con `neon_superuser`; la
   revisión operativa vigente para Actions es
   [`persistence-access-amendment-v2.md`](persistence-access-amendment-v2.md),
   emitida por `GB-003.06`. Esta v3 no autoriza secretos de migración hasta
   que `GB-003.04` los configure por ambiente.

## 2. Artefactos vigentes

| Artefacto | Propósito | SHA-256 |
| --- | --- | --- |
| [`authuser.openapi.yaml`](authuser.openapi.yaml) | Registro, login, sesión y contraseña; sin cambio | `599B87EC17EA5B232D217A6062DF59DA6A0E56FA72D4FD1E415C1F7A8481853E` |
| [`jwt-revocation.md`](jwt-revocation.md) | JWT RS256, versión y revocación; sin cambio | `E4794708C543CED192A1DBB584F3775EC3ABF8B21D86FC3E47C5A912EC06B8A6` |
| [`game-igdb.openapi.yaml`](game-igdb.openapi.yaml) | API Game con `igdbId`, rating 0–100 y Bearer de usuario | `74B88B79128F7D88F430CAED86CECBA843CA181AA1D08BFE011BD61FE6D3307F` |
| [`igdb-mapping.md`](igdb-mapping.md) | OAuth Twitch, consultas IGDB v4, filtros, imágenes, límites y fallos | `5026BD9A84EC8A4510EEE9B77030255901BEB172AE13AD55B27CFE6DE67E0A57` |
| [`persistence-igdb.md`](persistence-igdb.md) | Forma mínima, URLs, CORS y matriz de secretos IGDB | `CAB289359AD21C0CC45A761C813071A2B1AD5C29F4F6D01780EF05CFF3DB4809` |
| [`mvp-traceability-igdb.md`](mvp-traceability-igdb.md) | Trazabilidad de los 16 criterios con IGDB | `F54542334394A22550BE924365A9E7ED7B020382F4804097AAE7764278742A59` |
| [`persistence-access-amendment.md`](persistence-access-amendment.md) | Estado comprobado de permisos Neon y riesgo de migradores | `884471DF75ADD16CA4A7628DCF609ADBE2F13CCCB5001F1AD53D7A2EBD2288FF` |
| [`persistence-access-amendment-v2.md`](persistence-access-amendment-v2.md) | Roles migradores SQL limitados y pruebas por ambiente de GB-003.06 | `8173C24ED299649C68FC88D7DDEF966867D57B8DF45408E2FF0206CF6E61A057` |

Los hashes se calculan sobre los archivos exactos de esta tabla. Este índice
se excluye de su propia huella. Si cambia un artefacto, generar una nueva
revisión explícita, no modificarlo silenciosamente.

## 3. Compatibilidad transversal

- **Visitante:** el navegador llama a rutas públicas propias de Next.js. Solo
  Next.js obtiene el token de aplicación Twitch y consulta IGDB con
  `Client-ID`/`Authorization`. No se exige ni se envía JWT de usuario.
- **Usuario autenticado:** AuthUser emite JWT RS256 de una hora; Game valida
  firma, vencimiento y versión vigente con AuthUser en cada ruta protegida.
  El mismo JWT llega de Frontend a Game como Bearer. Ni Game ni AuthUser
  usan credenciales IGDB.
- **Favorito:** clave `(sub, igdbId)`; `POST` crea la instantánea básica,
  `GET` filtra/sugiere, `PATCH /{igdbId}/snapshot` sincroniza solo tras detalle
  IGDB exitoso y `DELETE /{igdbId}` elimina el propio. Un fallo IGDB conserva
  la tarjeta y omite el PATCH.
- **Catálogo:** `total_rating` descendente solo para listado inicial sin
  búsqueda; con `search` puede primar la relevancia. Plataforma y año se
  combinan mediante filtros `where` de IGDB. El frontend usa `limit`/`offset`
  acotados y deduplicación por `igdbId`.
- **Entornos:** `IGDB_CLIENT_ID` y `IGDB_CLIENT_SECRET` solo en el servidor
  Frontend. El token de aplicación se renueva según `expires_in` y jamás se
  expone. Las URL de Neon y claves JWT siguen las reglas de la enmienda.

## 4. Pruebas de contrato y límite de la verificación

La revisión de la [documentación oficial de IGDB](https://api-docs.igdb.com/)
confirma que hay soporte documental para todos los flujos del MVP: catálogo y
`total_rating`, búsqueda/sugerencias de juegos y plataformas, condiciones de
plataforma/fecha combinadas, paginación, detalle, descripciones, géneros,
desarrolladores e imágenes. También confirma OAuth de aplicación, ausencia de
CORS directo y límites 4 req/s y 8 peticiones abiertas. La funcionalidad de
cuenta, JWT y favoritos sigue siendo propia de GameBook y no depende de
autenticación de usuario IGDB.

Esta es una **verificación documental**, no un smoke test con las credenciales
del autor ni una garantía de disponibilidad futura de IGDB. `GB-009` probará
el adaptador y las consultas reales en preview; `GB-012` repetirá aceptación
en producción, incluyendo token expirado/renovación, tres filtros simultáneos,
desplazamiento infinito y `429`. Si esos ensayos revelan una incompatibilidad,
se reabre el contrato afectado antes de continuar con dependientes.
