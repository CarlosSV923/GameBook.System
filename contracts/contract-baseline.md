# GB-002-CONTRACTS-v1 — línea base de contratos

**Referencia histórica:** para la implementación vigente, leer también
[`GB-002-CONTRACTS-v2`](contract-baseline-v2.md), que incorpora una enmienda
de permisos y migraciones Neon sin alterar los seis artefactos ni sus huellas
de esta revisión.

Esta es la referencia coordinada para comenzar las tareas de aplicación. La
línea base reúne AuthUser, Game, Frontend, RAWG y persistencia sin añadir
funcionalidad fuera de `specs/mvp.md`.

- Estado: revisión documental completada para `GB-002.07`.
- Alcance: contratos HTTP `/v1`, identidad JWT, revocación, mapeo RAWG,
  persistencia mínima, variables por nombre y errores.
- Seguridad: no contiene valores de secretos, conexiones, claves privadas ni
  tokens reales.
- Regla: un consumidor o productor implementa contra esta línea base; un cambio
  incompatible requiere actualizar los documentos relacionados y una nueva
  referencia coordinada.

## 1. Dependencias y artefactos incluidos

Las dependencias `GB-002.02`, `GB-002.04`, `GB-002.05` y `GB-002.06` figuran
`[ RESOLVED ]` en [`../tasks/mvp.md`](../tasks/mvp.md). Los artefactos que
forman esta versión son:

| Artefacto | Responsabilidad | Versión declarada |
| --- | --- | --- |
| [`authuser.openapi.yaml`](authuser.openapi.yaml) | Registro, login, sesión y cambio de contraseña | OpenAPI `0.1.0` |
| [`game.openapi.yaml`](game.openapi.yaml) | Favoritos, filtros, sugerencias, snapshot y eliminación | OpenAPI `0.1.0` |
| [`jwt-revocation.md`](jwt-revocation.md) | Claims, RS256, claves, expiración, revocación y fallo cerrado | GB-002.03 |
| [`rawg-mapping.md`](rawg-mapping.md) | Mapeo RAWG, límites, cuota, atribución y proxy Next.js | GB-002.05 |
| [`persistence-environments.md`](persistence-environments.md) | Neon, esquemas, migraciones, URLs, CORS y variables | GB-002.06 |
| [`mvp-traceability.md`](mvp-traceability.md) | Trazabilidad de los 16 criterios a operaciones y propietarios | GB-002.01 |

## 2. Decisiones de compatibilidad

### AuthUser ↔ Game

- Game envía el mismo `Authorization: Bearer <JWT>` emitido por AuthUser; no
  existe un token de servicio alternativo.
- AuthUser firma RS256 con `JWT_PRIVATE_KEY`; Game verifica con
  `JWT_PUBLIC_KEY` y los valores compartidos de `JWT_ISSUER`/`JWT_AUDIENCE`.
- `sub` es el único UUID de usuario confiable. Game no recibe `userId` como
  autoridad y nunca crea una FK hacia el esquema `auth`.
- En cada operación protegida Game verifica localmente y después consulta
  `GET /v1/auth/session`. Un `401` se propaga como sesión inválida; una caída,
  timeout o `5xx` de AuthUser produce `503 AUTHUSER_UNAVAILABLE` y ningún
  acceso a favoritos.
- `PATCH /v1/users/me/password` responde `204`; el incremento atómico de
  `sessionVersion` invalida el token actual y todos los anteriores antes de
  `exp`.

### Game ↔ Frontend

- Todas las rutas protegidas de Game usan `/v1` y Bearer JWT; sus respuestas no
  incluyen `userId` de autoridad ni datos de otra cuenta.
- `GET /v1/favorites` combina `name`, `platformId`, `yearFrom` y `yearTo` con
  AND, devuelve `page`, `pageSize`, `total`, `hasNext` y orden determinista.
- `GET /v1/favorites/suggestions` consulta todos los favoritos propios y
  devuelve sugerencias acotadas de `name` o `platform`.
- Guardar usa `(sub, rawgId)` como clave lógica y devuelve `409
  FAVORITE_ALREADY_EXISTS`; snapshot solo actualiza un favorito propio ya
  existente; ausencia propia devuelve `404 FAVORITE_NOT_FOUND`.
- El frontend conserva el JWT solo en `sessionStorage`, limpia la sesión ante
  `401` y no traduce `503` como credenciales inválidas.

### Frontend ↔ RAWG

- Solo el servidor de Next.js usa `RAWG_API_KEY`; el navegador recibe datos
  mapeados, nunca la key ni URLs arbitrarias para consultar.
- Catálogo, detalle y plataformas se consultan según
  [`rawg-mapping.md`](rawg-mapping.md), con fechas inclusivas, paginación
  acotada, orden inicial `-rating`, valores ausentes explícitos y atribución.
- Un fallo de detalle RAWG conserva el favorito local y no llama a snapshot;
  un error remoto se normaliza a un código `RAWG_*` interno.

## 3. Forma uniforme de error

Los dos microservicios usan `{ code, message, requestId?, details? }`.
`code` es estable; `message` no contiene secretos ni detalles internos.

| Código | HTTP | Productor | Consumidor esperado |
| --- | --- | --- | --- |
| `VALIDATION_ERROR` | 400 | AuthUser/Game | Mostrar errores por campo; no enviar de nuevo sin corregir. |
| `TOKEN_MISSING`, `TOKEN_INVALID`, `TOKEN_EXPIRED`, `SESSION_REVOKED` | 401 | AuthUser/Game | Limpiar sesión y exigir login, salvo que la operación sea pública. |
| `INVALID_CREDENTIALS` | 401 | AuthUser | Mensaje genérico sin enumerar cuentas. |
| `FAVORITE_NOT_FOUND` | 404 | Game | Tratar como ausencia propia sin revelar otras cuentas. |
| `EMAIL_ALREADY_REGISTERED`, `FAVORITE_ALREADY_EXISTS` | 409 | AuthUser/Game | Mostrar conflicto contextual sin duplicar datos. |
| `AUTHUSER_UNAVAILABLE` | 503 | Game | Estado temporal; no borrar una sesión potencialmente válida. |
| `INTERNAL_ERROR` | 500 | AuthUser/Game | Estado genérico y `requestId`; sin stack trace. |

## 4. Fixtures contractuales mínimos

Los siguientes fixtures son ficticios y describen la compatibilidad mínima entre
productores y consumidores. Los ejemplos completos de cada operación viven en
los OpenAPI enlazados.

| Fixture | Operación | Resultado |
| --- | --- | --- |
| Registro correcto | `POST /v1/auth/register` con `fullName`, email y confirmación | `201`, identidad pública; nunca JWT/hash. |
| Email duplicado | Registro con correo normalizado existente | `409 EMAIL_ALREADY_REGISTERED`. |
| Login correcto | `POST /v1/auth/login` | `200`, `accessToken`, `Bearer`, `expiresIn=3600`, identidad mínima. |
| Credenciales incorrectas | Login con contraseña incorrecta | `401 INVALID_CREDENTIALS` genérico. |
| Sesión vigente | `GET /v1/auth/session` con JWT RS256 y `ver` actual | `200`, identidad pública. |
| JWT expirado | Cualquier ruta protegida con `exp` alcanzado | `401 TOKEN_EXPIRED`; no refresh automático. |
| JWT manipulado | Firma/algoritmo/issuer/audience/sub inválidos | `401 TOKEN_INVALID`; no se accede a datos. |
| Token revocado | JWT con `ver` anterior tras cambio de contraseña | `401 SESSION_REVOKED` en AuthUser y Game. |
| Guardar favorito | `POST /v1/favorites` con Bearer válido | `201`; unicidad `(sub, rawgId)`. |
| Duplicar favorito | Mismo `sub` y `rawgId` dos veces | `409 FAVORITE_ALREADY_EXISTS`. |
| Favorito inexistente | Snapshot o DELETE de un favorito propio ausente | `404 FAVORITE_NOT_FOUND`. |
| AuthUser caído | Game protegido sin respuesta/`5xx` de AuthUser | `503 AUTHUSER_UNAVAILABLE`; no lectura ni escritura. |
| RAWG caído en detalle | Detalle RAWG falla al abrir un favorito | Tarjeta permanece; no se ejecuta snapshot; código `RAWG_UNAVAILABLE`. |
| Cambio correcto | PATCH de contraseña con token vigente | `204`; el token usado queda revocado inmediatamente. |

## 5. Revisión de las tres líneas de trabajo

La coordinación revisó cada frontera desde la perspectiva del productor y
consumidor previsto:

| Línea | Comprobación | Resultado |
| --- | --- | --- |
| AuthUser | Rutas OpenAPI, identidad pública, Bearer, errores, claims, expiración y cambio atómico de versión | Compatible con `jwt-revocation.md` y consumidores; sin hash/token en respuestas. |
| Game | Rutas OpenAPI, snapshot mínimo, `(sub, rawgId)`, filtros, paginación, sugerencias, 404/409/503 | Compatible con AuthUser y Frontend; aislamiento por `sub` explícito. |
| Frontend | URLs públicas, `sessionStorage`, errores traducibles, paginación/sugerencias y proxy RAWG servidor | Compatible con ambos OpenAPI y `rawg-mapping.md`; ningún secreto al navegador. |

No se realizó una revisión de código porque esta fase documental no contiene
implementación. Las futuras tareas deben reportar cualquier discrepancia
contra esta referencia antes de cambiar un payload o comportamiento.

## 6. Política de compatibilidad

- Cambios aditivos opcionales pueden proponerse como una revisión menor si no
  alteran semántica, códigos, límites, aislamiento ni nombres existentes.
- Cambiar una ruta, método, campo obligatorio, tipo, código, claim, error,
  límite de paginación, variable o regla de seguridad es incompatible: exige
  actualizar OpenAPI, fixtures, consumidores y esta referencia antes de
  implementarse.
- Ningún repositorio puede publicar una forma distinta de `ErrorResponse`,
  emitir un JWT que Game no pueda verificar o aceptar una operación protegida
  cuando AuthUser no confirma la sesión.
- La próxima revisión contractual deberá crear otra referencia versionada; no
  se sobrescribe silenciosamente esta línea base.

## 7. Huellas SHA-256 de la referencia

Las huellas se calcularon localmente con `Get-FileHash -Algorithm SHA256` sobre
los seis artefactos incluidos. El archivo de esta línea base se excluye de sus
propias huellas para evitar una referencia circular.

| Archivo | SHA-256 |
| --- | --- |
| `authuser.openapi.yaml` | `599B87EC17EA5B232D217A6062DF59DA6A0E56FA72D4FD1E415C1F7A8481853E` |
| `game.openapi.yaml` | `72990FF57DF5819F02A626A8638C5BA1565D2EA5463A7365F7F1E188F121BB50` |
| `jwt-revocation.md` | `E4794708C543CED192A1DBB584F3775EC3ABF8B21D86FC3E47C5A912EC06B8A6` |
| `rawg-mapping.md` | `729400A393E91368FBDD2CF99FCB92003023D8EC487752C1588A918C1EB155C6` |
| `persistence-environments.md` | `6969F438A745CBC70C13AE3E83FC547D2A2C704A1EA29B17004E3BBCB5EC6571` |
| `mvp-traceability.md` | `E210A9089F6912085383F96791C96821C9ADD8655D3F96DBB8B66C2639832515` |

La referencia se considera congelada solo con los valores SHA-256 sustituidos;
si cualquiera de los seis archivos cambia, se debe generar una nueva revisión.

## 8. Fuera de alcance

Esta línea base no crea repositorios, no modifica ramas, no implementa
controladores, no ejecuta migraciones, no configura Vercel/Neon y no introduce
un paquete de código compartido.
