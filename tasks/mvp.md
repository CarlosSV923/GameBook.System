# GameBook — Tareas ejecutables del MVP

## 1. Propósito y regla de ejecución

Este registro descompone [`../plan/mvp.md`](../plan/mvp.md) en entregables pequeños y verificables. [`../specs/mvp.md`](../specs/mvp.md) conserva la autoridad sobre el producto. Los estados vigentes y la evidencia de los repositorios ya creados se indican abajo; modificar este documento no autoriza a iniciar otras subtareas.

Estados permitidos, escritos exactamente así:

- `[ NEW ]`: no iniciada o detenida por dependencias; no se ejecuta.
- `[ ACTIVE ]`: asignada a una sola herramienta/persona, con todas sus dependencias en `[ RESOLVED ]`, y en ejecución.
- `[ RESOLVED ]`: entregable comprobado y evidencia registrada; si la subtarea produce PR, todos sus PR deben estar fusionados en la rama de destino y verificados por el agente **después de que el autor notifique que los completó**. Para código de aplicación se exige además CI verde como criterio de calidad, no como protección de GitHub; se aplican las excepciones de publicación en `main` y de tareas sin PR.

**Regla de bloqueo absoluta:** una herramienta de IA **no debe iniciar, modificar archivos para, ni pasar a `[ ACTIVE ]` una subtarea si cualquiera de sus dependencias indicadas no está en `[ RESOLVED ]`**. Una dependencia de grupo (`GB-###`) exige que **todas** sus subtareas estén en `[ RESOLVED ]`. Una dependencia de subtarea (`GB-###.##`) exige solo esa subtarea; esto permite el paralelismo previsto. `—` significa que no hay dependencia. El estado del grupo se deriva: `[ ACTIVE ]` si alguna subtarea está activa, `[ RESOLVED ]` si todas están resueltas y `[ NEW ]` en los demás casos, incluso si tiene progreso parcial pero nadie trabaja en ella. Si se reabre una dependencia por una regresión, se detienen sus dependientes y vuelven a `[ NEW ]` hasta corregirla.

La coordinación es la única responsable de actualizar este archivo y de asignar dueño a una subtarea; las herramientas de IA pueden proponer cambios de estado con enlace a issue/PR/prueba, pero no editarlo simultáneamente. Una subtarea `[ ACTIVE ]` tendrá dueño, repositorio, rama o recurso, fecha de inicio y evidencia de avance en su issue de GitHub o nota de coordinación. **Cada PR creado se registra de inmediato en la tabla de PR de este archivo**, con ID de subtarea, repositorio, URL, rama origen/destino y estado; un issue o nota externa no sustituye ese registro. El autor revisa y fusiona/cierra personalmente los PR; el agente no los fusiona por su cuenta. Tras el aviso del autor, el agente comprueba el estado real `MERGED`, la rama de destino y el entregable; registra fecha/evidencia de la comprobación y solo entonces cambia la subtarea a `[ RESOLVED ]` si todos sus criterios se cumplen. Un PR abierto o cerrado sin merge no resuelve la tarea. Las tareas de un mismo repo pueden solaparse solo con propiedad de archivos disjunta y contratos congelados. Los números son estables; una rama de implementación puede usar `feature/004-03/resumen-de-tarea` para `GB-004.03`.

**Rama y entorno de comprobación:** hasta `GB-012`, los entregables de las tres aplicaciones se verifican en `develop` (o en el PR que se integrará allí) y se ejecutan localmente; no se exige despliegue Vercel Preview. `main` solo se comprueba como rama predeterminada existente, no por su contenido. La verificación de contenido y despliegue en `main` corresponde a la publicación final, **después** de la aceptación integral local. Rige la [enmienda local-first](../environments/deployment-policy-local-first.md) sobre toda referencia histórica a Vercel Preview.

No se inventan requisitos. Si surge una decisión funcional nueva, primero se pregunta al autor y se actualiza la especificación y el plan; solo después se modifica o añade la tarea afectada. Los commits y PR de aplicaciones estarán en inglés y seguirán las reglas de `plan/mvp.md`. Las tareas `GB-013` y `GB-014` ocurren **después** de finalizar las tres aplicaciones; `GameBook.System` no se crea antes.

## 2. Preparación y contratos

### [ RESOLVED ] GB-001 — Crear los tres repositorios públicos de aplicaciones

Ámbito: coordinación y GitHub. Cierre del grupo: existen `GameBook.Microservice.AuthUser`, `GameBook.Microservice.Game` y `GameBook.Frontend`, con URL verificada, `main`/`develop` y README bilingües iniciales **en `develop`**. **No incluye `GameBook.System` ni exige publicar los README en `main` antes de `GB-012`.**

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-001.01** — Validar acceso a GitHub CLI y nombres | — | `gh auth status` satisfactorio, propietario de GitHub identificado, nombres exactos y visibilidad pública confirmados sin publicar secretos. |
| [ RESOLVED ] **GB-001.02** — Crear/publicar los tres repositorios | GB-001.01 | Tres repos públicos creados con `gh`, `main` predeterminada y remotos verificados; no se crea un monorepo ni `GameBook.System`. |
| [ RESOLVED ] **GB-001.03** — Preparar flujo de ramas y PR | GB-001.02 | `develop`, `main`/`develop` sin protección ni requisitos obligatorios de merge, CI informativo, plantilla de PR en inglés y convención `feature/[task-number]/[resumen]` documentada. |
| [ RESOLVED ] **GB-001.04** — Crear README bilingües iniciales | GB-001.02 | En `develop` de cada repo: `README.md` inglés, `README.es.md` español, enlaces recíprocos entre idiomas y descripción inicial coherente. Visibilidad en la portada de GitHub se verifica tras la publicación final en `main`. |
| [ RESOLVED ] **GB-001.05** — Verificar repositorios y traspaso | GB-001.03, GB-001.04 | URLs, ramas, visibilidad y permisos verificados; ambos README comprobados **solo en `develop`** de cada aplicación y lista de repos entregada para los contratos. No se exige README en `main` antes de `GB-012`. |

### [ RESOLVED ] GB-002 — Congelar contratos y decisiones técnicas compartidas

Ámbito: coordinación, con revisión de los tres proyectos. Cierre del grupo: contratos OpenAPI y fixtures aprobados; consumidores y productores pueden implementarse por separado sin adivinar rutas ni campos.

**Contrato vigente:** [`GB-002-CONTRACTS-v3`](../contracts/contract-baseline-v3.md) para IGDB. `GB-002.01`–`GB-002.07` y v1/v2 son cierres históricos de RAWG, conservados sin reescribir su evidencia. Todo trabajo posterior debe leer v3; no implementar los archivos RAWG históricos.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-002.01** — Trazar requisitos a operaciones | GB-001 | Inventario de los 16 criterios de `specs/mvp.md`, operaciones necesarias, propietario de cada dato y casos de error; ninguna función nueva. |
| [ RESOLVED ] **GB-002.02** — Definir contrato AuthUser | GB-002.01 | OpenAPI para registro, login, sesión vigente y cambio de contraseña; validaciones, cuerpos, respuestas, ejemplos ficticios y Bearer documentados. |
| [ RESOLVED ] **GB-002.03** — Definir identidad JWT y revocación | GB-002.02 | Claims `sub` UUID, `ver`, `iat`, `exp`, emisor/audiencia, RS256, clave privada/pública, caducidad de una hora, `401` revocado y fallo cerrado de Game cuando AuthUser no responde. |
| [ RESOLVED ] **GB-002.04** — Definir contrato Game | GB-002.01, GB-002.03 | OpenAPI para guardar, listar/filtrar/paginar, sugerir, actualizar instantánea y eliminar; unicidad `(userId,rawgId)`, errores estables y aislamiento por `sub`. |
| [ RESOLVED ] **GB-002.05** — Definir mapeo y límites RAWG | GB-002.01 | Campos de tarjeta/detalle, plataformas y fechas, filtros combinados, sugerencias, paginación, valores ausentes, atribución, errores, cuota y llamadas servidoras Next.js verificados contra documentación RAWG. |
| [ RESOLVED ] **GB-002.06** — Definir persistencia y entornos | GB-002.03, GB-002.04, GB-002.05 | Esquemas Neon `auth`/`game`, migraciones independientes, forma mínima de plataformas, contratos de errores, URLs/CORS y matriz de variables **sin valores secretos**. |
| [ RESOLVED ] **GB-002.07** — Revisar y congelar contratos | GB-002.02, GB-002.04, GB-002.05, GB-002.06 | OpenAPI, fixtures de éxito/error/expiración/revocación y decisión de compatibilidad revisados por AuthUser, Game y Frontend; versión o hash de referencia anotado. |
| [ RESOLVED ] **GB-002.08** — Verificar viabilidad IGDB y decisión de puntuación | GB-002.07 | Documentación oficial contrastada con los 16 criterios; autor eligió `total_rating` combinado; se registran OAuth de aplicación, 4 req/s, CORS servidor, filtros, detalle e imágenes sin usar credenciales reales. |
| [ RESOLVED ] **GB-002.09** — Publicar contratos IGDB v3 | GB-002.08 | Especificación, plan, Game OpenAPI, mapeo, persistencia y trazabilidad revisados para `igdbId`, escala 0–100 y secretos Twitch; baseline v3 con hashes, v1/v2 preservados como historial. Sin código ni PR de aplicación. |

### [ RESOLVED ] GB-003 — Preparación histórica de Neon y Vercel sin código de producto

Ámbito: coordinación de entornos. El cierre histórico acredita los roles SQL runtime y migradores limitados probados por `GB-003.02`/`GB-003.06` y el inventario de credenciales publicado entonces. **No acredita infraestructura vigente**: las referencias Vercel de este grupo se conservan como historial. Las aplicaciones usan Neon `develop`/IGDB de pruebas localmente; AuthUser y Game se publican en Render y Frontend puede publicarse en Vercel solo después de la aceptación local. Ninguna credencial se expone en GitHub.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-003.01** — Preparar bases de datos por entorno | GB-002 | Proyecto Neon y entorno de prueba/producción definidos, con conexión documentada solo por nombre de variable. |
| [ RESOLVED ] **GB-003.02** — Aislar runtime y definir migraciones controladas | GB-003.01 | Esquemas `auth`/`game` y roles SQL runtime `*_app` limitados, verificados en `develop`/`production`. La restricción de los migradores queda para `GB-003.06`; este cierre acredita solo el aislamiento runtime. |
| [ RESOLVED ] **GB-003.06** — Restringir migradores para GitHub Actions | GB-003.02 | Crear por SQL `gamebook_auth_migrator_limited` y `gamebook_game_migrator_limited`, sin `neon_superuser`, `CREATEDB` ni `CREATEROLE`; conceder `USAGE`/`CREATE` solo en el esquema propio, sin `CREATE` en la base. Probar en `develop` conexión, creación/alteración propia y rechazo de lectura/DDL en el esquema ajeno; replicar y verificar en `production` solo tras éxito. Ajustar grants por defecto para los roles `*_app`, retirar de uso los migradores antiguos, registrar cómo se gestionan sus roles propietarios y publicar una nueva revisión contractual. Si Neon/Prisma impide el patrón, no continuar a secretos: documentar evidencia y consultar al autor. |
| [ RESOLVED ] **GB-003.03** — Preparar los tres proyectos Vercel (histórico) | GB-001, GB-002 | Se crearon proyectos vinculados a `main` y previews. Esta evidencia se conserva, pero los proyectos serán eliminados por el autor; no son el destino de publicación vigente. |
| [ RESOLVED ] **GB-003.04** — Publicar credenciales disponibles antes del código (histórico) | GB-003.03, GB-003.06 | Se inventariaron nombres/ámbitos de `AUTH_DATABASE_URL`, `GAME_DATABASE_URL`, `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET` en Vercel inicial y URLs directas solo en GitHub `develop`/`production`. Tras la eliminación manual de Vercel, esas variables deberán configurarse **de nuevo, solo para Production** en `GB-012.05`–`GB-012.07`; la evidencia antigua no acredita su presencia actual. |
| [ RESOLVED ] **GB-003.05** — Verificar inventario y traspaso histórico | GB-003.04 | Conserva evidencia de los nombres y responsabilidades, sin validar valores. Su calendario de previews queda sustituido por la [enmienda local-first](../environments/deployment-policy-local-first.md). Las pruebas de credenciales de desarrollo pasan a la integración local y las de producción a `GB-012`. |

## 3. Bases técnicas que pueden avanzar en paralelo

**Ventana 1:** `GB-006` puede comenzar tras `GB-002` `[ RESOLVED ]`; `GB-004` y `GB-005` esperan además a `GB-003` `[ RESOLVED ]`. Las tres avanzarán en paralelo en repositorios diferentes cuando se cumplan esas condiciones.

### [ RESOLVED ] GB-004 — Base DDD de AuthUser

Ámbito: `GameBook.Microservice.AuthUser`. Cierre del grupo: arquitectura y persistencia de usuarios listas para los casos de uso, con pruebas y arranque local comprobados; sin Vercel.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-004.01** — Inicializar NestJS y pnpm | GB-002, GB-003 | `src/api`, `src/application`, `src/domain`, `src/infrastructure`, `src/main.ts`, `tests/unit`, `tests/integration`, `prisma/`, scripts y `pnpm-lock.yaml` sin npm/yarn. |
| [ RESOLVED ] **GB-004.02** — Modelar usuario e interfaces | GB-004.01 | Entidad UUID, correo normalizado, reglas de contraseña, versión de sesión y contratos de repositorio/criptografía sin dependencias NestJS/Prisma en dominio. |
| [ RESOLVED ] **GB-004.03** — Crear esquema y migración propios | GB-004.02 | Prisma en `auth`, índice único de correo y migración reproducible; integración con esquema de prueba, sin tocar `game` ni ejecutar migraciones desde Vercel. |
| [ RESOLVED ] **GB-004.04** — Implementar adaptadores base | GB-004.03 | Repositorio Prisma, hash con sal y puertos de firma/verificación. Generar de forma segura el par RS256 **de desarrollo** y cargar `JWT_PRIVATE_KEY`, `JWT_ISSUER` y `JWT_AUDIENCE` en configuración local privada; entregar a Game solo la clave pública correspondiente. Los valores productivos se generan/publican en `GB-012.11`, no aquí. Validar carga sin imprimir valores. |
| [ RESOLVED ] **GB-004.05** — Preparar frontera API y logs | GB-004.04 | Módulos/controladores base, validación y formato de error, `requestId` y logs en inglés sin contraseñas ni JWT. Configurar CORS por lista exacta: si `CORS_ALLOWED_ORIGINS` aún falta, denegar llamadas cross-origin sin impedir el arranque del servicio; nunca abrir `*` por defecto. |
| [ RESOLVED ] **GB-004.06** — Probar base local | GB-004.05 | Unitarias/integración en `tests/`, lint/tipos/build con `pnpm`, CI verde y arranque NestJS/Prisma **local**. Verificar `AUTH_DATABASE_URL` de Neon `develop` con rol runtime propio y configuración JWT de desarrollo sin mostrar valores. Probar rechazo CORS no autorizado; el origen exacto del frontend local se integrará en `GB-010.05`. Sin prueba Vercel. |
| [ RESOLVED ] **GB-004.07** — Implementar Action de migraciones AuthUser | GB-004.03, GB-003.06 | Workflow propio en `.github/workflows/` con `pnpm`, migraciones `auth` versionadas y secreto `AUTH_DATABASE_DIRECT_URL` del ambiente correcto; ejecutar en `develop` al integrar cambios de migración y preparar ejecución controlada desde `main` para producción. Verificar éxito, idempotencia, rechazo de credencial ausente/esquema incorrecto y que Vercel no ejecuta migraciones. PR a `develop`, CI y aviso/merge del autor antes de `[ RESOLVED ]`. |

### [ RESOLVED ] GB-005 — Base DDD de Game

Ámbito: `GameBook.Microservice.Game`. Cierre del grupo: persistencia mínima y frontera de autenticación listas, sin acceso a tablas AuthUser.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-005.01** — Inicializar NestJS y pnpm | GB-002, GB-003 | Mismo árbol `src/` y `tests/` acordado, `prisma/`, scripts y lockfile propios; solo código del servicio Game. |
| [ RESOLVED ] **GB-005.02** — Modelar favorito e interfaces | GB-005.01 | Identidad lógica `(userId,igdbId)`, puntuación combinada 0–100 y campos básicos nulos cuando proceda, plataformas y contratos de repositorio/casos de uso según v3. |
| [ RESOLVED ] **GB-005.03** — Crear esquema y migración propios | GB-005.02 | Tablas `game` de favorito/plataformas, unicidad e índices de filtros, sin FK ni migraciones sobre `auth` y sin ejecutar migraciones desde Vercel. |
| [ RESOLVED ] **GB-005.04** — Implementar repositorio y consultas | GB-005.03 | Persistencia Prisma, filtros AND, orden estable, paginación acotada y selección de plataformas del usuario. |
| [ RESOLVED ] **GB-005.05** — Preparar guardia JWT y cliente AuthUser | GB-005.01, GB-002 | Verificación local RS256 y cliente de sesión con doble de prueba; token inválido `401`, dependencia caída `503`, sin asumir AuthUser desplegado todavía. |
| [ RESOLVED ] **GB-005.06** — Preparar API, logs y pruebas base | GB-005.04, GB-005.05, GB-004.04 | Frontera HTTP, errores, `requestId`, logs seguros, unitarias/integración aisladas, CI `pnpm` y arranque **local**. Cargar `JWT_PUBLIC_KEY`, `JWT_ISSUER` y `JWT_AUDIENCE` de desarrollo compatibles con AuthUser; verificar `GAME_DATABASE_URL` de Neon `develop` y clave pública sin revelar valores. Sin CORS configurado, denegar cross-origin sin abrir `*` ni bloquear arranque; fijar el origen exacto del frontend local en `GB-011.05`. Sin Vercel. |
| [ RESOLVED ] **GB-005.07** — Implementar Action de migraciones Game | GB-005.03, GB-003.06 | Workflow propio en `.github/workflows/` con `pnpm`, migraciones `game` versionadas y secreto `GAME_DATABASE_DIRECT_URL` del ambiente correcto; ejecutar en `develop` al integrar cambios de migración y preparar ejecución controlada desde `main` para producción. Verificar éxito, idempotencia, rechazo de credencial ausente/esquema incorrecto y que Vercel no ejecuta migraciones. PR a `develop`, CI y aviso/merge del autor antes de `[ RESOLVED ]`. |

### [ RESOLVED ] GB-006 — Dirección visual y base de Frontend

Ámbito: `GameBook.Frontend`. **Usar `interface-design`** para dirección y cualquier UI. Cierre del grupo: base Next.js coherente, tema/idioma persistentes y pruebas iniciales.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-006.01** — Definir dirección de interfaz | GB-002 | Con `interface-design`: dominio, mundo de color, firma visual, patrones genéricos a evitar, jerarquía de vistas y dirección aprobada si requiere decisión del autor; sin código prematuro. |
| [ RESOLVED ] **GB-006.02** — Inicializar Next.js y pnpm | GB-006.01 | Aplicación modular con `features/` y `shared/`, scripts/lockfile propios y separación de rutas servidoras IGDB/Twitch frente a UI cliente. En su PR a `develop`, corregir `README.md`, `README.es.md` y `CONTRIBUTING.md` iniciales de Frontend que aún mencionan RAWG; el autor debe fusionarlo antes del cierre. |
| [ RESOLVED ] **GB-006.03** — Crear sistema visual y preferencias | GB-006.02 | Tokens claro/oscuro, tema inicial del sistema y elección manual persistente; idioma inicial inglés y elección EN/ES persistente para visitantes y usuarios. |
| [ RESOLVED ] **GB-006.04** — Crear adaptadores y mocks tipados | GB-006.02, GB-002 | Clientes AuthUser/Game y contrato proxy IGDB v3 con dobles; URLs configurables, JWT solo en llamadas protegidas, credenciales/token de aplicación solo servidor. |
| [ RESOLVED ] **GB-006.05** — Preparar navegación y componentes base | GB-006.03, GB-006.04 | Layout y navbar, controles accesibles reutilizables, estados base y catálogo de mensajes propios EN/ES sin perder sesión/filtros al cambiar preferencias. |
| [ RESOLVED ] **GB-006.06** — Verificar base visual y CI | GB-006.05 | Unitarias/integración viables, lint/tipos/build con `pnpm`, revisión `interface-design` en escritorio/móvil y ambos temas/idiomas; CI verde. |

## 4. Funcionalidades que consumen los contratos

**Ventana 2:** tras resolver las bases, `GB-007` (AuthUser), los primeros entregables de `GB-008` (Game) y `GB-009` (Frontend) pueden avanzar en paralelo. `GB-008.05` **espera `GB-007` `[ RESOLVED ]`** antes de integrar la validación real de revocación. `GB-010` puede crear UI con mocks mientras AuthUser avanza, pero `GB-010.05` espera `GB-007`. Ninguna herramienta interpreta «contrato definido» como permiso para ignorar una dependencia explícita.

### [ RESOLVED ] GB-007 — Casos de uso y API AuthUser

Ámbito: `GameBook.Microservice.AuthUser`. Cierre del grupo: flujos de cuenta y revocación documentados con Swagger y probados localmente.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-007.01** — Registrar usuario | GB-004 | Caso de uso y endpoint con nombre/correo/contraseña/confirmación, validación y correo único; respuesta sin JWT, errores `400`/`409` probados. |
| [ RESOLVED ] **GB-007.02** — Iniciar sesión y emitir JWT | GB-007.01 | Credenciales genéricas al fallar, hash verificado, JWT RS256 de una hora con UUID/versión/claims acordados y respuesta contractual. |
| [ RESOLVED ] **GB-007.03** — Validar sesión vigente | GB-007.02 | Endpoint Bearer verifica firma, caducidad y versión persistida; rechaza token ausente, inválido, expirado o revocado. |
| [ RESOLVED ] **GB-007.04** — Cambiar contraseña y revocar | GB-007.03 | Requiere contraseña actual, valida nueva, actualiza hash+versión atómicamente; token actual y demás tokens previos dejan de funcionar. |
| [ RESOLVED ] **GB-007.05** — Documentar y probar API | GB-007.01, GB-007.02, GB-007.03, GB-007.04 | Swagger/OpenAPI con Bearer, validaciones/errores sin secretos; unitarias e integración reales de registro, unicidad, login, cambio, expiración y revocación. |
| [ RESOLVED ] **GB-007.06** — Verificar servicio local y contrato | GB-007.05 | CI verde, Swagger UI accesible localmente, OpenAPI compatible con `GB-002` y URL/puerto local documentados para Game y Frontend. |

### [ RESOLVED ] GB-008 — Favoritos y API Game

Ámbito: `GameBook.Microservice.Game`. Cierre del grupo: API protegida completa, aislada por usuario y compatible con JWT/AuthUser.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-008.01** — Guardar y eliminar favorito propio | GB-005 | Casos de uso/HTTP, unicidad concurrente, aviso de duplicado, eliminación de solo `(sub,igdbId)`; pruebas de éxito/error/usuario ajeno. |
| [ RESOLVED ] **GB-008.02** — Listar, filtrar y paginar | GB-005 | Filtros AND por nombre/plataforma/año o rango, orden estable, páginas acotadas y fin de lista; datos solo del `sub`. |
| [ RESOLVED ] **GB-008.03** — Sugerir nombre y plataforma | GB-008.02 | Endpoint sobre **todos** los favoritos propios, no solo la página visible; respuestas acotadas y sin datos de otras cuentas. |
| [ RESOLVED ] **GB-008.04** — Sincronizar instantánea | GB-008.01 | Actualiza solo campos básicos del favorito propio existente después de detalle IGDB exitoso; no crea ni borra implícitamente. |
| [ RESOLVED ] **GB-008.05** — Integrar JWT y revocación reales | GB-005, GB-007 | Configurar `AUTHUSER_URL` **local** de Game hacia AuthUser según el modo de ejecución (host o DNS interno de Compose); probar la misma cabecera Bearer enviada por frontend, verificación local más consulta a AuthUser en cada ruta protegida, `401`/`503` distinguidos y fallo cerrado. La URL productiva espera a `GB-012.11`. |
| [ RESOLVED ] **GB-008.06** — Swagger y pruebas completas | GB-008.01, GB-008.02, GB-008.03, GB-008.04, GB-008.05 | Documentación Bearer y fixtures; integración Prisma + JWT válido/ausente/manipulado/vencido/revocado, AuthUser caído y aislamiento por UUID. |
| [ RESOLVED ] **GB-008.07** — Verificar servicio local y contrato | GB-008.06 | CI verde, Swagger UI accesible localmente, OpenAPI compatible con `GB-002` y URL/puerto local documentados para Frontend. |

### [ RESOLVED ] GB-009 — Catálogo público y consulta IGDB

Ámbito: `GameBook.Frontend`. **Usar `interface-design`** en todos los entregables visuales. Cierre del grupo: visitante consulta juegos mejor puntuados, filtra y abre detalles sin JWT.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-009.01** — Integrar IGDB solo en servidor Next.js | GB-006 | Cliente Twitch `client_credentials` con renovación por expiración; consultas `POST` APICalypse IGDB de juegos, detalle y plataformas; Client ID/Secret y token solo servidor, errores `401`/`403`/`429`/`5xx`, límites 4 req/s y 8 concurrentes, ningún acceso directo del navegador. Probar con mocks y acreditar **localmente** una solicitud real con credenciales IGDB de desarrollo privadas y renovación segura del token; no usar ni exponer credenciales de producción. |
| [ RESOLVED ] **GB-009.02** — Crear listado y tarjetas | GB-006.06, GB-009.01 | Orden inicial `sort total_rating desc` (0–100, sin puntuación nula), imagen/nombre/año/plataformas/puntuación, faltantes seguros y jerarquía visual revisada. |
| [ RESOLVED ] **GB-009.03** — Crear filtros y sugerencias | GB-009.01, GB-009.02 | Nombre (`search`) y plataforma (`/platforms` search) con sugerencias, año único o rango UTC inclusivo de años, los tres combinados con `where` y `&`, cancelación de búsquedas obsoletas y estado vacío. |
| [ RESOLVED ] **GB-009.04** — Añadir desplazamiento infinito | GB-009.03 | `limit`/`offset` con detección de `hasNext`, siguientes páginas sin duplicados por `igdbId`, señal de carga y fin, reinicio de paginación al cambiar filtros; pruebas de límites y cambios remotos. |
| [ RESOLVED ] **GB-009.05** — Crear modal de detalle | GB-009.01, GB-009.02 | `summary`, géneros, compañías desarrolladoras, fecha con precisión real e imágenes IGDB cuando existan; cierre por botón/teclado, foco accesible y fallo remoto visible. |
| [ RESOLVED ] **GB-009.06** — Añadir atribución y estados externos | GB-009.03, GB-009.04, GB-009.05 | Enlace visible a IGDB, carga/error/vacío, imagen faltante y contenido externo seguro; sin inventar datos ni traducir IGDB. |
| [ RESOLVED ] **GB-009.07** — Probar y revisar catálogo | GB-009.06 | Mocks Twitch/IGDB para token, renovación, éxito/error/429, filtros combinados/paginación/modal, revisión visual móvil/escritorio, claro/oscuro y EN/ES; CI verde. |

### [ RESOLVED ] GB-010 — Cuenta, sesión y perfil en Frontend

Ámbito: `GameBook.Frontend`. **Usar `interface-design`** para formularios, navegación y estados. Cierre del grupo: registro, login, perfil y logout contra AuthUser real con sesión coherente.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-010.01** — Crear formularios de registro/login | GB-006 | Campos y errores acordados, contraseña confirmada, regla de ocho caracteres/mayúscula/número/signo; registro exitoso navega a login sin sesión automática. |
| [ RESOLVED ] **GB-010.02** — Gestionar token y cliente AuthUser | GB-010.01 | JWT solo en `sessionStorage`, Bearer en peticiones protegidas, estado de usuario derivado de sesión válida; el token OAuth de IGDB permanece en el servidor y no se confunde con el JWT. |
| [ RESOLVED ] **GB-010.03** — Construir navegación y perfil | GB-010.02 | Navbar visitante con login/registro; usuario con perfil/favoritos; perfil cambia solo contraseña solicitando la actual. |
| [ RESOLVED ] **GB-010.04** — Manejar expiración, revocación y logout | GB-010.03 | `401` y una hora cierran sesión, cambio de contraseña limpia token, logout vuelve al catálogo público; `503` no se muestra como credenciales inválidas. |
| [ RESOLVED ] **GB-010.05** — Integrar AuthUser real en local | GB-007, GB-010.04 | Configurar `NEXT_PUBLIC_AUTHUSER_URL` con la URL local accesible por el navegador y `CORS_ALLOWED_ORIGINS` de AuthUser con el origen exacto del Frontend local; reconstruir Frontend tras cambiar una variable `NEXT_PUBLIC_`. Verificar contratos, CORS y errores localmente; registro→login→perfil→cambio de contraseña→nuevo login, con revocación confirmada. Producción espera a `GB-012`. |
| [ RESOLVED ] **GB-010.06** — Probar y revisar cuenta | GB-010.05 | Unitarias/integración frontend, formularios/estados y traducciones completas EN/ES, temas, móvil/escritorio y CI verde. |

### [ RESOLVED ] GB-011 — Favoritos en Frontend

Ámbito: `GameBook.Frontend`. **Usar `interface-design`** en tarjetas, modales, tooltips y confirmaciones. Cierre del grupo: experiencia personal completa contra Game, con fallback IGDB.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-011.01** — Preparar acciones según sesión | GB-009, GB-010 | Estrella/corazón y acción en modal; visitante ve «Para guardar en favoritos debes:»/equivalente EN y botones de login/registro; no se guarda sin sesión. |
| [ RESOLVED ] **GB-011.02** — Construir vista personal | GB-011.01 | Tarjetas locales, mismos tres filtros/sugerencias y desplazamiento infinito que catálogo, vacíos/carga/fin, sin pedir detalle IGDB para cada tarjeta. |
| [ RESOLVED ] **GB-011.03** — Confirmar guardado y eliminación | GB-011.01, GB-011.02 | Confirmaciones contextuales en tarjeta/modal, éxito/error/duplicado, eliminación propia y actualización visible sin duplicados. |
| [ RESOLVED ] **GB-011.04** — Sincronizar al abrir detalle | GB-011.02, GB-009 | Detalle obtenido de IGDB y snapshot actualizado solo si responde; ante fallo permanecen tarjeta/favorito y se informa indisponibilidad, con opción de eliminar. |
| [ RESOLVED ] **GB-011.05** — Integrar Game real en local | GB-008, GB-011.03, GB-011.04 | Configurar `NEXT_PUBLIC_GAME_URL` con la URL local accesible por el navegador y `CORS_ALLOWED_ORIGINS` de Game con el origen exacto del Frontend local; reconstruir Frontend tras cambiar una variable `NEXT_PUBLIC_`. Probar mismo JWT AuthUser Bearer hacia Game, operaciones reales, filtros, sugerencias, revocación, CORS y `503` localmente. Producción espera a `GB-012`. |
| [ RESOLVED ] **GB-011.06** — Probar y revisar favoritos | GB-011.05 | Pruebas UI/API con estados visitante/autenticado, confirmaciones, aislamiento, IGDB fallido, EN/ES, temas y móvil/escritorio; CI verde. |

## 5. Integración, publicación y documentación final

### [ RESOLVED ] GB-012 — Integrar localmente y publicar las tres aplicaciones

Ámbito: coordinación y tres repositorios de aplicaciones. Cierre del grupo: criterios 1–16 demostrados en producción y documentación de cada repo preparada para reclutadores.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-012.10** — Entregar Docker Compose de integración local | GB-004.01, GB-005.01, GB-006.02 | Versionar la orquestación en `GameBook.Frontend` y, si hacen falta, Dockerfiles de desarrollo en los repos dueños mediante PR registrados. Con clones hermanos, `docker compose up` levanta los tres servicios con puertos, variables locales privadas, health checks/esperas y apagado documentados en README EN/ES; Neon `develop` externo, sin PostgreSQL alternativo, sin migraciones implícitas ni secretos de producción. Verificar arranque limpio, reconstrucción y conectividad básica; CI/PR y merge del autor antes de cerrar. |
| [ RESOLVED ] **GB-012.01** — Ejecutar contratos y flujo transversal **local** | GB-007, GB-008, GB-009, GB-010, GB-011, GB-012.10 | Desde Docker Compose, registro→login→guardar→filtrar→detalle/sync→eliminar→cambiar contraseña→revocación→nuevo login→logout, más visita pública sin JWT. Confirmar localmente URLs, JWT, CORS, Neon `develop`, IGDB de pruebas, Swagger y Actions `develop`; corregir discrepancias antes de seguir, sin registrar secretos. **Ningún proyecto Vercel debe crearse o desplegarse para este cierre.** |
| [ RESOLVED ] **GB-012.02** — Auditar calidad y seguridad local | GB-012.01 | CORS, secretos locales aislados, datos por `sub`, fallo cerrado AuthUser, Twitch/IGDB caído o limitado, renovación OAuth, accesibilidad, temas/idiomas, logs, Swagger, pruebas automatizadas y arranque Compose desde cero revisados; documentar evidencia de que el MVP completo funciona localmente. |
| [ RESOLVED ] **GB-012.03** — Preparar README bilingües de aplicaciones | GB-012.02 | `README.md` EN y `README.es.md` ES preparados y comprobados en `develop` de cada repo: propósito, arquitectura, setup `pnpm`, variables sin valores, pruebas y uso; enlaces entre idiomas válidos. Las URLs de producción se completan en `GB-012.09`. |
| [ RESOLVED ] **GB-012.04** — Configurar release-please, CI y política Git de despliegue | GB-012.03 | Tres workflows apuntan a `main`, commits Conventional Commits conservados, checks de PR/release funcionando y permisos mínimos verificados. La política de despliegue queda por proveedor: los backends se publican en Render y Frontend puede publicarse en Vercel desde `main`; no se exige configuración Vercel en los backends. |
| [ RESOLVED ] **GB-012.11** — Preparar publicación productiva por proveedor | GB-012.02, GB-012.04 | **Después** de la aceptación local, preparar el inventario de variables productivas y proveedores: Render para AuthUser/Game y Vercel para Frontend si corresponde. No imprimir valores ni usar URLs de migración en proveedores runtime. Las referencias históricas a proyectos Vercel iniciales se conservan solo como trazabilidad y no son destino de los backends. |
| [ RESOLVED ] **GB-012.05** — Migrar y publicar AuthUser en Render | GB-003, GB-004.07, GB-007, GB-012.11 | PR `develop → main` por `gh`; Action de migración `auth` en `main` con migrador limitado y resultado verificado **antes** de crear/configurar el servicio Render. Configurar AuthUser en Render con raíz correcta, variables runtime de Neon `production` y JWT productivo, sin `AUTH_DATABASE_DIRECT_URL`; no incluir configuración Vercel. Hasta que exista el dominio Frontend, CORS deniega cross-origin; el origen final se fija en `GB-012.07`. Verificar login/sesión/Swagger, función y logs en Render. |
| [ RESOLVED ] **GB-012.06** — Migrar y publicar Game en Render | GB-003, GB-005.07, GB-008, GB-012.05 | PR `develop → main` por `gh`; Action de migración `game` en `main` con migrador limitado y resultado verificado **antes** de configurar Render. Game publicado en Render con variables runtime de Neon `production`, clave pública JWT productiva y `AUTHUSER_URL` real, sin `GAME_DATABASE_DIRECT_URL`; no incluye configuración Vercel. CORS continúa denegando orígenes no configurados hasta `GB-012.07`. JWT/AuthUser, CRUD, Swagger/OpenAPI, build, arranque, logs y disponibilidad quedaron verificados. |
| [ RESOLVED ] **GB-012.07** — Publicar Frontend y CORS productivo | GB-011, GB-012.06 | PR `develop → main` por `gh`; crear Frontend sin primer build prematuro, configurar preset Next.js y solo variables Production, incluidas credenciales IGDB/Twitch productivas y `NEXT_PUBLIC_AUTHUSER_URL`/`NEXT_PUBLIC_GAME_URL` reales; **después** conectar Git/desplegar `main`, sin previews. Tras obtener el origen Frontend real, fijar `CORS_ALLOWED_ORIGINS` exacto en ambos backends, redeplegar lo necesario y probar navegador→AuthUser/Game; Client Secret y token IGDB no expuestos. |
| [ RESOLVED ] **GB-012.08** — Ejecutar aceptación de producción | GB-012.07 | Evidencia de los 16 criterios de `specs/mvp.md`, incluidos fallos externos/429, Swagger, IGDB atribuido, temas/idiomas y aislamiento; incidencias cerradas. |
| [ RESOLVED ] **GB-012.09** — Cerrar versiones, README y evidencia | GB-012.08 | release-please/tag/notas por app, `main` sincronizada a `develop`, URLs reales agregadas a ambos README de cada aplicación y evidencias de arquitectura inventariadas para Archify. |
| [ RESOLVED ] **GB-012.12** — Implementar desactivación lógica de cuentas | GB-012.09 | AuthUser conserva las cuentas con un indicador booleano de deshabilitación; Perfil permite solicitar la desactivación; los JWT de la cuenta se invalidan; login/registro muestran el estado deshabilitado en EN/ES sin duplicar ni reactivar; Game rechaza operaciones protegidas de esa cuenta; migración, contratos, pruebas, CI/PR y flujo local verificados. |
| [ RESOLVED ] **GB-012.13** — Corregir selección de sugerencias de filtros | GB-012.12 | Implementado en `GameBook.Frontend` mediante gestión del foco a nivel de campo, selección anticipada para puntero y `autocomplete="off"` en las sugerencias de catálogo; commits `3b57bcb` y `2ed3ba9`. `pnpm typecheck`, `pnpm lint`, `pnpm test` (23 archivos/68 pruebas), `pnpm build` y comprobación visual de clic/teclado/autocompletado pasan. PR [#38](https://github.com/CarlosSV923/GameBook.Frontend/pull/38) fusionado a `develop` como `fd3083876cfd719886fa08cf516bd7e92a21f754`; PR [#39](https://github.com/CarlosSV923/GameBook.Frontend/pull/39) fusionado a `main` como `2cd4b0badb89265d3ebfdf1ca039acccb9ba6731`; release-please [#40](https://github.com/CarlosSV923/GameBook.Frontend/pull/40) fusionado como `6de4f55aa439440c3fc565dce6c901fe1d426a61`, tag `gamebook-frontend-v0.4.1`; todos los checks `repository-baseline` verificaron `SUCCESS`. El autor avisó y el agente comprobó los merges, destinos, entregable y CI el 2026-09-27 UTC. |
| [ RESOLVED ] **GB-012.14** — Corregir actualización productiva de contraseña | GB-012.13 | `INVALID_CREDENTIALS` sigue siendo un `401` válido cuando la contraseña actual no coincide, pero el Frontend ya no cierra la sesión por ese error y conserva los errores de formulario; los códigos de sesión inválida sí revocan la sesión. Tras un cambio exitoso se muestra un modal EN/ES que explica el cierre de sesión y ofrece iniciar sesión de nuevo. PR [#41](https://github.com/CarlosSV923/GameBook.Frontend/pull/41) y promoción [#42](https://github.com/CarlosSV923/GameBook.Frontend/pull/42) fusionados; CI, contenido final en `main` y evidencia de la subtarea verificados. |
| [ RESOLVED ] **GB-012.17** — Mostrar y confirmar estado de favoritos en catálogo y detalle | GB-012.16 | Para usuarios autenticados, marcar en el catálogo completo los juegos que ya pertenecen a sus favoritos y mantener el mismo estado al abrir el modal de detalle. Tras agregar un juego desde el catálogo, solicitar confirmación antes de eliminarlo; el detalle debe mostrarlo como agregado y reutilizar exactamente el modal de confirmación de eliminación vigente, con textos EN/ES, antes de ejecutar la eliminación. Verificar que el estado se sincroniza entre catálogo y detalle, sin cambiar el comportamiento público para usuarios no autenticados. |
| [ RESOLVED ] **GB-012.18** — Expresar paginación de favoritos con RxJS | GB-012.17 | Sustituir el bucle `do...while` de la carga completa de favoritos por un flujo RxJS con `expand`, manteniendo solicitudes secuenciales, parada por `hasNext`, cancelación por `AbortSignal`, deduplicación de IDs y la API `Promise<Set<number>>`. Verificar que la experiencia y el contrato de favoritos no cambian. |
| [ RESOLVED ] **GB-012.19** — Normalizar extensiones de imports TypeScript en AuthUser | GB-012.18 | Sustituir únicamente las extensiones `.js` de imports relativos por `.ts` en `GameBook.Microservice.AuthUser`, conservando imports de paquetes y la funcionalidad/runtime. Verificar tests, E2E, lint, tipado, build, formato y diff; registrar PR y merge antes de resolver. |
| [ RESOLVED ] **GB-012.20** — Normalizar extensiones de imports TypeScript en Game | GB-012.18 | Sustituir únicamente las extensiones `.js` de imports relativos por `.ts` en `GameBook.Microservice.Game`, conservando imports de paquetes y la funcionalidad/runtime. Verificar tests, E2E, lint, tipado, build, formato y diff; registrar PR y merge antes de resolver. |
| [ RESOLVED ] **GB-012.21** — Notificar espera de arranque de servicios Render en Frontend | GB-012.20 | En `GameBook.Frontend`, mostrar un alert bilingüe accesible cuando el primer intento de un healthcheck de AuthUser o Game falle; ocultarlo con transición gradual si el siguiente intento responde correctamente, o automáticamente a los 30 segundos, y permitir cierre manual. Si el primer healthcheck responde correctamente, no mostrarlo. Mantener la lógica de reintentos, contratos y experiencia existente. Verificar claro/oscuro, EN/ES, responsive, `prefers-reduced-motion`, foco/teclado/ARIA, tests, typecheck, lint, build, formato y PR/merge. |

### [ RESOLVED ] GB-013 — Documentar arquitectura final con Archify

Ámbito: coordinación; **usar el skill `archify` en las subtareas de generación, validación y entrega**. Cierre del grupo: diagrama general EN/ES y diagramas específicos de cada repositorio basados en hechos, validados, enlazados desde los README de cada aplicación y listos para publicarse en `GameBook.System`.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-013.01** — Reunir evidencia arquitectónica | GB-012 | Código, OpenAPI, migraciones, variables por nombre, dominios/despliegues y flujos JWT/IGDB OAuth verificados; diferencias con plan señaladas. |
| [ RESOLVED ] **GB-013.02** — Generar diagrama general EN | GB-013.01 | Fuente Archify y HTML inglés con Frontend, AuthUser, Game, Neon, Twitch OAuth, IGDB, límites y relaciones reales; sin secretos. |
| [ RESOLVED ] **GB-013.03** — Generar variante ES equivalente | GB-013.02 | Fuente/HTML español con **misma topología** y etiquetas propias traducidas; registrar que controles fijos del visor permanecen en inglés. |
| [ RESOLVED ] **GB-013.04** — Validar cada variante | GB-013.02, GB-013.03 | Validación `archify` exigida por skill para EN y ES sin errores advertidos; corregir solo diagnósticos reales. |
| [ RESOLVED ] **GB-013.05** — Entregar y revisar visualmente | GB-013.04 | `deliver` y `visual-check` para cada HTML, comprobación visual de tamaños requeridos y recibos/limitaciones honestos; no confundir validación automática con revisión perceptual. |
| [ RESOLVED ] **GB-013.06** — Preparar artefactos finales | GB-013.05 | Fuentes y HTML congelados, nombres EN/ES consistentes y paquete listo para la publicación documental única. |
| [ RESOLVED ] **GB-013.07** — Diagramas del repositorio Frontend | GB-013.06 | Fuentes, HTML e imágenes oscuras EN/ES en `GameBook.Frontend/architecture/`; README bilingües enlazados por idioma; GitHub Pages evaluado y publicado si es viable. |
| [ RESOLVED ] **GB-013.08** — Diagramas del repositorio AuthUser | GB-013.06 | Fuentes, HTML e imágenes oscuras EN/ES en `GameBook.Microservice.AuthUser/architecture/`; README bilingües enlazados por idioma; GitHub Pages evaluado y publicado si es viable. |
| [ RESOLVED ] **GB-013.09** — Diagramas del repositorio Game | GB-013.06 | Fuentes, HTML e imágenes oscuras EN/ES en `GameBook.Microservice.Game/architecture/`; README bilingües enlazados por idioma; GitHub Pages evaluado y publicado si es viable. |

#### GB-013.01 — Evidencia arquitectónica reunida

- Estado: `[ RESOLVED ]` (2026-09-27 UTC). Dependencia `GB-012` verificada como completada en sus subtareas de integración, publicación y evidencia; al cierre de esta subtarea, `GB-013.02`–`GB-013.06` estaban `[ NEW ]`.
- Método y límite: se inspeccionaron directamente los tres repositorios, sus contratos, código fuente, esquemas/migraciones Prisma, plantillas `.env.example`, README bilingües, workflows y commits actuales de `main`. El servicio `codebase-memory` no estuvo disponible en esta sesión porque su transporte cerró; por eso no se usó como evidencia y no se afirma una auditoría graph-completa. No se leyeron valores de secretos, no se generó ningún diagrama ni HTML y no se creó `GameBook.System`.
- Repositorios y estado final en GitHub: `GameBook.Frontend` `main`=`f9735f1d4b4b6fe0dd983e63538af5da9b2b6eb4`; `GameBook.Microservice.AuthUser` `main`=`cba3b2a2c751086f4e1214384404348cafec2632`; `GameBook.Microservice.Game` `main`=`23d9ed9232958fbf422c760e684a8c085b04bfbc`.
- Topología implementada: el navegador consume `GameBook.Frontend` en Vercel; el Frontend llama a AuthUser y Game en Render. AuthUser persiste en PostgreSQL/Neon dentro del esquema `auth`; Game persiste en la misma base dentro del esquema `game`, con roles runtime separados. Game valida el JWT localmente y consulta `GET /v1/auth/session` de AuthUser para confirmar la sesión vigente antes de atender rutas protegidas.
- Frontend verificado: `src/app/api/igdb/` contiene las rutas server-side de catálogo, detalle, sugerencias y plataformas; `src/server/igdb/` contiene la configuración server-only, proveedor de token Twitch, cliente IGDB, límites y mapeo de errores; `src/features/api/` contiene clientes AuthUser/Game/IGDB; `src/shared/auth/session-storage.ts` conserva el JWT en `sessionStorage`; `src/shared/api/http.ts` usa Axios + RxJS y `src/shared/api/healthcheck.ts` consulta `GET /health` antes de AuthUser/Game, con 15 segundos por intento y hasta 15 reintentos adicionales.
- AuthUser verificado: `src/api/` expone autenticación, sesión, contraseña, desactivación lógica y salud; `src/application/` coordina casos de uso; `src/domain/` contiene usuarios y reglas; `src/infrastructure/cryptography/rsa-jwt.ts` firma/verifica RS256 y `src/infrastructure/persistence/prisma/` encapsula Prisma. `GET /health` devuelve HTTP `200` y `{ status: 'ok' }`; Swagger se publica en `/docs` y `/docs/openapi.json`.
- Game verificado: `src/api/` expone salud y favoritos con `JwtAuthGuard`; `src/application/` contiene los casos de uso de favoritos; `src/domain/` contiene el modelo/repositorio; `src/infrastructure/auth/` consulta AuthUser; `src/infrastructure/cryptography/` verifica RS256; `src/infrastructure/persistence/prisma/` encapsula el esquema `game`. `GET /health` devuelve HTTP `200` y `{ status: 'ok' }`; Swagger se publica en `/docs` y `/docs/openapi.json`.
- Contratos y OpenAPI: la fuente vigente es `contracts/contract-baseline-v3.md`, con `contracts/authuser.openapi.yaml`, `contracts/game-igdb.openapi.yaml`, `contracts/igdb-mapping.md`, `contracts/persistence-igdb.md` y `contracts/persistence-access-amendment-v2.md`. Los endpoints runtime de AuthUser incluyen `/v1/auth/register`, `/v1/auth/login`, `/v1/auth/session`, `/v1/users/me/password` y `/v1/users/me`; Game expone `/v1/favorites`, sugerencias, snapshot y eliminación, con `igdbId` y Bearer del JWT de AuthUser. `contracts/game.openapi.yaml` conserva `rawgId` y se clasifica como histórico; no se usará para Archify.
- Persistencia y migraciones: AuthUser usa `src/infrastructure/persistence/prisma/migrations/20260924030000_init_auth_user` y `20260926220000_add_user_disabled_status`; `auth.User` contiene `id`, identidad, `passwordHash`, `sessionVersion` e `isDisabled`. Game usa `20260924040000_init_game_favorites`; `game.Favorite` y `game.FavoritePlatform` se identifican por `userId`/`igdbId`, con cascada entre favorito y plataformas. La enmienda v2 acredita los roles runtime `gamebook_auth_app` y `gamebook_game_app` limitados a su esquema y migradores separados; las URLs directas de migración no entran en Render/Vercel.
- Variables nominales verificadas sin valores: Frontend `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET`, `NEXT_PUBLIC_AUTHUSER_URL`, `NEXT_PUBLIC_GAME_URL`, `PORT`; AuthUser `AUTH_DATABASE_URL`, `JWT_PRIVATE_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE`, `CORS_ALLOWED_ORIGINS`, `PORT`, `AUTH_DATABASE_DIRECT_URL`; Game `GAME_DATABASE_URL`, `JWT_PUBLIC_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE`, `AUTHUSER_URL`, `CORS_ALLOWED_ORIGINS`, `PORT`, `GAME_DATABASE_DIRECT_URL`. `IGDB_CLIENT_ID`/`IGDB_CLIENT_SECRET` solo se leen desde el servidor de Next.js.
- Dominios y despliegue verificados en README y política vigente: Frontend `https://gamebook-frontend.vercel.app`; AuthUser `https://gamebook-microservice-authuser.onrender.com`; Game `https://gamebook-microservice-game.onrender.com`; Swagger productivo en `/docs` y JSON en `/docs/openapi.json`. Localmente se documentan Frontend `:3000`, AuthUser `:3001` y Game `:3002`; producción usa `main`, Render para backends y Vercel para Frontend, con Neon `production` y credenciales IGDB/Twitch server-only.
- Flujo JWT verificado: AuthUser firma RS256 con clave privada, emite claims `sub`, `ver`, `iat`, `exp`, `iss` y `aud`; Frontend envía el mismo token como `Authorization: Bearer`; Game verifica firma, vencimiento, emisor/audiencia y coherencia del `sub` contra la sesión validada por AuthUser. `sessionVersion` revoca tokens y `isDisabled` bloquea sesiones de cuentas deshabilitadas.
- Flujo IGDB OAuth verificado: las rutas públicas de Next.js llaman al cliente server-only; este obtiene/cachéa un token de aplicación mediante `https://id.twitch.tv/oauth2/token`, llama a `https://api.igdb.com/v4` con `Client-ID` y Bearer, limita concurrencia/frecuencia, renueva el token ante `401/403` y normaliza los datos antes de responder al navegador. Ningún backend de GameBook recibe credenciales IGDB.
- Diferencias y decisiones para las siguientes subtareas: el plan final exige un diagrama general EN/ES y opcionalmente el flujo JWT/favoritos; esta subtarea solo reúne hechos, por lo que no se generaron artefactos Archify. `GameBook.System` queda correctamente pendiente de `GB-014`. La documentación histórica RAWG y las referencias antiguas de Vercel Preview no representan la topología vigente; la variante EN de GB-013.02 deberá basarse en la evidencia actual y conservar la misma topología para GB-013.03.

#### GB-013.02 — Diagrama general EN generado

- Estado: `[ RESOLVED ]` (2026-09-27 UTC). Dependencia `GB-013.01` verificada como `[ RESOLVED ]`; al cierre de esta subtarea, `GB-013.03`–`GB-013.06` estaban `[ NEW ]`.
- Artefactos: [fuente Archify EN](../architecture/GameBook.System-architecture-en.json) y [HTML standalone EN](../architecture/GameBook.System-architecture-en.html). El diagrama contiene Browser, `GameBook.Frontend`, AuthUser, Game, Neon PostgreSQL, Twitch OAuth e IGDB API; muestra Vercel, Render, Neon, el límite de credenciales IGDB server-only, healthchecks, JWT/Bearer, validación de sesión Game→AuthUser y persistencia por esquemas.
- Evidencia de repositorio: la fuente declara el repositorio Frontend, revisión `f9735f1d4b4b6fe0dd983e63538af5da9b2b6eb4`, y tres referencias verificables de código (`src/app/api/igdb/games/route.ts`, `src/server/igdb/igdb-runtime-config.ts`, `src/shared/api/healthcheck.ts`). Debido a que Archify admite un único `meta.repository` por fuente y la arquitectura reúne tres repositorios, las evidencias directas de AuthUser/Game permanecen registradas y verificadas en GB-013.01; el diagrama no atribuye sus archivos a la revisión del Frontend.
- Validación Archify: `validate architecture --quality showcase --repo-root GameBook.Frontend` terminó `ok=true`, con las 9 comprobaciones, composición `showcase`, 0 errores, 0 advertencias, 0 cruces, 0 corredores ambiguos y 0 problemas de legibilidad.
- Entrega congelada: `deliver` terminó `ok=true`; especificación SHA-256 `6520397fb666aa2c4dd8d3e1199a30eb4ac61731bf34b73ee6d148a65fb1ff69` (5.992 bytes) y HTML SHA-256 `7bb8041d4d32ff2151f80bd737158d7f43fdb3ff9648946ed7b2e360169fe8d9` (813.632 bytes).
- Evidencia automática de navegador: `visual-check` terminó `status=pass` en 1440×900, 1600×1000, 1920×1080 y 2048×1320, sin overflow horizontal/vertical; readability, viewer chrome y capturas pasaron en claro y oscuro. El recibo quedó en [visual-check JSON](../architecture/GameBook.System-architecture-en.visual-check.json), con contact sheet en [visual-check HTML](../architecture/GameBook.System-architecture-en.visual-check.html).
- Revisión perceptual: inspección del agente sobre capturas 1440×900 y 2048×1320 en ambos temas confirmó jerarquía, contraste, relaciones distinguibles, límites de despliegue legibles, tarjetas de evidencia completas y ausencia visible de recortes o solapamientos. Esta revisión perceptual se mantiene separada de la validación automática.

#### GB-013.03 — Variante ES equivalente generada

- Estado: `[ RESOLVED ]` (2026-09-27 UTC). Dependencia `GB-013.02` verificada como `[ RESOLVED ]`; al cierre de esta subtarea, `GB-013.04`–`GB-013.06` estaban `[ NEW ]`.
- Artefactos: [fuente Archify ES](../architecture/GameBook.System-architecture-es.json) y [HTML standalone ES](../architecture/GameBook.System-architecture-es.html). La variante conserva los mismos componentes, IDs, posiciones, tamaños, límites, conexiones, rutas y relaciones del diagrama EN; únicamente traduce las etiquetas y el contenido authored al español.
- Evidencia de repositorio: la fuente declara el repositorio Frontend, revisión `f9735f1d4b4b6fe0dd983e63538af5da9b2b6eb4`, y las mismas tres referencias verificables de código (`src/app/api/igdb/games/route.ts`, `src/server/igdb/igdb-runtime-config.ts`, `src/shared/api/healthcheck.ts`). Debido a que Archify admite un único `meta.repository` por fuente y la arquitectura reúne tres repositorios, las evidencias directas de AuthUser/Game permanecen registradas y verificadas en GB-013.01.
- Idioma y visor: `meta.locale` se omitió intencionalmente porque el skill de Archify solo admite `en` y `zh-CN`; por ello los controles fijos del Viewer y el atributo `<html lang>` permanecen en inglés. La misma decisión queda declarada en una tarjeta del diagrama. Esto no altera el contenido authored español ni la topología equivalente.
- Validación Archify: `validate architecture --quality showcase --repo-root GameBook.Frontend` terminó `ok=true`, con las 9 comprobaciones, composición `showcase`, 0 errores, 0 advertencias, 0 cruces, 0 corredores ambiguos, 0 problemas de rutas/clearance y 0 problemas de legibilidad.
- Entrega congelada: `deliver` terminó `ok=true`; especificación SHA-256 `c84fdc9520b0596c376110c0674248482deeca2e100cf97d76eaee7c875396d5` (6.162 bytes) y HTML SHA-256 `c4dfc0986cfdfdaf496955e99a15e2df713dcb8fde24897b5db858aa8a0686db` (813.934 bytes). El JSON fue parseado nuevamente después de la entrega.
- Evidencia automática de navegador: `visual-check` terminó `status=pass` para 1440×900, 1600×1000, 1920×1080 y 2048×1320, en claro y oscuro; todas las capturas quedaron sin overflow horizontal/vertical y pasaron readability, viewer chrome y containment. El recibo está en [visual-check JSON](../architecture/GameBook.System-architecture-es.visual-check.json), el contact sheet en [visual-check HTML](../architecture/GameBook.System-architecture-es.visual-check.html) y las capturas generadas incluyen los cuatro viewports/temas.
- Revisión perceptual: inspección del agente mediante `view_image` sobre las capturas 1440×900 y 2048×1320 en ambos temas confirmó contenido español legible, contraste y jerarquía consistentes, topología equivalente a EN, relaciones distinguibles, ausencia visible de recortes/solapamientos y controles fijos del visor en inglés. Esta revisión perceptual se mantiene separada de la validación automática.

#### GB-013.04 — Variantes validadas

- Estado: `[ RESOLVED ]` (2026-09-28 UTC). Dependencias `GB-013.02` y `GB-013.03` verificadas como `[ RESOLVED ]`; al cierre de esta subtarea, `GB-013.05` y `GB-013.06` estaban `[ NEW ]`.
- Validación Archify EN: `node bin/archify.mjs validate architecture tasks/GameBook.System-architecture-en.json --quality showcase --repo-root GameBook.Frontend --json` terminó con `ok=true`; las 9 comprobaciones pasaron, perfil `showcase`, 0 errores, 0 advertencias, 0 cruces, 0 corredores ambiguos, 0 problemas de clearance/rutas y 0 problemas de legibilidad.
- Validación Archify ES: `node bin/archify.mjs validate architecture tasks/GameBook.System-architecture-es.json --quality showcase --repo-root GameBook.Frontend --json` terminó con `ok=true`; las 9 comprobaciones pasaron, perfil `showcase`, 0 errores, 0 advertencias, 0 cruces, 0 corredores ambiguos, 0 problemas de clearance/rutas y 0 problemas de legibilidad.
- Evidencia adicional: la raíz de evidencia usada fue el repositorio Git real `GameBook.Frontend`; no se modificaron las fuentes ni los HTML. `GB-013.04` queda cerrada sin adelantar `GB-013.05`.

#### GB-013.05 — Entrega y revisión visual

- Estado: `[ RESOLVED ]` (2026-09-28 UTC). Dependencia `GB-013.04` verificada como `[ RESOLVED ]`; al cierre de esta subtarea, `GB-013.06` estaba `[ NEW ]`.
- Dueño: Codex (coordinación), usando exclusivamente las fuentes Archify ya congeladas de GB-013.02 y GB-013.03.
- Dependencia: `GB-013.04`, que figura `[ RESOLVED ]`; `GB-013.06` no se inicia.
- Alcance: ejecutar `deliver` para EN y ES, después `visual-check` sobre cada HTML entregado en 1440×900, 1600×1000, 1920×1080 y 2048×1320; registrar recibos SHA-256, sidecars, estado automático y revisión perceptual separadamente.
- Restricciones: no modificar las fuentes JSON salvo que una entrega falle por un diagnóstico real; no confundir `visual-check` con revisión perceptual; no iniciar `GB-013.06`.
- Entrega EN: `tasks/GameBook.System-architecture-en.html`, `deliver` terminó con `ok=true`, especificación SHA-256 `6520397fb666aa2c4dd8d3e1199a30eb4ac61731bf34b73ee6d148a65fb1ff69` (5.992 bytes) y artefacto SHA-256 `7bb8041d4d32ff2151f80bd737158d7f43fdb3ff9648946ed7b2e360169fe8d9` (813.632 bytes); validación 9/9 `showcase`, 0 errores y 0 advertencias.
- Entrega ES: `tasks/GameBook.System-architecture-es.html`, `deliver` terminó con `ok=true`, especificación SHA-256 `c84fdc9520b0596c376110c0674248482deeca2e100cf97d76eaee7c875396d5` (6.162 bytes) y artefacto SHA-256 `c4dfc0986cfdfdaf496955e99a15e2df713dcb8fde24897b5db858aa8a0686db` (813.934 bytes); validación 9/9 `showcase`, 0 errores y 0 advertencias.
- Evidencia automática EN: `visual-check` terminó `status=pass`, `evidenceKind=automated-browser`, Chrome disponible, estado `read/still`, sin overflow horizontal o vertical en 1440×900, 1600×1000, 1920×1080 y 2048×1320; readability y viewer chrome pasaron. Capturas claro/oscuro de los extremos y recibo quedaron en [EN visual-check JSON](../architecture/GameBook.System-architecture-en.visual-check.json) y [EN contact sheet](../architecture/GameBook.System-architecture-en.visual-check.html).
- Evidencia automática ES: `visual-check` terminó `status=pass`, `evidenceKind=automated-browser`, Chrome disponible, estado `read/still`, sin overflow horizontal o vertical en 1440×900, 1600×1000, 1920×1080 y 2048×1320; readability y viewer chrome pasaron. Capturas claro/oscuro de los extremos y recibo quedaron en [ES visual-check JSON](../architecture/GameBook.System-architecture-es.visual-check.json) y [ES contact sheet](../architecture/GameBook.System-architecture-es.visual-check.html).
- Revisión perceptual: `visual_review: passed`, `correction_rounds: 0`. Inspeccioné las ocho capturas de extremos EN/ES, claro/oscuro, en 1440×900 y 2048×1320; confirmé jerarquía, contraste, topología, relaciones distinguibles, tarjetas completas y ausencia visible de recortes o solapamientos. Esta revisión humana/imagen-capaz se mantiene separada de `browser_evidence: passed`.
- GB-013.05 queda cerrada sin modificar los JSON fuente ni adelantar `GB-013.06`.

#### GB-013.06 — Artefactos finales preparados

- Estado: `[ RESOLVED ]` (2026-09-28 UTC). Dependencia `GB-013.05` verificada como `[ RESOLVED ]`; `GB-014` permanece `[ NEW ]` y no se inicia.
- Dueño: Codex (coordinación), con alcance limitado al paquete documental Archify de EN/ES.
- Alcance y evidencia: inventario de 16 artefactos esperados completado, sin faltantes ni archivos inesperados; nombres `GameBook.System-architecture-en*` y `GameBook.System-architecture-es*` consistentes, y `meta.output` coincide con cada HTML entregado. Las dos fuentes y sus HTML conservan `quality: showcase`; EN registra `meta.locale: en` y ES omite intencionalmente `meta.locale` porque el visor solo admite `en`/`zh-CN`, decisión ya documentada en GB-013.03.
- Integridad congelada: EN JSON SHA-256 `6520397fb666aa2c4dd8d3e1199a30eb4ac61731bf34b73ee6d148a65fb1ff69` (5.992 bytes), HTML `7bb8041d4d32ff2151f80bd737158d7f43fdb3ff9648946ed7b2e360169fe8d9` (813.632 bytes); ES JSON `c84fdc9520b0596c376110c0674248482deeca2e100cf97d76eaee7c875396d5` (6.162 bytes), HTML `c4dfc0986cfdfdaf496955e99a15e2df713dcb8fde24897b5db858aa8a0686db` (813.934 bytes). Los recibos de entrega enlazan los hashes/tamaños correspondientes y los recibos `visual-check` están en `status=pass`.
- Paridad EN/ES: 7 componentes, IDs, extremos de conexiones y envolturas de límites coinciden; las referencias de repositorio y revisiones son equivalentes. La revisión automática y perceptual ya registrada en GB-013.05 permanece válida; no se modificaron fuentes JSON ni HTML después de su entrega.
- Restricciones: no publicar `GameBook.System`, no modificar fuentes congeladas, no crear documentación general de GB-014 ni incorporar secretos, cachés o dependencias.

#### GB-013.07 — Diagramas del repositorio Frontend

- Estado: `[ RESOLVED ]` (2026-09-28 UTC).
- Dueño previsto: Codex (coordinación), con cambios limitados a `GameBook.Frontend` y su documentación enlazada.
- Dependencia: `GB-013.06`, que figura `[ RESOLVED ]`; las subtareas `GB-013.08` y `GB-013.09` son independientes y no se adelantan.
- Alcance: ejecutar `archify` sobre la arquitectura efectivamente implementada de `GameBook.Frontend`, incluyendo sus límites Next.js navegador/servidor, rutas server-side de IGDB/Twitch, clientes AuthUser/Game, Axios + RxJS y healthchecks, sin exponer secretos ni sustituir la evidencia del diagrama general.
- Entregables: crear `GameBook.Frontend/architecture/` y conservar fuentes JSON y HTML standalone bilingües con nombres `GameBook.Frontend-architecture-en.*` y `GameBook.Frontend-architecture-es.*`; ambas variantes deben tener la misma topología y etiquetas/contenido authored en su idioma.
- README: generar una imagen PNG de cada variante en tema oscuro a partir de una captura validada, con nombres `GameBook.Frontend-architecture-en-dark.png` y `GameBook.Frontend-architecture-es-dark.png`; cada README debe mostrar y enlazar únicamente la imagen y el HTML interactivo publicado de su propio idioma, sin enlaces al HTML fuente del repositorio ni a la variante opuesta.
- GitHub Pages: evaluar la viabilidad con el repositorio y permisos existentes; si es viable, publicar los dos HTML estáticos y usar sus URLs por idioma en los README; si no es viable, conservar los HTML en `architecture/`, enlazarlos con rutas relativas y registrar la limitación sin bloquear el resto de criterios.
- Cierre: `validate`, `deliver` y `visual-check` de Archify pasan para EN/ES en 1440×900, 1600×1000, 1920×1080 y 2048×1320, incluyendo claro/oscuro; revisar perceptualmente las capturas oscuras, validar enlaces/imágenes de ambos README, ejecutar CI y completar el flujo de PR del repositorio. Registrar por separado validación automática, revisión visual y resultado de Pages.
- Restricciones: no modificar la funcionalidad de Frontend, no alterar los artefactos generales congelados de GB-013.02–GB-013.06, no crear `GameBook.System` ni iniciar `GB-014`.

- Inicio y recurso: 2026-09-28 UTC, rama `feature/013-07/frontend-architecture` en `GameBook.Frontend`; dependencia `GB-013.06` verificada como `[ RESOLVED ]` y `GB-013.08`/`GB-013.09` no iniciadas.
- Entregables: commit `4d0e815` añade `architecture/` con fuentes JSON EN/ES, HTML standalone, recibos `visual-check`, capturas de los cuatro viewports y copias oscuras para README; el ajuste actual restringe `README.md` y `README.es.md` a la imagen y al HTML interactivo publicado de su idioma correspondiente.
- Archify EN: `validate` y `deliver` pasaron 9/9 `showcase`, 0 errores y 0 advertencias; fuente SHA-256 `18857636a52a7b21164b2e809597f4d8e11697905deb28dbe3bc591265bab3a4` (5.700 bytes) y HTML `bc9a90137bac42156ff121d90638b159ad6fcae46d81b245af8089785e03b8db` (816.872 bytes).
- Archify ES: `validate` y `deliver` pasaron 9/9 `showcase`, 0 errores y 0 advertencias; fuente SHA-256 `944d3b0b7ed028bedec2d2bf55d741a111a7c2838ff4fdb79b81bf07405479b7` (6.096 bytes) y HTML `1b76b81a4a978c4c8f2490d6ebf82de94c043b166b4126eb92834d2a2f60c211` (817.686 bytes). La variante ES conserva el contenido authored traducido y deja los controles fijos del visor en inglés.
- Evidencia de navegador: ambos `visual-check` terminaron `status=pass`, Chrome disponible, sin overflow en 1440×900, 1600×1000, 1920×1080 y 2048×1320, con capturas claras/oscuras y controles del visor aprobados. La revisión perceptual de EN/ES oscuro en 1440×900 y 2048×1320 confirmó legibilidad, contraste, topología, relaciones y ausencia de recortes/solapamientos.
- GitHub Pages: el repositorio público confirmó `has_pages=false` inicialmente; se habilitó mediante API en modo `workflow`, con URL base `https://carlossv923.github.io/GameBook.Frontend/`. El workflow `.github/workflows/architecture-pages.yml` publica `architecture/` desde `main`; la URL aún no contiene los HTML porque el PR no ha llegado a `main`.
- Calidad del repositorio: `pnpm test` pasó con 27 archivos y 80 pruebas; `pnpm lint`, `pnpm typecheck` y `pnpm build` pasaron. `pnpm format:check` sigue reportando 79 archivos preexistentes fuera del alcance; no se modificaron para esta subtarea. El inventario de artefactos, enlaces de ambos README y `git diff --check` pasaron.
- Ajuste de README (2026-09-28 UTC): el commit `0e7e52b` elimina los enlaces al HTML fuente y a la variante opuesta; cada README muestra únicamente la imagen y el HTML interactivo publicado de su idioma, y la imagen también enlaza a esa URL de GitHub Pages. El PR [Frontend #57](https://github.com/CarlosSV923/GameBook.Frontend/pull/57), sobre la misma rama, fue fusionado hacia `develop` a las 02:43:43 UTC con merge `bd7bf1e0f03bcc16cfddea6a98a3c67d46418397`; ambos checks `repository-baseline` terminaron `SUCCESS`.
- PR de implementación verificado: [Frontend #55](https://github.com/CarlosSV923/GameBook.Frontend/pull/55), `feature/013-07/frontend-architecture` → `develop`, fusionado el 2026-09-28 a las 02:32:02 UTC con merge `6634ddbdcf5de325a5535f4f59a8cc89cb14b492`; ambos checks `repository-baseline` terminaron `SUCCESS`.
- Promoción inicial verificada: [Frontend #56](https://github.com/CarlosSV923/GameBook.Frontend/pull/56), `develop` → `main`, fusionada el 2026-09-28 a las 02:34:46 UTC con merge `f3d0d5b344a5b1693f88d0bcc30df41e83676826`; ambos checks `repository-baseline` terminaron `SUCCESS`.
- Cierre verificado (2026-09-28 UTC): el PR de promoción del ajuste [Frontend #58](https://github.com/CarlosSV923/GameBook.Frontend/pull/58), `develop` → `main`, fue fusionado a las 02:45:25 UTC con merge `7e9f4eb2862b5832c466f3b65048a814a75977ed`; ambos checks `repository-baseline` terminaron `SUCCESS`. `main` fue verificado en ese commit y las variantes EN/ES de GitHub Pages respondieron HTTP 200 con contenido Archify. GB-013.07 queda `[ RESOLVED ]`.

#### GB-013.08 — Diagramas del repositorio AuthUser

- Estado: `[ RESOLVED ]` (2026-09-28 UTC).
- Dueño previsto: Codex (coordinación), con cambios limitados a `GameBook.Microservice.AuthUser` y su documentación enlazada.
- Dependencia: `GB-013.06`, que figura `[ RESOLVED ]`; `GB-013.07` y `GB-013.09` pueden ejecutarse de forma independiente.
- Alcance: ejecutar `archify` sobre la arquitectura efectivamente implementada de `GameBook.Microservice.AuthUser`, incluyendo API NestJS, capas de aplicación/dominio/infraestructura, Prisma y esquema `auth`, JWT RS256, desactivación lógica y healthcheck, sin incluir valores de secretos.
- Entregables: crear `GameBook.Microservice.AuthUser/architecture/` y conservar fuentes JSON y HTML standalone bilingües con nombres `GameBook.Microservice.AuthUser-architecture-en.*` y `GameBook.Microservice.AuthUser-architecture-es.*`; ambas variantes deben tener la misma topología y etiquetas/contenido authored en su idioma.
- README: generar una imagen PNG de cada variante en tema oscuro a partir de una captura validada, con nombres `GameBook.Microservice.AuthUser-architecture-en-dark.png` y `GameBook.Microservice.AuthUser-architecture-es-dark.png`; cada README debe mostrar y enlazar únicamente la imagen y el HTML interactivo publicado de su propio idioma, sin enlaces al HTML fuente del repositorio ni a la variante opuesta.
- GitHub Pages: evaluar la viabilidad con el repositorio y permisos existentes; si es viable, publicar los dos HTML estáticos y usar sus URLs por idioma en los README; si no es viable, conservar los HTML en `architecture/`, enlazarlos con rutas relativas y registrar la limitación sin bloquear el resto de criterios.
- Cierre: `validate`, `deliver` y `visual-check` de Archify pasan para EN/ES en 1440×900, 1600×1000, 1920×1080 y 2048×1320, incluyendo claro/oscuro; revisar perceptualmente las capturas oscuras, validar enlaces/imágenes de ambos README, ejecutar CI y completar el flujo de PR del repositorio. Registrar por separado validación automática, revisión visual y resultado de Pages.
- Restricciones: no modificar endpoints, migraciones ni comportamiento de AuthUser, no alterar los artefactos generales congelados de GB-013.02–GB-013.06, no crear `GameBook.System` ni iniciar `GB-014`.
- Inicio y recurso: 2026-09-28 UTC, rama `feature/013-08/authuser-architecture` en `GameBook.Microservice.AuthUser`; `GB-013.06` verificada como `[ RESOLVED ]` y `origin/develop` actualizado al merge del healthcheck `33a7c6438478ae542c9f8fc36afbf2cad3ff0b49`.
- Entregables: commit `a0c4bf3` añade fuentes JSON EN/ES, HTML standalone, recibos y capturas `visual-check`, además de `GameBook.Microservice.AuthUser-architecture-en-dark.png` y `...-es-dark.png`. Cada README enlaza únicamente la imagen y el HTML interactivo publicado de su idioma; no se enlazan HTML fuente ni la variante opuesta.
- Archify EN: `validate` y `deliver` pasaron 9/9 `showcase`, 0 errores y 0 advertencias; fuente SHA-256 `5a46aeac0b5ba8d3f0f42f93e9db22cba07bcfead49f2b1b6727011ea817c807` (6.847 bytes) y HTML `cbab340ab5b91733e482db34e635cc0f6c5584420004d8baf13f27306773b9a3` (818.104 bytes).
- Archify ES: `validate` y `deliver` pasaron 9/9 `showcase`, 0 errores y 0 advertencias; fuente SHA-256 `b0200f23b45b5cabfb755811e1ac1df678f9b37a98ec61b57b2ba2c742e8afb3` (7.007 bytes) y HTML `b53f809d5cf0623275bcd605d89b7ceb358603658e1ddbe8b356f1d74be72ff1` (818.545 bytes). La variante ES omite `meta.locale`, por lo que los controles fijos del visor permanecen en inglés conforme al contrato de Archify.
- Evidencia de navegador y revisión perceptual: ambos `visual-check` terminaron `status=pass` con Chrome disponible, sin overflow en 1440×900, 1600×1000, 1920×1080 y 2048×1320; capturas claras/oscuras y controles del visor aprobados. La inspección de las capturas oscuras EN/ES en 1440×900 y 2048×1320 confirmó legibilidad, contraste, topología, relaciones y ausencia de recortes/solapamientos.
- GitHub Pages: el repositorio público confirmó Pages inicialmente deshabilitado; se habilitó mediante API en modo `workflow`, con URL base `https://carlossv923.github.io/GameBook.Microservice.AuthUser/`. El workflow `.github/workflows/architecture-pages.yml` publica `architecture/` desde `main`; la publicación final queda pendiente de la promoción a `main`.
- Calidad del repositorio: `pnpm test` pasó con 15 archivos y 59 pruebas; `pnpm test:e2e` con 7 archivos y 32 pruebas; `pnpm lint`, `pnpm exec tsc --noEmit -p tsconfig.build.json`, `pnpm build` y `git diff --check` pasaron. Lint conserva advertencias `unbound-method` existentes en pruebas, sin errores.
- PR de implementación verificado: [AuthUser #37](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/37), `feature/013-08/authuser-architecture` → `develop`, fusionado el 2026-09-28 a las 03:01:01 UTC con merge `f14014c6e34cf1a3075d065cfb445097e61749a5`; ambos checks `repository-baseline` terminaron `SUCCESS`.
- Cierre verificado (2026-09-28 UTC): el PR de promoción [AuthUser #38](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/38), `develop` → `main`, fue fusionado a las 03:03:36 UTC con merge `12618e85e5404d05cc7a515621d58797f58bedfd`; ambos checks `repository-baseline` terminaron `SUCCESS`. `main` fue verificado en ese commit y las variantes EN/ES de GitHub Pages respondieron HTTP 200 con contenido Archify. GB-013.08 queda `[ RESOLVED ]`.

#### GB-013.09 — Diagramas del repositorio Game

- Estado: `[ RESOLVED ]` (2026-09-28 UTC).
- Dueño previsto: Codex (coordinación), con cambios limitados a `GameBook.Microservice.Game` y su documentación enlazada.
- Dependencia: `GB-013.06`, que figura `[ RESOLVED ]`; `GB-013.07` y `GB-013.08` pueden ejecutarse de forma independiente.
- Alcance: ejecutar `archify` sobre la arquitectura efectivamente implementada de `GameBook.Microservice.Game`, incluyendo API NestJS, capas de aplicación/dominio/infraestructura, Prisma y esquema `game`, validación JWT/sesión con AuthUser, favoritos y healthcheck, sin incluir valores de secretos.
- Entregables: crear `GameBook.Microservice.Game/architecture/` y conservar fuentes JSON y HTML standalone bilingües con nombres `GameBook.Microservice.Game-architecture-en.*` y `GameBook.Microservice.Game-architecture-es.*`; ambas variantes deben tener la misma topología y etiquetas/contenido authored en su idioma.
- README: generar una imagen PNG de cada variante en tema oscuro a partir de una captura validada, con nombres `GameBook.Microservice.Game-architecture-en-dark.png` y `GameBook.Microservice.Game-architecture-es-dark.png`; cada README debe mostrar y enlazar únicamente la imagen y el HTML interactivo publicado de su propio idioma, sin enlaces al HTML fuente del repositorio ni a la variante opuesta.
- GitHub Pages: evaluar la viabilidad con el repositorio y permisos existentes; si es viable, publicar los dos HTML estáticos y usar sus URLs por idioma en los README; si no es viable, conservar los HTML en `architecture/`, enlazarlos con rutas relativas y registrar la limitación sin bloquear el resto de criterios.
- Cierre: `validate`, `deliver` y `visual-check` de Archify pasan para EN/ES en 1440×900, 1600×1000, 1920×1080 y 2048×1320, incluyendo claro/oscuro; revisar perceptualmente las capturas oscuras, validar enlaces/imágenes de ambos README, ejecutar CI y completar el flujo de PR del repositorio. Registrar por separado validación automática, revisión visual y resultado de Pages.
- Restricciones: no modificar endpoints, persistencia ni comportamiento de Game, no alterar los artefactos generales congelados de GB-013.02–GB-013.06, no crear `GameBook.System` ni iniciar `GB-014`.

- Inicio y recurso: 2026-09-28 UTC, rama `feature/013-09/game-architecture` en `GameBook.Microservice.Game`, basada en `origin/develop` `c0ce96a7be59c5cec37607754eae3552f77c8ce7`; `GB-013.06` verificada como `[ RESOLVED ]` y `GB-013.07`/`GB-013.08` no se adelantaron.
- Evidencia estructural: `codebase-memory` en modo `full` identificó las rutas de favoritos, healthcheck y documentación OpenAPI, junto con las capas API, aplicación, dominio, Prisma, JWT y cliente AuthUser. La comprobación de cobertura de 13 rutas no registró incidencias; marcó los archivos fuente como `metadata_changed`, por lo que se contrastaron directamente contra el checkout actual. La migración Prisma quedó como evidencia parcial del índice y no se usó para afirmar cobertura completa.
- Entregables: commit `ebda62c` añade `architecture/` con fuentes JSON EN/ES, HTML standalone, recibos y capturas `visual-check`, además de `GameBook.Microservice.Game-architecture-en-dark.png` y `...-es-dark.png`. Cada README enlaza únicamente la imagen y el HTML interactivo publicado de su idioma; no se enlazan HTML fuente ni la variante opuesta.
- Archify EN: `validate` y `deliver` pasaron 9/9 `showcase`, 0 errores y 0 advertencias; fuente SHA-256 `8f980f6d9725962930bf59a76da2bf5166ce24632be93e80c9ec23307bb7f9e9` (6.885 bytes) y HTML `2f5c5b96cfda88eef3f0294b799a86b9defa256ce72a9334c8d4c5f16b602563` (818.122 bytes).
- Archify ES: `validate` y `deliver` pasaron 9/9 `showcase`, 0 errores y 0 advertencias; fuente SHA-256 `304d727a85f57bced9c8492a1518dea75aa31472ce9bb61ce3fe28a0e086ce77` (7.007 bytes) y HTML `6cd366baf072f12517268e399c40cb5828d1f7cf261a0d2fdfda28ac7ee2423e` (818.479 bytes). La variante ES omite `meta.locale`, por lo que los controles fijos del visor permanecen en inglés conforme al contrato de Archify.
- Evidencia de navegador y revisión perceptual: ambos `visual-check` terminaron `status=pass` con Chrome disponible, sin overflow en 1440×900, 1600×1000, 1920×1080 y 2048×1320; capturas claras/oscuras y controles del visor aprobados. La inspección de las capturas oscuras EN/ES en 1440×900 y 2048×1320 confirmó legibilidad, contraste, topología, relaciones y ausencia de recortes/solapamientos (`visual_review: passed`).
- README y Pages: se añadieron las secciones bilingües con la preview oscura y las URLs localizadas `https://carlossv923.github.io/GameBook.Microservice.Game/GameBook.Microservice.Game-architecture-en.html` y `https://carlossv923.github.io/GameBook.Microservice.Game/GameBook.Microservice.Game-architecture-es.html`. El repositorio público confirmó Pages inicialmente deshabilitado; se habilitó mediante API en modo `workflow`, con URL base `https://carlossv923.github.io/GameBook.Microservice.Game/`. El workflow `.github/workflows/architecture-pages.yml` publicará `architecture/` desde `main`; la publicación final y los HTTP 200 quedan pendientes de la promoción a `main`.
- Calidad local: `pnpm test` pasó con 13 archivos y 66 pruebas; `pnpm test:e2e` con 3 archivos y 22 pruebas; `pnpm lint`, `pnpm typecheck`, `pnpm build` y `git diff --check` pasaron. `pnpm format:check` reportó 49 archivos TypeScript preexistentes fuera del alcance; no se reformatearon para esta subtarea.
- PR de implementación: [Game #39](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/39), `feature/013-09/game-architecture` → `develop`, fusionado el 2026-09-28 a las 03:16:06 UTC con merge `e1158b094111fa0816162b7dc6adfadb0c7a1c21`; ambos checks `repository-baseline` terminaron `SUCCESS` (runs `36373001193` y `36373019506`). La promoción [Game #40](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/40), `develop` → `main`, fue fusionada el 2026-09-28 a las 03:19:42 UTC con merge `0ea4aed9960f75492b1e70811bc187a61bb3c116`; ambos checks `repository-baseline` terminaron `SUCCESS`. `main` fue verificado en ese commit y las dos variantes de GitHub Pages respondieron HTTP 200 con contenido Archify. GB-013.09 queda `[ RESOLVED ]` y GB-013 se cierra sin iniciar GB-014.

### [ ACTIVE ] GB-014 — Crear y mantener la documentación publicada de GameBook.System

Ámbito: coordinación y documentación; `GameBook.System` es cuarto repositorio **solo documental**. La publicación inicial mantiene únicamente `main` y todo el contenido documental completo. Los ajustes documentales posteriores pueden usar PR hacia `main`; no hay release-please, Vercel ni pnpm.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-014.01** — Inventariar material publicable | GB-012, GB-013 | Lista de `AGENTS.md`, `CLAUDE.md`, `specs/`, `plan/`, `tasks/`, `contracts/` (v3 vigente y v1/v2 históricas), documentación nueva y Archify; excluir secretos, `.agents/`, cachés, dependencias y archivos ajenos. |
| [ RESOLVED ] **GB-014.02** — Redactar documentación del sistema real | GB-014.01 | En inglés: visión, flujos, arquitectura, implementación DDD/Prisma/JWT/IGDB OAuth, pruebas, logs, despliegue, versiones y limitaciones, con evidencia de los tres repos. |
| [ RESOLVED ] **GB-014.03** — Completar pares de documentación general | GB-014.02 | La documentación general nueva (`docs/` y README) tendrá pares completos EN/ES; los documentos SDD, contratos, ambientes, `AGENTS.md` y `CLAUDE.md` ya generados en español se conservarán en español. |
| [ RESOLVED ] **GB-014.04** — Preparar README principal y diagramas | GB-014.03, GB-013 | `README.md` inglés y `README.es.md` español con enlaces a los tres repos, los diagramas EN/ES y guía documental; fuentes/HTML Archify incluidos. |
| [ RESOLVED ] **GB-014.05** — Ejecutar prepublicación exhaustiva | GB-014.04 | Inventario sin Markdown huérfano, enlaces relativos/externos válidos, cero secretos, nombres de repos correctos, estado temporal de AGENTS/CLAUDE actualizado y SDD histórico identificado. |
| [ RESOLVED ] **GB-014.09** — Ajustar navegación documental, nombres Archify y overview bilingüe | GB-014.05 | README sin enlaces de fuente/recibo ni idioma redundante; cada README abre solo su HTML; artefactos `architecture/` nombrados `GameBook.System-*`; overview con sección Release Please y arquitectura localizada. |
| [ RESOLVED ] **GB-014.10** — Unificar documentación general en README | GB-014.09 | README EN/ES conserva la información importante de overview sin duplicarla, `docs/` se elimina, no quedan enlaces/referencias rotos y el inventario refleja una única documentación por idioma. |
| [ RESOLVED ] **GB-014.06** — Crear/publicar GameBook.System | GB-014.05 | `gh` crea repo público **solo con `main`** y sube un único commit inicial con todo listo; URL y contenido público verificados. |
| [ RESOLVED ] **GB-014.11** — Publicar diagramas de arquitectura con GitHub Pages | GB-014.06 | Añadir un workflow de GitHub Actions que publique los HTML de `architecture/` en GitHub Pages, reemplazar en cada README el enlace local por la URL Pages de su idioma, actualizar la documentación SDD y abrir un PR desde una rama basada en `main`; verificar workflow, URLs y contenido tras la integración. |
| [ RESOLVED ] **GB-014.07** — Enlazar documentación desde aplicaciones | GB-014.06 | `README.md` y `README.es.md` de AuthUser, Game y Frontend enlazan `GameBook.System` mediante el flujo normal de PR de cada aplicación. |
| [ NEW ] **GB-014.08** — Comprobar entrega documental | GB-014.07 | Cuatro repos públicos accesibles, README principal de cada uno en inglés, versiones españolas completas, enlaces bidireccionales y diagramas EN/ES abiertos; sin publicaciones adicionales en `GameBook.System`. |

#### GB-014.01 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencias `GB-012` y `GB-013` verificadas como `[ RESOLVED ]` antes del inicio.
- Se preparó el staging local `GameBook.System/` sin crear todavía el repositorio remoto ni publicar contenido. El inventario `GameBook.System/inventory/publication-inventory.json` registra el alcance autorizado, las exclusiones y los documentos que quedan para GB-014.02–GB-014.04.
- El staging contiene `AGENTS.md`, `CLAUDE.md`, `specs/mvp.md`, `plan/mvp.md`, `tasks/mvp.md`, todos los archivos de `contracts/`, los ocho documentos de `environments/` y los dieciséis artefactos existentes de `architecture/` (fuentes, HTML, recibos de revisión y capturas EN/ES).
- Validación: el JSON se parseó correctamente; el inventario esperaba y encontró 45 archivos; no hubo archivos ausentes ni inesperados, no hubo diferencias SHA-256 contra las fuentes copiadas y no aparecieron `secrets/`, `.agents/`, `.codebase-memory/`, `.pnpm-store/`, repositorios de aplicaciones, dependencias ni cachés dentro del staging.
- En el momento del cierre de GB-014.01 no se habían generado traducciones, documentación del sistema, README final, repositorio remoto ni PR; las acciones pendientes quedaron para GB-014.02–GB-014.06.

#### GB-014.02 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.01` verificada como `[ RESOLVED ]` antes del inicio.
- Se redactó la documentación inglesa de 179 líneas sobre propósito, responsabilidades, arquitectura, flujos público/autenticado, DDD, Prisma, JWT RS256, revocación, OAuth de Twitch/IGDB, pruebas, logs, despliegue, versiones y limitaciones; GB-014.10 la consolidó posteriormente en `README.md`.
- La evidencia se contrastó contra `origin/main` actualizado de los tres repositorios: Frontend `38e9a82cd9f73725108f066925f9d83af6581c61` (`0.9.0`), AuthUser `fb522e0b7172d9afa60409a8fa638b22c3a56983` (`0.3.0`) y Game `b825daa6629ae83b9a2d1c801e7c0e086e59dfea` (`0.3.0`). La exploración estructural de `codebase-memory` corroboró rutas, límites y capas; las afirmaciones finales se verificaron directamente contra los refs de `main`.
- El inventario se actualizó para incluir el overview general; el staging contiene ahora 46 archivos, sin diferencias de inventario, rutas absolutas locales ni valores de secretos detectables. Los tres enlaces relativos a los artefactos Archify se resolvieron correctamente.
- No se crearon traducciones ni README final, no se creó repositorio remoto, no se publicaron cambios y no se inició GB-014.03; esas acciones permanecen en las subtareas siguientes.

#### GB-014.03 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.02` verificada como `[ RESOLVED ]` antes del inicio.
- Decisión del autor registrada durante la ejecución: los documentos SDD, contratos, ambientes, `AGENTS.md` y `CLAUDE.md` ya generados en español se conservan en español; no se envían a servicios externos ni se crean copias artificiales de esos documentos. Los pares bilingües se limitan a la documentación general nueva y a los README de GB-014.04.
- Se creó la traducción española completa de la documentación general, conservando rutas, enlaces, identificadores, commits y decisiones técnicas; GB-014.10 la consolidó posteriormente en `README.es.md`.
- Validación: ambos documentos generales tenían 179 líneas, no había `.es.md` huérfanos, los enlaces relativos a `architecture/` resolvían correctamente y los hashes de todos los documentos históricos en español (`AGENTS.md`, `CLAUDE.md`, `specs/`, `plan/`, `tasks/`, `contracts/` y `environments/`) coincidían con sus copias del staging.
- No se crearon README todavía, no se creó repositorio remoto, no se publicaron cambios y no se inició GB-014.04.

#### GB-014.04 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencias `GB-014.03` y `GB-013` verificadas como `[ RESOLVED ]` antes del inicio.
- Se añadieron [`README.md`](../README.md) en inglés y [`README.es.md`](../README.es.md) en español, con enlaces recíprocos, tabla de los tres repositorios, URLs productivas, guía documental y alcance del repositorio documental.
- Cada README enlaza únicamente el diagrama Archify, la imagen oscura, la fuente JSON y el recibo de revisión de su propio idioma: EN usa `GameBook.System-architecture-en.*` y ES usa `GameBook.System-architecture-es.*`. Ambos describen la existencia de la variante opuesta sin mostrarla como diagrama principal.
- Validación: ambos README tienen 53 líneas, todos sus enlaces relativos resuelven dentro del staging y la revisión de enlaces confirmó que el diagrama principal de cada README corresponde a su idioma. El inventario se actualizó para incluir los dos README; el staging contiene 49 archivos sin diferencias de inventario.
- No se creó repositorio remoto, no se publicó contenido y no se inició GB-014.05.

#### GB-014.05 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.04` verificada como `[ RESOLVED ]` antes del inicio.
- Se actualizó únicamente el staging documental: `AGENTS.md` y `CLAUDE.md` ya describen el sistema implementado, sus despliegues actuales y el paquete Archify existente; se retiraron referencias publicables a skills locales. Los documentos SDD, contratos y ambientes se identifican como trazabilidad histórica y se conserva la decisión del autor de mantenerlos en español.
- Auditoría de Markdown: el staging contiene 29 archivos Markdown y 50 archivos en total, con pares completos únicamente para la documentación general nueva (`README.md`/`README.es.md` y el overview que después se consolidó en ambos README); no hay Markdown huérfano bajo la política aprobada. Las rutas relativas Markdown resolvieron `0` enlaces rotos.
- Auditoría externa: se comprobaron 174 enlaces Markdown únicos mediante HTTP HEAD con redirecciones; `174` respondieron con estado 2xx/3xx y `0` fallaron. Los enlaces de los servicios Render apuntan a `/docs`, porque la raíz de esos servicios devuelve 404 al no definir una ruta.
- Seguridad y alcance: no quedaron claves privadas, cadenas de conexión, JWT ni credenciales funcionales; los ejemplos de credenciales/token del contrato se marcaron como placeholders en el staging. Los nombres de repositorio esperados son exactamente `GameBook.Frontend`, `GameBook.Microservice.AuthUser` y `GameBook.Microservice.Game`; no hay repositorios de aplicaciones, dependencias, cachés ni directorios excluidos dentro del staging.
- El recibo estructurado quedó en `inventory/prepublication-audit.json`; `GB-014.06` no se inició, no se creó repositorio remoto y no se publicó contenido.

#### GB-014.09 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.05` verificada como `[ RESOLVED ]` antes del inicio.
- Se renombraron los 16 artefactos de `architecture/` para usar el prefijo `GameBook.System-architecture-` y se actualizaron sus referencias internas, inventario, recibos y documentación.
- `README.md` y `README.es.md` ahora muestran únicamente la imagen oscura y el enlace al HTML interactivo de su propio idioma. Se retiraron los enlaces a fuentes JSON y recibos visuales, y el texto del enlace dejó de repetir el idioma.
- La documentación general bilingüe muestra únicamente el diagrama correspondiente a su idioma. La sección de versiones se sustituyó por un resumen de release-please con enlaces a la página de releases de Frontend, AuthUser y Game, sin commits ni versiones puntuales.
- Validación: los cuatro JSON de arquitectura se parsearon correctamente; no quedan nombres `gb-013.02-*`/`gb-013.03-*`, las rutas Markdown del staging resolvieron `0` enlaces rotos, los README no contienen enlaces a fuentes/recibos y el inventario conserva correspondencia con los 50 archivos del staging.
- No se creó repositorio remoto, no se publicó contenido y no se inició GB-014.06.

#### GB-014.10 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.09` verificada como `[ RESOLVED ]` antes del inicio.
- Se consolidó el contenido importante de los dos overview generales previos dentro de `README.md` y `README.es.md`, respectivamente. Los README conservan arquitectura localizada, flujos, límites de implementación, seguridad, pruebas, despliegue, releases y trazabilidad SDD sin mantener una copia paralela.
- Se eliminó exclusivamente `docs/`, que contenía los dos overview ya integrados. El staging queda con un único documento general por idioma: `README.md` y `README.es.md`.
- Validación: `docs/` no existe, el inventario y el recibo de auditoría se actualizaron a 48 archivos totales y 27 Markdown, no hay enlaces Markdown relativos rotos ni enlaces a los antiguos archivos de overview, y ambos README mantienen sus enlaces recíprocos, de arquitectura y de releases válidos.
- No se creó repositorio remoto, no se publicó contenido y no se inició GB-014.06.

#### GB-014.10 — Evidencia de reapertura y cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). La subtarea fue reabierta a solicitud del autor después de su cierre anterior y volvió a cerrarse tras completar este ajuste.
- Se añadió a cada README la captura oscura `1440x900` del diagrama Archify correspondiente: EN usa `GameBook.System-architecture-en.visual-check.1440x900.dark.png` y ES usa `GameBook.System-architecture-es.visual-check.1440x900.dark.png`.
- Las limitaciones de ambos README explican que AuthUser y Game utilizan el plan gratuito de Render, que los servicios pueden suspenderse tras inactividad y que el frontend puede experimentar una espera adicional mientras se reactivan; también documentan la mitigación mediante healthcheck, reintentos y mensaje de activación.
- Validación: las dos imágenes existen, sus enlaces Markdown resuelven, los enlaces relativos del staging siguen en `0` rotos, `docs/` continúa ausente y el inventario conserva 48 archivos y 27 Markdown. No se creó repositorio remoto ni se publicó contenido.

#### GB-014.06 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.05` y los ajustes documentales `GB-014.09`/`GB-014.10` verificadas como `[ RESOLVED ]` antes de la publicación.
- Se publicó el paquete documental completo en [`CarlosSV923/GameBook.System`](https://github.com/CarlosSV923/GameBook.System) como repositorio público, con `main` como única rama y rama predeterminada. La publicación contiene todo el staging validado, incluidos `README.md`, `README.es.md`, `architecture/`, `inventory/`, `specs/`, `plan/`, `tasks/`, `contracts/` y `environments/`.
- El repositorio se creó mediante `gh repo create` desde este staging y conserva un único commit inicial; no se crearon `develop`, PR, releases ni publicaciones adicionales en `GameBook.System`.
- La validación pública comprobó la URL del repositorio, la visibilidad pública, la rama predeterminada `main`, la existencia de los dos README, los HTML de arquitectura EN/ES y los recibos de inventario. La validación local conserva 48 archivos, 27 Markdown, `docs/` ausente, inventario completo, `0` enlaces relativos rotos y `0` JSON inválidos.

#### GB-014.11 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.06` verificada como `[ RESOLVED ]` antes del inicio.
- Se creó la rama `feature/014-11/github-pages-architecture` desde `main`. El alcance se limita a publicar estáticamente los HTML existentes en `architecture/`, actualizar los enlaces localizados de ambos README y documentar la automatización.
- El workflow `.github/workflows/architecture-pages.yml` usa `workflow_dispatch` y pushes a `main`, publica `architecture/` mediante las acciones oficiales de Pages y no ejecuta código de aplicación ni usa secretos.
- PR de implementación: [GameBook.System #1](https://github.com/CarlosSV923/GameBook.System/pull/1), `feature/014-11/github-pages-architecture` → `main`, fusionado con merge `6ad2dc1a6949c135a5d4adace390f3c7bd8b6ae2`. GitHub Pages quedó habilitado en modo `workflow` con URL base `https://carlossv923.github.io/GameBook.System/`.
- Validación remota: el workflow `Deploy architecture diagrams to GitHub Pages` terminó `success` en el run [36519319633](https://github.com/CarlosSV923/GameBook.System/actions/runs/36519319633) sobre `main`. `https://carlossv923.github.io/GameBook.System/GameBook.System-architecture-en.html` y `https://carlossv923.github.io/GameBook.System/GameBook.System-architecture-es.html` respondieron HTTP `200`, con contenido de sus respectivos idiomas.
- La subtarea queda `[ RESOLVED ]`; el commit de seguimiento registra esta evidencia en `main` sin cambiar el alcance del workflow.

#### GB-014.07 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-09-29, America/Guayaquil). Dependencia `GB-014.06` y GB-014.11 verificadas como `[ RESOLVED ]` antes del inicio.
- Los seis README de las tres aplicaciones enlazan la documentación de `GameBook.System` en su idioma: README.md apunta a `README.md` y README.es.md apunta a `README.es.md`, conservando los enlaces existentes entre aplicaciones.
- PRs de implementación fusionados en `develop`: Frontend [#62](https://github.com/CarlosSV923/GameBook.Frontend/pull/62), AuthUser [#41](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/41) y Game [#43](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/43). Sus checks `repository-baseline` terminaron correctamente.
- PRs de promoción fusionados en `main`: Frontend [#63](https://github.com/CarlosSV923/GameBook.Frontend/pull/63) con merge `0a21666357ef9af187d606f7d57dcf7f1b945bfb`, AuthUser [#42](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/42) con merge `f1ff07f220ce42a2bd0c553b0e4b005fd04e1fe7` y Game [#44](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/44) con merge `c3d6e0e820fc3427a80f6376978c8f5bbfe05b6f`.
- Verificación pública: los seis README de `main` respondieron HTTP 200 y contienen el enlace localizado correspondiente a `GameBook.System`. GB-014.07 queda `[ RESOLVED ]`; GB-014.08 permanece `[ NEW ]`.

## 6. Criterio de traspaso entre herramientas

Al solicitar una subtarea, la coordinación entregará: ID exacto, estado actual, dependencias ya `[ RESOLVED ]` con evidencia, repositorio y archivos de propiedad, secciones de `specs/mvp.md` y `plan/mvp.md`, contrato congelado aplicable, skill obligatorio si corresponde, comando de pruebas y definición de cierre de la tabla. La herramienta devolverá PR/commit o evidencia equivalente, pruebas ejecutadas, cambios de contrato propuestos, limitaciones y archivos tocados. La coordinación registrará todos los PR en este archivo. El autor los completará y avisará; el agente verificará en GitHub y revisará los demás criterios antes de marcar `[ RESOLVED ]`.

El estado vigente y su evidencia se mantienen en las tablas y en el registro de coordinación. Los PR documentados más abajo no se presumen fusionados por el hecho de estar creados.

## 7. Registro de coordinación

### Cambio de política de despliegue (2026-09-23)

- Decisión del autor: eliminará **personalmente** los tres proyectos Vercel iniciales desde la interfaz. La consulta de solo lectura de `GB-012.11` confirmó que ya no aparecen `gamebook-microservice-authuser`, `gamebook-microservice-game` ni `gamebook-frontend`; permanece únicamente el proyecto ajeno `portafolio-personal`, que no fue modificado.
- La evidencia de `GB-003.03`–`GB-003.05` permanece histórica y sus estados `[ RESOLVED ]` no se revocan. Los proyectos y variables Vercel que describen no constituyen infraestructura vigente para nuevas tareas. La [enmienda operativa](../environments/deployment-policy-local-first.md) reemplaza la estrategia de Preview.
- Ninguna tarea anterior a la aceptación integral local (`GB-012.01`/`GB-012.02`) requiere, crea o prueba Vercel. `GB-012.10` entrega Docker Compose; Neon `develop` y credenciales IGDB/Twitch de pruebas permanecen separados de `production`. `GB-012.05`–`GB-012.07` crean proyectos Vercel **después** de que el código y las migraciones estén en `main`, evitando importar de nuevo ramas vacías.
- Esta actualización documental no cambia otros estados `[ ACTIVE ]`/`[ NEW ]`, no crea PR, no certifica el merge de PR existentes ni ejecuta código de aplicación.

### Cambio de plataforma de backends (2026-09-26)

- Decisión del autor: AuthUser y Game dejan de publicarse en Vercel y usarán Render. Frontend conserva Vercel como proveedor posible de producción; esta decisión no convierte los proyectos históricos Vercel de backend en infraestructura vigente.
- AuthUser ya está publicado en Render; su enlace documental es [`Swagger de AuthUser`](https://gamebook-microservice-authuser.onrender.com/docs). La comprobación HTTP del 2026-09-26 devolvió `200` para `/docs` y `/docs/openapi.json`; la raíz no define una ruta y devuelve `404`, sin que ello indique un fallo de arranque.
- `GB-012.05` y `GB-012.06` pasan a exigir configuración y humo en Render. Las URLs directas de migración permanecen exclusivamente en GitHub Actions; `AUTH_DATABASE_URL`/`GAME_DATABASE_URL` son runtime del proveedor correspondiente.
- Se retiró de AuthUser la configuración Vercel versionada, la carpeta local `.vercel`, la validación Vercel del CI y las referencias Vercel de ambos README. Las referencias históricas de `GB-003` y de los diagnósticos previos se conservan como trazabilidad, no como instrucciones vigentes.
- Este cambio no resuelve por sí mismo todos los criterios de `GB-012.05`: permanecen pendientes las pruebas de login/sesión, función, logs y la integración CORS con el dominio final de Frontend.

### GB-001.01

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: espacio de trabajo local `GameBook`; consulta de GitHub CLI sin creación ni publicación de repositorios.
- Inicio: 2026-09-22 (America/Guayaquil).
- Cierre: el 2026-09-22, el autor ejecutó `gh auth status` en el terminal integrado y aportó una salida satisfactoria: cuenta activa `CarlosSV923`, protocolo HTTPS y alcance `repo` (además de los alcances mostrados). El propietario queda identificado como `CarlosSV923`. Se confirmaron los nombres exactos `GameBook.Microservice.AuthUser`, `GameBook.Microservice.Game` y `GameBook.Frontend`, y su futura visibilidad pública, conforme a la tabla de GB-001 y a `specs/mvp.md` §3 y §8. No se crearon repositorios ni se expusieron secretos. Las comprobaciones fallidas del proceso aislado de Codex no invalidan la evidencia del terminal integrado del autor.

### GB-001.02

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: GitHub de `CarlosSV923`; creación pública de los tres repositorios de aplicación mediante GitHub CLI, sin `GameBook.System` ni monorepo.
- Inicio: 2026-09-22 (America/Guayaquil).
- Cierre: dependencia GB-001.01 verificada como `[ RESOLVED ]`. Se crearon mediante `gh` los repositorios públicos [GameBook.Microservice.AuthUser](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser), [GameBook.Microservice.Game](https://github.com/CarlosSV923/GameBook.Microservice.Game) y [GameBook.Frontend](https://github.com/CarlosSV923/GameBook.Frontend). Para materializar `main` sin anticipar README ni código, cada repositorio recibió solo un commit vacío `chore: initialize repository`: `83a2b14`, `7b6bce5` y `ab0de91`, respectivamente. La comprobación final de GitHub devolvió visibilidad `PUBLIC` y rama predeterminada `main` para los tres; cada clon local está en `main` con `origin` HTTPS hacia su URL correspondiente. No se creó `GameBook.System`, no se creó un monorepo y no se añadieron archivos de aplicación.

### GB-001.03

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: tres repositorios de aplicación de `CarlosSV923`, ramas `main`/`develop` y configuración GitHub Actions/PR.
- Inicio: 2026-09-22 (America/Guayaquil).
- Cierre histórico: dependencia GB-001.02 verificada como `[ RESOLVED ]`. En cada repositorio se publicó el commit `chore: configure repository workflow` (`6bbe696` AuthUser, `84b8fbf` Game, `1321c74` Frontend), con `CONTRIBUTING.md` en inglés, `.github/pull_request_template.md` y `.github/workflows/ci.yml`. La convención `feature/[task-number]/[summary]`, el destino `develop` y el flujo de publicación a `main` quedaron documentados. Se creó y verificó `develop` en los tres repositorios. GitHub Actions CI (`repository-baseline`) terminó con `success` en `main` y `develop`. Inicialmente se configuraron seis protecciones de rama con aprobación y check obligatorios; esta decisión quedó sustituida por la política personal del autor.
- Corrección posterior: con autorización explícita del autor se eliminaron en GitHub las seis protecciones (`main` y `develop` de AuthUser, Game y Frontend). La API de GitHub devolvió `protected: false` para cada rama. Los workflows de CI y la plantilla de PR permanecen; no se fusionó ningún PR durante esta corrección. GB-001.03 sigue `[ RESOLVED ]` bajo el criterio actualizado.

### GB-001.04

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: tres PR hacia `develop`: AuthUser [#1](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/1), Game [#1](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/1) y Frontend [#1](https://github.com/CarlosSV923/GameBook.Frontend/pull/1).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: el autor notificó que había fusionado los tres PR. El 2026-09-23 UTC, la API de GitHub confirmó `merged: true`, `merged_at` no nulo y destino `develop` para los tres (commits de merge `230bcde78a5b3faa750602c52f9b51df13a93d95`, `aa28a7b2c98e4bdfc6125071d4116d07c7beb949` y `01e3c7d3520e9d6b8d81ce47bd4c7f50f0cc2a1a`, respectivamente). Se consultaron en `develop` `README.md` y `README.es.md` de cada repositorio: los seis existen, el principal está en inglés, la versión española enlaza de vuelta y ambos describen el propósito inicial correcto. El CI `repository-baseline` había terminado en `SUCCESS` para los tres PR. GB-001.04 queda `[ RESOLVED ]`.

### GB-001.05

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: verificación de GitHub para `CarlosSV923/GameBook.Microservice.AuthUser`, `CarlosSV923/GameBook.Microservice.Game` y `CarlosSV923/GameBook.Frontend`.
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre (2026-09-23 UTC): GitHub confirma los tres repositorios `PUBLIC`, URLs correctas, `main` como rama predeterminada, permiso del propietario `ADMIN` y ramas `main`/`develop` existentes. Los tres PR de GB-001.04 están `MERGED` hacia `develop`; ambos README de cada aplicación existen allí y se habían comprobado con título, descripción y enlaces recíprocos EN/ES. La antigua exigencia de verlos en `main` antes de producción queda reemplazada por la decisión explícita del autor: verificar entregables en `develop` y reservar `develop → main` para `GB-012`, al final de las implementaciones. Repositorios listos para el traspaso a contratos: [AuthUser](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser), [Game](https://github.com/CarlosSV923/GameBook.Microservice.Game) y [Frontend](https://github.com/CarlosSV923/GameBook.Frontend). No se creó ni fusionó ningún PR nuevo para este cierre; GB-001.05 y su grupo GB-001 quedan `[ RESOLVED ]`.

### GB-002.01

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [contracts/mvp-traceability.md](../contracts/mvp-traceability.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: la dependencia `GB-001` figura `[ RESOLVED ]` con evidencia en este registro. La matriz enumera exactamente los 16 criterios de `specs/mvp.md`, relaciona las operaciones previstas de AuthUser, Game y Next.js con cada criterio, identifica el propietario de cada dato y registra los casos de error y dependencia aplicables. La validación automatizada confirmó `criteria_count=16`, los valores `1` a `16`, y la presencia de las secciones de operaciones, errores y control de alcance. No se añadieron endpoints, campos, filtros, comportamientos ni código de aplicación; no se creó PR.

### GB-002.02

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [contracts/authuser.openapi.yaml](../contracts/authuser.openapi.yaml).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencia `GB-002.01` verificada como `[ RESOLVED ]`. El contrato OpenAPI 3.0.3 documenta las cuatro operaciones previstas (`POST /v1/auth/register`, `POST /v1/auth/login`, `GET /v1/auth/session` y `PATCH /v1/users/me/password`), sus cuerpos y respuestas, validaciones de nombre/correo/contraseña, identidad UUID, JWT Bearer, vigencia de 3600 segundos, revocación al cambiar contraseña, errores estables y ejemplos ficticios. La validación estructural confirmó las cuatro rutas, cuatro operaciones, esquemas requeridos, 15 respuestas HTTP documentadas y cero marcadores de secretos reales; no se generó código ni se creó PR.

### GB-002.03

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recursos: [contracts/jwt-revocation.md](../contracts/jwt-revocation.md) y [contracts/authuser.openapi.yaml](../contracts/authuser.openapi.yaml).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencia `GB-002.02` verificada como `[ RESOLVED ]`. El contrato fija RS256, encabezados `alg`/`typ`, claims obligatorios `sub` UUID, `ver`, `iat`, `exp`, `iss` y `aud`, vigencia nominal de 3600 segundos, variables de claves/emisor/audiencia sin valores, revocación atómica por incremento de versión y códigos `401` estables. También documenta la verificación local de Game seguida de `GET /v1/auth/session`, el `503 AUTHUSER_UNAVAILABLE` con fallo cerrado y fixtures de éxito, expiración, manipulación, revocación y caída de AuthUser. La validación confirmó las siete secciones, todos los claims/variables/códigos requeridos, cero marcadores de secretos y sin espacios finales ni tabuladores; no se generó código ni se creó PR.

### GB-002.04

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recursos: [contracts/game.openapi.yaml](../contracts/game.openapi.yaml) y [contracts/jwt-revocation.md](../contracts/jwt-revocation.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencias `GB-002.01` y `GB-002.03` verificadas como `[ RESOLVED ]`. El contrato OpenAPI 3.0.3 documenta guardar, listar/filtrar/paginar, sugerir sobre todos los favoritos propios, actualizar instantánea y eliminar. Fija filtros AND, intervalo anual inclusivo, orden determinista, límites de página/sugerencias, unicidad `(sub, rawgId)`, ausencia de `userId` como autoridad, aislamiento por `sub`, errores `400/401/404/409/503` y fallo cerrado cuando AuthUser no responde. La validación estructural confirmó cuatro rutas, cinco operaciones, todos los parámetros/esquemas/códigos requeridos, cero propiedades `userId`, cero marcadores de secretos y sin espacios finales ni tabuladores; no se generó código ni se creó PR.

### GB-002.05

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [contracts/rawg-mapping.md](../contracts/rawg-mapping.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencia `GB-002.01` verificada como `[ RESOLVED ]`; el contrato también se alinea con las decisiones de `GB-002.04`. La documentación oficial de RAWG se consultó para confirmar API key obligatoria, `/games`, `/platforms`, parámetros `dates`, `platforms`, `page`, `page_size`, `ordering=-rating`, paginación, campos de lista/detalle, cuota y atribución. El inventario fija el mapeo RAWG→tarjeta/detalle/instantánea, fechas y ausencias sin inventar, sugerencias, límites del proxy servidor Next.js, errores internos `RAWG_*`, preservación ante fallos de detalle y enlace de atribución. La validación confirmó las seis secciones, fuentes oficiales, campos/límites/códigos requeridos, cero marcadores de secretos y sin espacios finales ni tabuladores; no se generó código ni se creó PR.

### GB-002.06

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [contracts/persistence-environments.md](../contracts/persistence-environments.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencias `GB-002.03`, `GB-002.04` y `GB-002.05` verificadas como `[ RESOLVED ]`. La matriz define los esquemas Neon `auth`/`game`, propietarios y límites sin FK entre servicios, forma mínima de `User`, `Favorite` y plataformas, conexiones runtime/directas independientes, migraciones controladas, errores `400/401/404/409/503/500`, URLs `/v1`, CORS por origen exacto y variables de AuthUser, Game y Frontend para local/preview/production sin valores. La validación confirmó 8 secciones, 14 filas de variables, 6 códigos HTTP, cero marcadores de secretos y sin espacios finales ni tabuladores; no se ejecutaron migraciones, no se desplegaron recursos, no se generó código ni se creó PR.

### GB-002.07

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [contracts/contract-baseline.md](../contracts/contract-baseline.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencias `GB-002.02`, `GB-002.04`, `GB-002.05` y `GB-002.06` verificadas como `[ RESOLVED ]`. La referencia `GB-002-CONTRACTS-v1` reúne ambos OpenAPI, JWT/revocación, RAWG, persistencia y trazabilidad; documenta compatibilidad AuthUser↔Game↔Frontend, errores comunes y fixtures de registro, login, sesión, expiración, manipulación, revocación, duplicado, ausencia, caída de AuthUser, caída de RAWG y cambio de contraseña. La revisión documental de las tres líneas confirmó rutas, claims, aislamiento, paginación, variables y manejo de errores; las seis huellas SHA-256 coinciden con los archivos incluidos y no quedan placeholders. No se generó código ni se creó PR. Al quedar todas las subtareas del grupo resueltas, GB-002 también queda `[ RESOLVED ]`.

### GB-002.08

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: documentación oficial de IGDB y decisión del autor en esta conversación.
- Inicio y cierre: 2026-09-23 (America/Guayaquil).
- Cierre: la documentación oficial confirmó soporte para catálogo, `total_rating`, búsqueda y sugerencias de juegos/plataformas, filtros `where` combinados, `limit`/`offset`, detalle, géneros, desarrolladores, fechas e imágenes. También confirmó OAuth de aplicación Twitch, ausencia de CORS directo y límite de 4 solicitudes/s con 8 abiertas. El autor eligió `total_rating` combinado para el listado y tarjetas. La comprobación fue documental, sin credenciales reales ni llamadas a la cuenta del autor; los ensayos de integración quedan para `GB-009`/`GB-012`. No se creó PR ni código.

### GB-002.09

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recursos: [`contracts/contract-baseline-v3.md`](../contracts/contract-baseline-v3.md), especificación, plan, AGENTS/CLAUDE y nuevos contratos IGDB.
- Inicio y cierre: 2026-09-23 (America/Guayaquil).
- Cierre: se publicó una línea base v3 separada con `igdbId`, `total_rating` 0–100, OAuth Twitch server-only, mapeo de catálogo/detalle, esquema mínimo y trazabilidad. Las huellas SHA-256 de sus siete artefactos se registraron; v1/v2 y sus hashes permanecen como historia RAWG. Se actualizaron tareas futuras sin alterar cierres históricos ni resolver `GB-003.06`. Los README y CONTRIBUTING iniciales de `GameBook.Frontend` aún mencionan RAWG en su rama de desarrollo: su corrección se incluye explícitamente en el PR futuro de `GB-006.02`, ya que el autor debe fusionar los PR de aplicación antes de dar una subtarea por resuelta. No se generó código de producto ni se creó/fusionó PR en esta revisión documental.

### GB-003.01

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [environments/gb-003.01-neon.md](../environments/gb-003.01-neon.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencia `GB-002` verificada como `[ RESOLVED ]`. El autor confirmó el proyecto Neon y las ramas para producción y pruebas/preview; la posterior comprobación con Neon CLI aclaró que sus nombres reales son `production` y `develop`, respectivamente (`main` era una denominación lógica/ramal Git, no una rama Neon). El registro documenta las conexiones únicamente por `AUTH_DATABASE_URL`, `AUTH_DATABASE_DIRECT_URL`, `GAME_DATABASE_URL` y `GAME_DATABASE_DIRECT_URL`, conserva los límites de esquemas `auth`/`game` y deja Vercel, dominios, permisos, secretos y migraciones para sus subtareas correspondientes. No se creó otra base, no se ejecutaron migraciones ni se generó código en GB-003.01.

### GB-003.02

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [environments/gb-003.02-schema-isolation.md](../environments/gb-003.02-schema-isolation.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencia `GB-003.01` verificada como `[ RESOLVED ]`. En Neon `develop` y `production`, los roles SQL nuevos `gamebook_auth_app` y `gamebook_game_app` conectaron realmente; las consultas comprobaron `neon_superuser=false`, `USAGE` solo en el esquema propio y `CREATE=false` en base y ambos esquemas. Se configuraron grants por defecto de tablas/secuencias futuras desde cada migrador. Tras verificar que no poseían tablas, esquemas ni ACL por defecto, se retiraron de ambas ramas los antiguos roles `*_runtime` sobreprivilegiados. Los dos `*_migrator` persisten con `neon_superuser`; la enmienda [`../contracts/persistence-access-amendment.md`](../contracts/persistence-access-amendment.md) describe ese estado y su riesgo. Por la decisión posterior del autor, no se entregarán a Actions: `GB-003.06` debe probar reemplazos limitados. No se crearon tablas, no se ejecutaron migraciones Prisma, no se configuraron aún secretos en GitHub/Vercel y no hubo PR. Evidencia detallada en [`../environments/gb-003.02-schema-isolation.md`](../environments/gb-003.02-schema-isolation.md). El aislamiento runtime queda `[ RESOLVED ]`.

### GB-003.06

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recursos: [environments/gb-003.06-migrator-isolation.md](../environments/gb-003.06-migrator-isolation.md) y [contracts/persistence-access-amendment-v2.md](../contracts/persistence-access-amendment-v2.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre: dependencia `GB-003.02` verificada como `[ RESOLVED ]`. En `develop` se crearon por SQL `gamebook_auth_migrator_limited` y `gamebook_game_migrator_limited`, ambos sin `neon_superuser`, `CREATEDB` ni `CREATEROLE`, con `NOINHERIT`, `CONNECT` y `USAGE`/`CREATE` únicamente en `auth` o `game`, respectivamente, y sin `CREATE` en la base. Cada rol conectó, creó/alteró/eliminó una tabla efímera propia y recibió rechazo de lectura y DDL en el esquema ajeno; las ACL por defecto apuntan solo a `gamebook_auth_app` o `gamebook_game_app`. Tras el éxito de `develop`, la misma preparación y prueba se replicó en `production`; no quedaron tablas efímeras. Los migradores heredados no se usarán en Actions y se registró su gestión como propietarios administrativos históricos; no se configuraron secretos, no se ejecutaron migraciones Prisma y no se generó código. La enmienda [`../contracts/persistence-access-amendment-v2.md`](../contracts/persistence-access-amendment-v2.md) se publicó con la huella `8173C24ED299649C68FC88D7DDEF966867D57B8DF45408E2FF0206CF6E61A057`; la evidencia operativa está en [`../environments/gb-003.06-migrator-isolation.md`](../environments/gb-003.06-migrator-isolation.md). Criterios de `develop`, replicación de `production`, grants por defecto, retirada operativa de credenciales antiguas y ausencia de secretos verificados.

### GB-003.03

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [environments/gb-003.03-vercel.md](../environments/gb-003.03-vercel.md).
- Inicio y cierre: 2026-09-23 (America/Guayaquil).
- Cierre: dependencias `GB-001` y `GB-002` verificadas como `[ RESOLVED ]`. La CLI autenticada de Vercel creó y enlazó `gamebook-microservice-authuser`, `gamebook-microservice-game` y `gamebook-frontend` al repositorio GitHub público correspondiente; la API confirmó `productionBranch=main`, Git deployments habilitados y los tres dominios predeterminados `*.vercel.app`. `develop` queda como origen de previews mediante la integración Git, con URLs de deployment propias. No se configuraron variables ni secretos, no se desplegó código y no se modificó el proyecto `portafolio-personal`.

### GB-003.04

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [environments/gb-003.04-secrets-and-variables.md](../environments/gb-003.04-secrets-and-variables.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Cierre (2026-09-23): dependencias `GB-003.03` y `GB-003.06` `[ RESOLVED ]`. `gh secret list` confirmó `AUTH_DATABASE_DIRECT_URL`/`GAME_DATABASE_DIRECT_URL` en `develop` y `production` del backend propio. `vercel env ls` confirmó `AUTH_DATABASE_URL` y `GAME_DATABASE_URL` en `Preview`/`Production` y `IGDB_CLIENT_ID`/`IGDB_CLIENT_SECRET` en `Preview`/`Production` de Frontend. Se verificaron solo nombres, tipos y ámbitos; no se recuperaron valores, no se probó OAuth y el HTTP `400` anterior con valores no utilizables no diagnostica las credenciales reales. El requisito original de publicar **todas** las variables antes de crear/desplegar aplicaciones producía una dependencia circular: las URLs de preview y los orígenes CORS aún no existen. El alcance se corrigió a credenciales determinables ahora; JWT, URLs y CORS quedaron asignados a tareas posteriores sin placeholders. No hubo código, PR ni migración de aplicación.

### GB-003.05

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [environments/gb-003.05-variable-handoff.md](../environments/gb-003.05-variable-handoff.md).
- Inicio y cierre: 2026-09-23 (America/Guayaquil).
- Cierre: `GB-003.04` figura `[ RESOLVED ]` tras la comprobación de los nombres publicados. El traspaso identifica las seis variables disponibles, sus ámbitos y las pruebas pendientes; asigna generación JWT a AuthUser, clave pública compatible a Game, `AUTHUSER_URL` a la integración Game→AuthUser, URLs públicas del frontend y CORS a las integraciones con despliegues reales, y valores de producción a `GB-012`. Aclara que CORS contiene orígenes **del frontend**, no URLs de backend/IGDB, y que en Preview se esperará a la URL de rama real. Se comprobaron enlaces relativos y consistencia de dependencias. No se leyeron secretos ni se publicaron valores ficticios; con todas las subtareas `GB-003` resueltas, el grupo queda `[ RESOLVED ]`.

### GB-004.01

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: [environments/gb-004.01-authuser-base.md](../environments/gb-004.01-authuser-base.md).
- Inicio: 2026-09-23 (America/Guayaquil).
- Rama: `feature/004-01/initialize-nestjs-pnpm` en `GameBook.Microservice.AuthUser`.
- Cierre: dependencias `GB-002` y `GB-003` verificadas como `[ RESOLVED ]`. Se generó el scaffolding NestJS 12 con TypeScript estricto y `pnpm`, se prepararon los directorios DDD `src/api`, `src/application`, `src/domain`, `src/infrastructure`, `tests/unit`, `tests/integration` y `prisma`, y se ajustaron scripts/configuración sin añadir funcionalidad de producto. `pnpm test`, `pnpm test:e2e`, `pnpm lint`, `pnpm build` y Prettier pasan. Tras el aviso del autor, el agente verificó el [PR AuthUser #2](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/2) como `MERGED` el 2026-09-24 01:35:10 UTC, con destino `develop`, commit de merge `0e53b303de28b34829cb47441651fb9b6c13fc23`, CI `repository-baseline` en `SUCCESS` y el entregable presente en `origin/develop`. El antiguo fallo de Vercel Preview no es un criterio bajo la política local-first.

### GB-004.02

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: dominio e interfaces de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-23 (America/Guayaquil).
- Dependencia verificada: `GB-004.01` figura `[ RESOLVED ]` y `develop` local fue actualizado por fast-forward a `origin/develop` antes de crear la rama de trabajo.
- Rama: `feature/004-02/model-user-interfaces` en `GameBook.Microservice.AuthUser`.
- Alcance: modelar exclusivamente la entidad de usuario, sus reglas de contraseña y los contratos de repositorio/criptografía, manteniendo el dominio independiente de NestJS, Prisma y HTTP. No se inicia `GB-004.03`.
- PR: [AuthUser #3](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/3), fusionado hacia `develop`.
- Cierre (2026-09-24 UTC): tras el aviso del autor, GitHub confirmó estado `MERGED`, destino `develop`, commit de merge `0fe82c79a342841383af93247a9060ce3922e993` y CI `repository-baseline` en `SUCCESS`. El entregable de GB-004.02 está presente en `origin/develop`; `pnpm test` (13/13), `pnpm test:e2e` (1/1), `pnpm lint`, `pnpm build` y Prettier habían pasado. No se inició GB-004.03 dentro de ese PR.

### GB-004.03

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: esquema Prisma y migración propia de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-23 (America/Guayaquil).
- Dependencia verificada: `GB-004.02` acaba de quedar `[ RESOLVED ]` tras comprobar su merge y entregable en `origin/develop`.
- Alcance: crear exclusivamente el esquema Prisma del servicio AuthUser en el esquema PostgreSQL `auth`, con usuario, correo único y migración reproducible; no tocar el esquema `game`, no implementar adaptadores de GB-004.04 y no ejecutar migraciones desde Vercel.
- Rama: `feature/004-03/create-auth-schema-migration` en `GameBook.Microservice.AuthUser`.
- Commit: `f0c0110` (`feat(authuser): add auth prisma schema and migration`).
- PRs: [AuthUser #4](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/4) y [AuthUser #5](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/5), ambos fusionados hacia `develop` por el autor. PR #4: merge `f0b13b769385a4a2e06789f45defe70ca0c3046a`, 2026-09-24 02:52:23 UTC, CI `repository-baseline` `SUCCESS`. PR #5 (`ajustes`): merge `65024ea9cd7b0aadee1608ad81502ead467cb478`, 2026-09-24 03:29:56 UTC, CI `repository-baseline` `SUCCESS`.
- Integración Neon `develop` verificada el 2026-09-24 UTC cargando `AUTH_DATABASE_DIRECT_URL` desde `.env`: tras corregir la credencial y usar el endpoint directo de Neon, `pnpm db:validate` pasó. Prisma conservaba el intento fallido `20260924030000_init_auth_user` y agotaba el advisory lock; con `PRISMA_SCHEMA_DISABLE_ADVISORY_LOCK=1` solo para esta ejecución, se marcó como `--rolled-back` y `pnpm db:migrate:deploy` la aplicó correctamente. La comprobación final `pnpm db:validate` y `pnpm prisma migrate status` confirmó `Database schema is up to date!`. Tras el aviso del autor, se verificaron ambos merges en `develop`, CI verde y el entregable final bajo `src/infrastructure/persistence/prisma/`; GB-004.03 queda cerrada.

### GB-004.04

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: adaptadores base de persistencia, hash y firma/verificación de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencias verificadas: `GB-004.03` y `GB-003.06` figuran `[ RESOLVED ]`.
- Alcance: implementar exclusivamente el repositorio Prisma de usuario, el adaptador de hash con sal y los puertos/adaptadores de firma/verificación JWT RS256; cargar configuración privada local sin versionar secretos y entregar a Game únicamente la clave pública de desarrollo. No implementar frontera HTTP de GB-004.05, pruebas base de GB-004.06, Actions de GB-004.07 ni cambios de `tasks/mvp.md` desde el recurso de trabajo.
- Rama: `feature/004-04/implement-base-adapters` en `GameBook.Microservice.AuthUser`, creada sobre `origin/develop` actualizado al commit `65024ea`.
- Commit: `86982db` (`feat(authuser): add base persistence and crypto adapters`).
- PR: [AuthUser #6](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/6), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 03:54:21 UTC con merge commit `3fce0efba61b3a39c14eb26e5b16718edd15e68b`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Cierre verificado: `origin/develop` contiene los adaptadores de configuración, criptografía, Prisma y sus pruebas bajo `src/infrastructure/` y `tests/unit/infrastructure/`; las validaciones locales fueron 24 pruebas unitarias, 1 e2e, lint, build y Prettier correctos. GB-004.04 queda cerrada.

### GB-004.05

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: frontera HTTP, validación, errores uniformes, `requestId`, logs y CORS de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencia verificada: `GB-004.04` figura `[ RESOLVED ]` y `develop` local fue actualizado por fast-forward a `origin/develop` en el commit `3fce0ef` antes de crear la rama de trabajo.
- Alcance: preparar exclusivamente la frontera base del servicio conforme al contrato vigente: rutas/controladores base sin casos de uso de `GB-007`, validación y cuerpo uniforme de error, correlación mediante `requestId`, logs estructurados en inglés sin secretos ni JWT y CORS por lista explícita. Si `CORS_ALLOWED_ORIGINS` falta, las llamadas cross-origin se rechazan sin impedir el arranque; nunca se habilita `*` por defecto. No iniciar `GB-004.06`, `GB-004.07` ni modificar `GameBook.Microservice.Game`.
- Rama: `feature/004-05/prepare-api-logging` en `GameBook.Microservice.AuthUser`.
- Commit: `3e94928` (`feat(authuser): prepare api boundary and safe logging`).
- PR: [AuthUser #7](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/7), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 04:19:10 UTC con merge commit `ec902a8436d44dd499c31ffd90e34395e0b7a224`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Cierre verificado: `origin/develop` contiene los módulos HTTP, filtro de errores, validación, `requestId`, CORS, logging y pruebas; las validaciones fueron `pnpm test` (32 pruebas), `pnpm test:e2e` (4 pruebas), lint, build y Prettier correctos. La frontera no registra cuerpos, contraseñas, JWT, claves privadas ni API keys; CORS no habilita `*` y sin lista configurada rechaza orígenes cross-origin. GB-004.05 queda cerrada.

### GB-004.06

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: pruebas base locales, arranque NestJS/Prisma y verificación de entorno de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencia verificada: `GB-004.05` figura `[ RESOLVED ]` y `develop` local fue actualizado por fast-forward a `origin/develop` en el commit `ec902a8` antes de crear la rama de trabajo.
- Alcance: verificar exclusivamente la base local del servicio: pruebas unitarias/integración, lint, tipos, build, arranque HTTP con Prisma, conexión runtime segura mediante `AUTH_DATABASE_URL` de Neon `develop`, configuración JWT de desarrollo sin revelar valores y rechazo CORS no autorizado. No implementar casos de uso de `GB-007`, Actions de `GB-004.07`, migraciones nuevas ni despliegue/prueba Vercel.
- Rama: `feature/004-06/verify-local-base` en `GameBook.Microservice.AuthUser`.
- Evidencia de avance: se añadió el proveedor de ciclo de vida Prisma, carga local de `.env`, una prueba de configuración/cliente runtime con datos ficticios y CI real para lint, unitarias, integración y build. El entorno local contiene `JWT_PRIVATE_KEY`, `JWT_ISSUER` y `JWT_AUDIENCE` sin versionar; la clave tiene formato PEM de clave privada válido. Las validaciones pasan: 32 unitarias, 6 de integración, `db:validate`, `db:generate`, lint, build, Prettier de los archivos de la subtarea y `git diff --check`. La conexión real con `AUTH_DATABASE_URL` de Neon `develop` verificó el rol `gamebook_auth_app`, `USAGE` sin `CREATE` en `auth`, ausencia de permisos en `game` y lectura Prisma de `auth.User` sin exponer datos. El arranque NestJS real respondió `200`; un origen CORS no autorizado no recibió `Access-Control-Allow-Origin`.
- Rama: `feature/004-06/verify-local-base` en `GameBook.Microservice.AuthUser`.
- Commits: `0580dbb` (`test(authuser): verify local runtime base`) y `c6719db` (`ci(authuser): generate prisma client before tests`).
- PR: [AuthUser #8](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/8), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 05:14:09 UTC con merge commit `013fc786b9b935b11993dbdff2be8ffab3c1990c`, destino `develop` y CI `repository-baseline` `SUCCESS` en ambos workflows.
- Cierre verificado: `origin/develop` contiene `.github/workflows/ci.yml`, `src/infrastructure/persistence/prisma/prisma-service.ts`, `src/main.ts`, `tests/integration/app.e2e-spec.ts` y `tests/integration/prisma-runtime.e2e-spec.ts`; el cliente Prisma se genera antes de ejecutar CI. GB-004.06 queda cerrada.

### GB-004.07

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: Action de migraciones Prisma del esquema `auth` de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencias verificadas: `GB-004.03` y `GB-003.06` figuran `[ RESOLVED ]`.
- Alcance: implementar exclusivamente el workflow propio de migraciones `auth` con `pnpm`, `AUTH_DATABASE_DIRECT_URL` y el migrador limitado; ejecutar en `develop` al integrar cambios de migración y dejar preparada la ejecución controlada desde `main` para producción. Verificar éxito, idempotencia, rechazo de secreto ausente o esquema incorrecto y ausencia de migraciones en Vercel. No modificar Game, Compose, Vercel ni ejecutar `GB-005.07`.
- Rama de trabajo: `feature/004-07/auth-migrations-action` en `GameBook.Microservice.AuthUser`, creada después de actualizar `develop` local al merge `013fc786` de `GB-004.06`.
- Commit: `f1c2c65` (`ci(authuser): add controlled migration workflow`).
- PR: [AuthUser #9](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/9), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 05:24:44 UTC con merge commit `6d51226997b796d84c6a76d672b2fe55f1fadea6`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Evidencia de avance: el workflow selecciona `develop` para push de migraciones y `production` solo para `workflow_dispatch` desde `main`; valida secreto presente, rol `gamebook_auth_migrator_limited`, esquema `auth` cuando la URL lo declara y migraciones versionadas; ejecuta `pnpm db:migrate:deploy` dos veces. Las validaciones locales pasan: `db:validate`, 32 unitarias, 6 de integración, lint, build, Prettier y `git diff --check`; los casos sintéticos de secreto ausente, rol/esquema incorrectos y URL válida fueron comprobados.
- Corrección operativa verificada: Neon `develop` mantiene `public._prisma_migrations` como tabla de metadatos de Prisma, propiedad de `neondb_owner`; se concedieron únicamente `SELECT`, `INSERT`, `UPDATE` y `DELETE` a `gamebook_auth_migrator_limited`, sin ampliar `CREATE` de base, superusuario ni acceso a tablas de aplicación.
- Cierre verificado: tras el ajuste, el workflow [run 35959824069](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/actions/runs/35959824069) terminó `SUCCESS`; `Apply AuthUser migrations` y `Verify migration idempotence` pasaron en `develop`. `origin/develop` contiene `.github/workflows/migrate-auth.yml` y la migración `20260924030000_init_auth_user`; GB-004.07 queda cerrada.

### GB-005.01

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: base NestJS y pnpm de `GameBook.Microservice.Game`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencias verificadas: `GB-002` y `GB-003` figuran `[ RESOLVED ]`.
- Rama: `feature/005-01/initialize-nestjs-pnpm` en `GameBook.Microservice.Game`.
- PR: [Game #2](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/2), fusionado hacia `develop`; el autor notificó el merge y GitHub confirmó el 2026-09-24 02:46:06 UTC con commit `4cc67509bebeb5a0ad0fbae66aa5ca515f041613` y CI `repository-baseline` `SUCCESS`.
- Cierre verificado: `origin/develop` contiene `src/main.ts`, `prisma/.gitkeep` y `tests/integration/app.e2e-spec.ts`; las pruebas, lint, build y comprobación de formato del task separado finalizaron correctamente.

### GB-005.02

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: dominio y puertos de casos de uso de favoritos de `GameBook.Microservice.Game`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencia verificada: `GB-005.01` figura `[ RESOLVED ]`.
- Rama: `feature/005-02/model-favorite-interfaces` en `GameBook.Microservice.Game`.
- Commit: `496cf7b3a3a0b2507b1332d0f6ffa5d551887c88` (`feat(game): model favorites and use-case ports`).
- PR: [Game #3](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/3), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 03:49:20 UTC con merge commit `7d2039dd5e9c50698d01755df09c5be4eff64641`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Cierre verificado: `origin/develop` contiene `Favorite`, `FavoritePlatform`, `FavoriteRepository`, los puertos de casos de uso y sus pruebas; el task separado reportó 17 pruebas unitarias, 1 e2e, lint, build y Prettier correctos. GB-005.02 queda cerrada.

### GB-005.03

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: esquema Prisma y migración propia del esquema `game` de `GameBook.Microservice.Game`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencia verificada: `GB-005.02` figura `[ RESOLVED ]`.
- Alcance: crear exclusivamente las tablas de favoritos y plataformas en `game`, con unicidad, índices, checks de IDs/puntuación y FK únicamente interna entre `FavoritePlatform` y `Favorite`; `userId` permanece como UUID lógico sin FK ni acceso a `auth`.
- Rama: `feature/005-03/create-game-schema-migration` en `GameBook.Microservice.Game`.
- Commit: `17c43eb` (`feat(game): add favorites schema and migration`).
- PR: [Game #4](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/4), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 04:20:04 UTC con merge commit `c6dadf1a600cfccce4651a0ff9a91fa18ae752ef`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Cierre verificado: `origin/develop` contiene `schema.prisma`, `prisma.config.ts` con `GAME_DATABASE_DIRECT_URL` y la migración `20260924040000_init_game_favorites`; el esquema usa únicamente `game`, la migración no referencia `auth`, y el task separado reportó `db:validate`, `db:generate`, pruebas (17/17), lint, build y diff de migración correctos. GB-005.03 queda cerrada.

### GB-005.04

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: repositorio Prisma y consultas de favoritos de `GameBook.Microservice.Game`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencia verificada: `GB-005.03` figura `[ RESOLVED ]`.
- Alcance: persistencia exclusivamente en el esquema `game`, con aislamiento por `userId`, guardado/upsert de instantáneas y plataformas, filtros AND, orden estable, paginación acotada, sugerencias limitadas y borrado delimitado por usuario.
- Rama: `feature/005-04/implement-favorite-repository` en `GameBook.Microservice.Game`.
- Commit: `86f09f8` (`feat(game): implement favorite repository`).
- PR: [Game #5](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/5), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 04:46:47 UTC con merge commit `1225b94f5a6fb86dcdd36c584e7022df08a24acb`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Cierre verificado: `origin/develop` contiene `prisma-client.ts`, `prisma-favorite-repository.ts` y sus pruebas; el repositorio limita las consultas al `userId`, usa filtros/orden/paginación/sugerencias acotados y el task separado reportó `db:generate`, 23 pruebas unitarias, lint y build correctos. GB-005.04 queda cerrada.

### GB-005.05

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: guardia JWT RS256 y cliente de sesión AuthUser de `GameBook.Microservice.Game`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencia verificada: `GB-005.01` y `GB-002` figuran `[ RESOLVED ]`.
- Alcance: verificación local RS256, claims issuer/audience/UUID/version/iat/exp, cliente `GET /v1/auth/session` con Bearer exacto, timeout/fallo cerrado y guardia con mapeo seguro 401/503; no leer tablas `auth` ni confiar en `userId` del cliente.
- Rama: `feature/005-05/jwt-auth-guard` en `GameBook.Microservice.Game`.
- Commit: `b2814f3` (`feat(game): add JWT auth guard and session client`).
- PR: [Game #6](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/6), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 05:01:08 UTC con merge commit `37476e56bffad68abdfd4f42748d9709a2f51c5c`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Cierre verificado: `origin/develop` contiene `src/api/auth/jwt-auth-guard.ts`, `src/application/ports/auth-user-session.ts`, `src/application/ports/jwt-ports.ts`, `src/infrastructure/auth/auth-user-session-client.ts`, `src/infrastructure/cryptography/rsa-jwt.ts` y las pruebas `tests/unit/api/auth/jwt-auth-guard.spec.ts`, `tests/unit/infrastructure/auth/auth-user-session-client.spec.ts` y `tests/unit/infrastructure/cryptography/rsa-jwt.spec.ts`; el task separado reportó `pnpm test` (36/36), lint, build y `git diff --check` correctos. GB-005.05 queda cerrada.

### GB-005.06

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: frontera HTTP, configuración runtime, validación, errores, `requestId`, logs y pruebas base de `GameBook.Microservice.Game`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencias verificadas: `GB-005.04`, `GB-005.05` y `GB-004.04` figuran `[ RESOLVED ]`.
- Rama: `feature/005-06/prepare-game-api-logging` en `GameBook.Microservice.Game`.
- PR: [Game #7](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/7), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 05:25:38 UTC con merge commit `fb6efdbbf09e6802ba41cceb2da604c105a94c69`, destino `develop` y CI `repository-baseline` `SUCCESS` en ambos workflows.
- Cierre verificado: `origin/develop` contiene los módulos HTTP (`api-exception.filter.ts`, `request-id.ts`, `request-logging.middleware.ts`, `validation-pipe.ts`), `game-runtime-config.ts` y sus pruebas; el CI reportó 12 archivos de prueba y 50 pruebas unitarias, 1 prueba de integración, lint y build correctos. La configuración exige `GAME_DATABASE_URL`, `JWT_PUBLIC_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE` y `AUTHUSER_URL`, valida PEM/URLs y no expone secretos; GB-005.06 queda cerrada.

### GB-005.07

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: Action de migraciones Prisma del esquema `game` de `GameBook.Microservice.Game`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencias verificadas: `GB-005.03` y `GB-003.06` figuran `[ RESOLVED ]`.
- Rama: `feature/005-07/game-migration-action` en `GameBook.Microservice.Game`.
- PR: [Game #8](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/8), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 05:40:19 UTC con merge commit `651521d54cf374774fe0f400d980052b1ac079fa`, destino `develop` y CI `repository-baseline` `SUCCESS`.
- Evidencia de avance: el workflow validó el secreto/rol/esquema y el `GRANT` mínimo sobre `public._prisma_migrations` fue aplicado en Neon `develop` a `gamebook_game_migrator_limited` con `SELECT`, `INSERT`, `UPDATE` y `DELETE`. El reintento del run [35960978415](https://github.com/CarlosSV923/GameBook.Microservice.Game/actions/runs/35960978415) superó validación e idempotencia de metadatos, pero falló al aplicar la migración por `permission denied for database neondb`.
- Corrección en curso: rama `fix/005-07/recover-game-migration` y commit `6659a9d` (`fix(game): recover failed migration with limited role`). La migración deja de crear el esquema `game` preprovisionado y el workflow recupera condicionalmente el registro fallido con `prisma migrate resolve --rolled-back` antes de volver a ejecutar `migrate deploy`; no se amplían privilegios del migrador.
- PR correctivo: [Game #9](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/9), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 05:56:09 UTC con merge commit `ff9c8482d46cb7230a329cb3308084fe1e3ac5c5`, destino `develop`.
- Cierre verificado: el Action [run 35962148757](https://github.com/CarlosSV923/GameBook.Microservice.Game/actions/runs/35962148757) terminó `SUCCESS` en `develop`; `Validate migration inputs`, `Recover failed Game migration`, `Apply Game migrations` y `Verify migration idempotence` finalizaron correctamente. `GB-005.07` y el grupo `GB-005` quedan resueltos.

### GB-006.01

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: dirección visual documentada de `GameBook.Frontend`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencia verificada: `GB-002` figura `[ RESOLVED ]`.
- Rama: `feature/006-01/interface-direction` en `GameBook.Frontend`.
- PR: [Frontend #2](https://github.com/CarlosSV923/GameBook.Frontend/pull/2), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 05:58:38 UTC con merge commit `8248ce820a6c6a40c84ee44ca730d4b153ab3ec1`, destino `develop`. El CI `repository-baseline` terminó `SUCCESS` en las ejecuciones asociadas.
- Entregable verificado en `origin/develop`: `.interface-design/direction.md`, con dominio, mundo de color, firma visual de la tira de índice, composición, jerarquía, tokens conceptuales, estados, accesibilidad y patrones rechazados.
- Cierre verificado: el autor aprobó el concepto «archivo de juegos / guía de campo» y la firma de la «tira de índice» el 2026-09-24. Con el PR fusionado, el entregable presente en `origin/develop` y el CI `SUCCESS`, `GB-006.01` queda resuelta; las siguientes subtareas deben conservar esta dirección.

### GB-006.02

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: base Next.js y pnpm de `GameBook.Frontend`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencia verificada: `GB-006.01` figura `[ RESOLVED ]`.
- Rama: `feature/006-02/initialize-nextjs` en `GameBook.Frontend`.
- PR: [Frontend #3](https://github.com/CarlosSV923/GameBook.Frontend/pull/3), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 06:17:29 UTC con merge commit `320e9cc25492a5d4d497c8c18aa2e15197870b77`, destino `develop`. El CI `repository-baseline` terminó `SUCCESS` en las ejecuciones asociadas.
- Cierre verificado: `origin/develop` contiene la base Next.js, `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `features/`, `shared/`, las rutas servidoras reservadas para IGDB/Twitch y la configuración de aplicación; `README.md`, `README.es.md` y `CONTRIBUTING.md` ya no contienen referencias RAWG. `GB-006.02` queda resuelta.

### GB-006.03

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: sistema visual y preferencias persistentes de `GameBook.Frontend`.
- Inicio: 2026-09-24 (UTC; task separado).
- Dependencia verificada: `GB-006.02` figura `[ RESOLVED ]`.
- Rama: `feature/006-03/visual-system-preferences` en `GameBook.Frontend`.
- PR: [Frontend #4](https://github.com/CarlosSV923/GameBook.Frontend/pull/4), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-25 00:37:09 UTC con merge commit `6ddb0e3a82ee6af5113041e60aedab00d894919c`, destino `develop`. Los checks `repository-baseline` asociados terminaron `SUCCESS`.
- Cierre verificado: `origin/develop` contiene tokens visuales claro/oscuro, detección inicial del tema del sistema, persistencia manual de tema e idioma mediante `localStorage`, idioma inicial inglés, mensajes EN/ES, controles accesibles de preferencias, proveedor compartido y script de tema temprano para evitar parpadeo. `GB-006.03` queda resuelta.

### GB-006.06

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: verificación visual, responsive y CI de `GameBook.Frontend`.
- Inicio: 2026-09-25 (UTC; task separado).
- Dependencia verificada: `GB-006.05` figura `[ RESOLVED ]`.
- Rama: `feature/006-06/verify-visual-ci` en `GameBook.Frontend`.
- Commit: `1e7cc00` (`test(frontend): verify responsive visual baseline`).
- PR: [Frontend #7](https://github.com/CarlosSV923/GameBook.Frontend/pull/7), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 01:50:15 UTC con merge commit `8f78341d88d5d5644b86c423c38c7aafe015c5fe`, destino `develop`. Los dos checks `repository-baseline` asociados terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene la corrección del breakpoint móvil de la navbar, estilos `focus-visible` y pruebas de mensajes EN/ES; el PR documenta validación de formato, lint, tipos, 9 pruebas unitarias, build y revisión local en escritorio/móvil, ambos temas e idiomas y sin errores de consola.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-006.06` queda resuelta; el grupo `GB-006` queda resuelto.

### GB-007.01

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: registro de usuarios de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencia verificada: `GB-004` figura `[ RESOLVED ]`.
- Rama: `feature/007-01/register-user` en `GameBook.Microservice.AuthUser`.
- Commit: `a4c4328` (`feat(authuser): add user registration`).
- PR: [AuthUser #10](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/10), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-24 06:18:47 UTC con merge commit `d957367ee49e410a8e207a73313d3cb5b6a7e573`, destino `develop`. El CI `repository-baseline` terminó `SUCCESS` en las ejecuciones asociadas.
- Evidencia integrada: `origin/develop` contiene `POST /v1/auth/register` con validación de DTO, normalización de nombre/correo, política de contraseña, hash Scrypt, respuesta pública sin JWT/hash y mapeo de unicidad a `409 EMAIL_ALREADY_REGISTERED`. El PR incorporó 39 pruebas unitarias, 9 de integración, build NestJS, TypeScript de `src`, lint, Prettier y `git diff --check` correctos. `GB-007.01` queda resuelta.

### GB-007.02

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: inicio de sesión y emisión de JWT RS256 de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencia verificada: `GB-007.01` figura `[ RESOLVED ]`; `develop` local fue actualizado por fast-forward a `origin/develop` en el commit `d957367` antes de crear la rama de trabajo.
- Alcance: implementar exclusivamente el caso de uso y endpoint contractual `POST /v1/auth/login`, verificación del hash sin revelar si existe la cuenta, emisión de JWT RS256 con `sub`, `ver`, `iss`, `aud`, `iat` y `exp` de una hora, y respuesta pública con token e identidad. No iniciar `GB-007.03`, cambio de contraseña, Swagger final ni modificar Game o Frontend.
- Rama: `feature/007-02/login-jwt` en `GameBook.Microservice.AuthUser`.
- Commits: `9a454c1` (`feat(authuser): add login and JWT issuance`) y `cae8f89` (`chore(authuser): remove starter hello world endpoint`).
- PR: [AuthUser #11](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/11), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-25 00:34:55 UTC con merge commit `421aa85827646912311ea31b031cfb00eb20395d`, destino `develop`. Los checks `repository-baseline` asociados terminaron `SUCCESS`.
- Evidencia local: el endpoint de ejemplo `Hello World` y sus pruebas fueron retirados; la frontera real se valida mediante `/v1/auth/login` y CORS. Pasan 42 pruebas unitarias, 12 de integración HTTP, TypeScript de build, build NestJS, lint, Prettier de los archivos modificados y `git diff --check`. El lint conserva únicamente advertencias `unbound-method` en pruebas.
- Cierre verificado: `origin/develop` contiene el endpoint contractual de login, verificación del hash Scrypt, respuesta genérica `401 INVALID_CREDENTIALS`, emisión RS256 con `sub`, `ver`, `iat`, `exp`, `iss` y `aud` de una hora, respuesta pública de identidad y pruebas integradas. `GB-007.02` queda resuelta.

### GB-007.03

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: validación de sesión vigente de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencia verificada: `GB-007.02` figura `[ RESOLVED ]`; `develop` local fue actualizado por fast-forward a `origin/develop` en el merge commit `421aa85` antes de crear la rama de trabajo.
- Alcance: implementar exclusivamente `GET /v1/auth/session` con Bearer JWT, verificación RS256 de firma y claims, clasificación contractual de token ausente/inválido/expirado y comprobación de existencia y `sessionVersion` persistida. La respuesta exitosa expondrá solo la identidad pública. No iniciar `GB-007.04`, `GB-007.05`, ni modificar Game o Frontend.
- Rama: `feature/007-03/validate-session` en `GameBook.Microservice.AuthUser`.
- Commit: `baeb2a8` (`feat(authuser): validate current session`).
- PR: [AuthUser #12](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/12), fusionado hacia `develop`; el autor notificó y GitHub confirmó el 2026-09-25 00:51:05 UTC con merge commit `19dd5d5110c988d586d03b38f2f8c29a54ec5cab`, destino `develop`. Los checks `repository-baseline` asociados terminaron `SUCCESS`.
- Evidencia local: 46 pruebas unitarias, 18 de integración HTTP, TypeScript de build, build NestJS, lint, Prettier de los archivos modificados y `git diff --check` correctos. El lint conserva únicamente advertencias `unbound-method` en pruebas.
- Cierre verificado: `origin/develop` contiene `GET /v1/auth/session`, validación estricta de Bearer, verificación RS256 de claims y expiración, comparación de `sessionVersion`, respuesta pública de identidad y pruebas de token ausente, inválido, expirado y revocado. `GB-007.03` queda resuelta.

### GB-007.04

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: cambio de contraseña y revocación atómica de sesiones de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-24 (America/Guayaquil).
- Dependencia verificada: `GB-007.03` figura `[ RESOLVED ]`; `develop` local fue actualizado por fast-forward a `origin/develop` en el merge commit `19dd5d5` antes de crear la rama de trabajo.
- Alcance: implementar exclusivamente `PATCH /v1/users/me/password`, validación del Bearer vigente, comprobación genérica de la contraseña actual, política de la nueva contraseña, reemplazo atómico de hash e incremento de `sessionVersion`, respuesta `204` y revocación inmediata de tokens anteriores. No iniciar `GB-007.05`, `GB-007.06`, ni modificar Game o Frontend.
- Rama: `feature/007-04/change-password` en `GameBook.Microservice.AuthUser`.
- Commit: `77dcfbd` (`feat(authuser): change password and revoke sessions`).
- PR: [AuthUser #13](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/13), fusionado hacia `develop`; el autor avisó y GitHub confirmó el 2026-09-25 01:03:10 UTC con merge commit `529290b89aa115cc802721f10d90b71bf9cdb2c5`, destino `develop`. Los checks `repository-baseline` asociados terminaron `SUCCESS`.
- Evidencia local: 51 pruebas unitarias, 23 de integración HTTP, build NestJS, lint y `git diff --check` correctos; el lint conserva únicamente advertencias `unbound-method` en pruebas.
- Cierre verificado: `origin/develop` contiene `PATCH /v1/users/me/password`, verificación de contraseña actual, política de contraseña nueva, reemplazo atómico de hash e incremento de `sessionVersion`, respuesta `204` y pruebas de revocación/fallos de autenticación. `GB-007.04` queda resuelta.

### GB-007.05

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: documentación Swagger/OpenAPI y pruebas completas del API de `GameBook.Microservice.AuthUser`.
- Inicio: 2026-09-25 (America/Guayaquil).
- Dependencias verificadas: `GB-007.01`, `GB-007.02`, `GB-007.03` y `GB-007.04` figuran `[ RESOLVED ]`.
- Alcance: documentar el API con Swagger/OpenAPI y Bearer, mantener validaciones y errores sin secretos, y completar pruebas unitarias/integración reales de registro, unicidad, login, cambio de contraseña, expiración y revocación. No iniciar `GB-007.06`, ni modificar Game o Frontend.
- Rama: `feature/007-05/document-test-api` en `GameBook.Microservice.AuthUser`.
- Commits: `891c0ab` (`feat(authuser): document API and cover auth flows`), `2c440d1` (`refactor(authuser): use nest config service`), `e3f58b7` (`test(authuser): load app module after test config`) y `830e1ec` (`refactor(authuser): split Nest modules by layer`).
- PR: [AuthUser #14](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/14), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 01:55:14 UTC con merge commit `50fa3296037785762d6b44e42bd2cf248a366b8f`, destino `develop`. Los dos checks `repository-baseline` asociados terminaron `SUCCESS`.
- Evidencia local: `pnpm install --frozen-lockfile`, 51 pruebas unitarias, 27 de integración HTTP, build NestJS, lint y `git diff --check` correctos; el lint conserva únicamente advertencias `unbound-method` en pruebas. El ajuste final separa `api`, `application` e `infrastructure` en módulos NestJS.
- Cierre verificado: `origin/develop` contiene Swagger UI en `/docs`, OpenAPI JSON en `/docs/openapi.json`, seguridad Bearer, esquemas sin secretos, flujos HTTP completos de registro/login/expiración/cambio/revocación y la composición modular por capas. `GB-007.05` queda resuelta.

### GB-007.06

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: verificación local del servicio AuthUser y compatibilidad de su contrato OpenAPI.
- Inicio: 2026-09-25 (America/Guayaquil).
- Dependencia verificada: `GB-007.05` figura `[ RESOLVED ]`; `develop` local se actualizó por fast-forward a `origin/develop` en el merge commit `50fa329` antes de crear la rama de trabajo.
- Alcance: alinear el puerto local de AuthUser con el contrato `http://localhost:3001`, documentar los puertos/URLs de AuthUser, Game y Frontend en ambos README y comprobar localmente Swagger UI, OpenAPI, rutas contractuales, seguridad Bearer y ausencia de secretos. No iniciar `GB-008` ni modificar Game o Frontend.
- Rama: `feature/007-06/verify-local-contract` en `GameBook.Microservice.AuthUser`.
- Commit: `799a4b9` (`test(authuser): verify local contract and openapi`).
- PR: [AuthUser #15](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/15), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 02:16:10 UTC con merge commit `8fe2f45ef81ba22e97e014bee42aa051ab08ff43`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia local: `pnpm install --frozen-lockfile`, `pnpm db:generate` con `AUTH_DATABASE_DIRECT_URL` privado configurado para el comando, 51 pruebas unitarias, 27 de integración HTTP, build NestJS, lint, Prettier y `git diff --check` correctos. AuthUser arrancó localmente en `http://localhost:3001`; `/docs` y `/docs/openapi.json` respondieron `200`, las cuatro rutas contractuales estuvieron presentes, Bearer protegió sesión/contraseña, la validación HTTP devolvió `400 VALIDATION_ERROR` y el documento no expuso secretos.
- Cierre verificado: tras el aviso del autor, `origin/develop` contiene el puerto local `3001`, las URLs documentadas de los tres servicios, el OpenAPI compatible y la configuración de Swagger/Bearer comprobada localmente. GitHub confirmó el merge hacia `develop` y los checks verdes; `GB-007.06` y el grupo `GB-007` quedan resueltos.

### GB-008.02

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: listado, filtros y paginación de favoritos propios de `GameBook.Microservice.Game`.
- Inicio: 2026-09-25 (UTC; task separado).
- Dependencia verificada: `GB-005` figura `[ RESOLVED ]`.
- Rama: `feature/008-02/list-filter-paginate` en `GameBook.Microservice.Game`.
- Commits: `74e7c5e` (`feat(game): list and filter own favorites`) y `0297ef1` (`refactor(game): split NestJS modules by layer`).
- PR: [Game #11](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/11), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 01:55:10 UTC con merge commit `aadc7441957033847118a627909c283296daeceb`, destino `develop`. Los dos checks `repository-baseline` asociados terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene `GET /v1/favorites` autenticado, filtros AND por nombre/plataforma/año, rango inclusivo de lanzamiento, orden estable, paginación acotada, validación de rangos y módulos NestJS separados por capas. El PR documenta typecheck, build, lint, 56 pruebas unitarias y 7 de integración.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-008.02` queda resuelta. `GB-008` permanece activa por las subtareas pendientes.

### GB-008.03

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: sugerencias acotadas de nombre y plataforma sobre favoritos propios de `GameBook.Microservice.Game`.
- Dependencia verificada: `GB-008.02` figura `[ RESOLVED ]`.
- Rama: `feature/008-03/suggest-favorites` en `GameBook.Microservice.Game`.
- Commits: `e9909da` (`feat(game): add favorite suggestions`) y `9eb5b1f` (`fix(game): avoid spreading favorite query DTO`).
- PR: [Game #12](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/12), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 02:15:55 UTC con merge commit `7c9dbfd74382a4e02593b0e7cab71d4da786acfd`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene `GET /v1/favorites/suggestions`, validación y recorte de `q`, límite contractual, respuesta acotada por tipo y consulta, aislamiento por usuario mediante el repositorio Prisma existente y cobertura HTTP/use-case. El PR documenta typecheck, build, oxlint, 57 pruebas unitarias y 10 de integración.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-008.03` queda resuelta. `GB-008` permanece activa por las subtareas pendientes.

### GB-008.04

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: sincronización parcial de la instantánea IGDB de favoritos propios en `GameBook.Microservice.Game`.
- Dependencia verificada: `GB-008.01` figura `[ RESOLVED ]`.
- Rama: `feature/008-04/sync-snapshot` en `GameBook.Microservice.Game`.
- PR: [Game #13](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/13), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 02:34:52 UTC con merge commit `717980249348347b62cec2e3c075210a80c8c77b`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene `PATCH /v1/favorites/{igdbId}/snapshot`, actualización parcial únicamente del favorito propio existente, reemplazo atómico de plataformas mediante Prisma, preservación de campos omitidos, rechazo de instantáneas vacías/ inválidas y `404` para favoritos inexistentes. El PR documenta typecheck, build, oxlint, 62 pruebas unitarias y 13 de integración.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-008.04` queda resuelta. `GB-008` permanece activa por las subtareas pendientes.

### GB-008.05

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: integración real de JWT RS256, revocación y consulta de sesión AuthUser en `GameBook.Microservice.Game`.
- Dependencias verificadas: `GB-005` y `GB-007` figuran `[ RESOLVED ]`.
- Rama: `feature/008-05/integrate-jwt-revocation` en `GameBook.Microservice.Game`.
- Commit: `9941f23` (`test(game): integrate JWT revocation boundary`).
- PR: [Game #14](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/14), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 02:49:16 UTC con merge commit `4a03ca6d6600c17318659ec64f6f0cbbbcb4c40f`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene la prueba de frontera HTTP para validar la firma local, reenviar exactamente el Bearer recibido, consultar la sesión en AuthUser, distinguir `401`/`503`, rechazar tokens ausentes/manipulados/revocados y fallar cerrado cuando AuthUser no está disponible. Ambos README documentan `AUTHUSER_URL` para host y Compose. El PR documenta typecheck, lint, build, pruebas unitarias, E2E y comprobación Prettier.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-008.05` queda resuelta. `GB-008` permanece activa por las subtareas pendientes.

### GB-008.06

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: documentación Swagger/OpenAPI y pruebas completas del API protegido de `GameBook.Microservice.Game`.
- Dependencias verificadas: `GB-008.01`, `GB-008.02`, `GB-008.03`, `GB-008.04` y `GB-008.05` figuran `[ RESOLVED ]`.
- Rama: `feature/008-06/swagger-complete-tests` en `GameBook.Microservice.Game`.
- Commit: `c39dec8` (`feat(game): document API and cover authenticated flows`).
- PR: [Game #15](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/15), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 03:13:14 UTC con merge commit `9b95692133de749cb6dfe40cd2c589be151ae974`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene Swagger UI en `/docs`, OpenAPI JSON en `/docs/openapi.json`, seguridad Bearer, DTOs/esquemas/fixtures y respuestas contractuales. La suite HTTP cubre sesión válida, ausencia/manipulación/expiración/revocación del JWT, aislamiento por UUID y caída de AuthUser con `503`. El PR documenta `pnpm install --frozen-lockfile`, typecheck, lint, build, 62 pruebas unitarias, 21 E2E y Prettier.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-008.06` queda resuelta. `GB-008` permanece activa por `GB-008.07` pendiente.

### GB-008.07

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: verificación del servicio local y contrato OpenAPI de `GameBook.Microservice.Game`.
- Dependencia verificada: `GB-008.06` figura `[ RESOLVED ]`.
- Rama: `feature/008-07/verify-local-contract` en `GameBook.Microservice.Game`.
- PR: [Game #16](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/16), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 03:33:42 UTC con merge commit `e253ccea9c6927dbf9ba446bd655d868f456f701`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` documenta el puerto local `3002`, expone Swagger UI en `/docs` y OpenAPI JSON en `/docs/openapi.json`, publica metadatos OpenAPI 3.0.3 y documenta en los README EN/ES las URLs de AuthUser, Game, Frontend, Swagger UI y OpenAPI JSON. El PR acredita `/docs` `200`, `/docs/openapi.json` `200`, 62 pruebas, 21 E2E, typecheck, lint y build. La validación local usó configuración temporal en memoria para los endpoints públicos de Swagger; no modificó `.env`.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-008.07` queda resuelta. Todas las subtareas de `GB-008` están resueltas; el grupo `GB-008` queda resuelto.

### GB-009.01

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: integración server-only de Twitch/IGDB en `GameBook.Frontend`.
- Dependencia verificada: `GB-006` figura `[ RESOLVED ]`.
- Rama: `feature/009-01/igdb-server-integration` en `GameBook.Frontend`.
- Commit: `2ea1463` (`feat(frontend): add server-side IGDB proxy`).
- PR: [Frontend #8](https://github.com/CarlosSV923/GameBook.Frontend/pull/8), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 02:18:35 UTC con merge commit `70cf0f19d39d65d958f1cd374b212a3a34d8f348`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene las rutas server-only de catálogo, detalle y sugerencias de IGDB, cliente Twitch, renovación ante `401`/`403`, límites locales de 4 solicitudes por segundo y 8 concurrentes, mapeo seguro de errores, pruebas con mocks y documentación de las rutas en ambos README. El PR documenta 14 pruebas, typecheck, lint y build correctos.
- Evidencia adicional de cierre: el autor registró en el PR el smoke test real local exitoso para catálogo, detalle, sugerencias de juegos y sugerencias de plataformas con credenciales de desarrollo; no se registraron ni versionaron credenciales.
- Cierre verificado: tras el aviso del autor, el merge hacia `develop`, el entregable integrado, los checks verdes y el smoke test real local están acreditados. `GB-009.01` queda resuelta. `GB-009` permanece activa por las subtareas pendientes.

### GB-009.02

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: listado público y tarjetas iniciales del catálogo IGDB en `GameBook.Frontend`.
- Dependencias verificadas: `GB-006.06` y `GB-009.01` figuran `[ RESOLVED ]`.
- Rama: `feature/009-02/catalog-cards` en `GameBook.Frontend`.
- Commits: `da3bf1e` (`feat(frontend): add initial IGDB catalog cards`), `824ded7` (`fix(frontend): move preferences into navbar`), `7d22834` (`fix(frontend): restyle catalog cards`) y `fdfac45` (`fix(frontend): give catalog full content width`).
- PR: [Frontend #9](https://github.com/CarlosSV923/GameBook.Frontend/pull/9), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 03:14:40 UTC con merge commit `5c74ff7d73f6c2119d3860f1edccd4b11d2fa02b`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` conecta la vista pública al proxy server-only de IGDB y renderiza tarjetas responsive con imagen, nombre, año, plataformas y `total_rating`, además de estados de carga, vacío, error/reintento, imagen faltante y metadatos ausentes. Mantiene temas claro/oscuro e idiomas EN/ES, valida la respuesta y restringe el host de imágenes. El PR documenta 16 pruebas unitarias, typecheck, lint y build.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-009.02` queda resuelta. `GB-009` permanece activa por las subtareas pendientes.

### GB-009.03

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: filtros combinados y sugerencias de catálogo IGDB en `GameBook.Frontend`.
- Dependencias verificadas: `GB-009.01` y `GB-009.02` figuran `[ RESOLVED ]`.
- Rama: `feature/009-03/catalog-filters` en `GameBook.Frontend`.
- Commit: `70b2ead` (`feat(frontend): add catalog filters and suggestions`).
- PR: [Frontend #10](https://github.com/CarlosSV923/GameBook.Frontend/pull/10), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 03:33:38 UTC con merge commit `0fc23cf5fcd14eb7fcfc98a931be9f071b7b37b8`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene filtros combinados de nombre, plataforma y año/rango inclusivo, sugerencias de juegos y plataformas con cancelación de solicitudes obsoletas, validación de consultas, estado sin resultados y estilos responsive. El PR documenta typecheck, lint, pruebas unitarias y build.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-009.03` queda resuelta. `GB-009` permanece activa por las subtareas pendientes.

### GB-009.04

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: paginación incremental y desplazamiento infinito del catálogo IGDB en `GameBook.Frontend`.
- Dependencia verificada: `GB-009.03` figura `[ RESOLVED ]`.
- Rama: `feature/009-04/infinite-scroll` en `GameBook.Frontend`.
- Commit: `6242833` (`feat(frontend): add infinite catalog pagination`).
- PR: [Frontend #11](https://github.com/CarlosSV923/GameBook.Frontend/pull/11), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 03:43:00 UTC con merge commit `c9c6b87f0eb5c57499b013bd734e1d6156abefdb`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` carga páginas con `limit`/`offset`, detecta `hasNext`, usa `IntersectionObserver` y un botón accesible de alternativa, reinicia la paginación al cambiar filtros, cancela solicitudes obsoletas, deduplica por `igdbId` y muestra carga, reintento y fin del catálogo. El PR documenta 24 pruebas unitarias, typecheck, lint, build y `git diff --check`.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-009.04` queda resuelta. `GB-009` permanece activa por las subtareas pendientes.

### GB-009.05

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: modal de detalle de juegos IGDB en `GameBook.Frontend`.
- Dependencias verificadas: `GB-009.01` y `GB-009.02` figuran `[ RESOLVED ]`.
- Rama: `feature/009-05/game-detail-modal` en `GameBook.Frontend`.
- Commit: `2de56f2` (`feat(frontend): add game detail modal`).
- PR: [Frontend #12](https://github.com/CarlosSV923/GameBook.Frontend/pull/12), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 03:57:29 UTC con merge commit `8bc7c05b413a9e1b3ed846ba5e9746acd8bb3657`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene consulta server-side del detalle, `summary`, géneros, desarrolladores, screenshots y fecha con precisión real sin inventar datos; el modal es responsive, accesible por teclado, cierra por Escape/backdrop, conserva el foco, bloquea el scroll y muestra estados de carga/error. El PR documenta 27 pruebas unitarias, typecheck, lint, build y `git diff --check`.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-009.05` queda resuelta. `GB-009` permanece activa por las subtareas pendientes.

### GB-009.06

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: atribución a IGDB y estados de contenido externo en `GameBook.Frontend`.
- Dependencias verificadas: `GB-009.03`, `GB-009.04` y `GB-009.05` figuran `[ RESOLVED ]`.
- Rama: `feature/009-06/external-states` en `GameBook.Frontend`.
- Commit: `3aa27b7` (`feat(frontend): add IGDB attribution and external states`).
- PR: [Frontend #13](https://github.com/CarlosSV923/GameBook.Frontend/pull/13), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 04:12:02 UTC con merge commit `1ac594db874802544dcc259fd22da2333b98f9ad`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene el enlace visible a IGDB, fallbacks seguros para portadas y screenshots, y mensajes explícitos de carga/error/vacío para catálogo y detalle en inglés y español. El PR documenta typecheck, lint, pruebas unitarias, build y `git diff --check`.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-009.06` queda resuelta. `GB-009` permanece activa por `GB-009.07` pendiente.

### GB-009.07

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: pruebas y revisión final del catálogo IGDB en `GameBook.Frontend`.
- Dependencia verificada: `GB-009.06` figura `[ RESOLVED ]`.
- Rama: `feature/009-07/catalog-review` en `GameBook.Frontend`.
- Commit: `e049efa` (`test(frontend): review catalog external states`).
- PR: [Frontend #14](https://github.com/CarlosSV923/GameBook.Frontend/pull/14), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 04:20:40 UTC con merge commit `01a4a115874bd35ac71bfc505c316c75475e79e8`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene cobertura de fallos explícitos Twitch/IGDB, estados traducidos del catálogo, atribución segura, corrección responsive de la navbar y pruebas para estados, atribución y cliente IGDB. El PR documenta 32 pruebas unitarias, typecheck, lint, build, Prettier, `git diff --check` y revisión visual local en claro/oscuro y EN/ES.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-009.07` queda resuelta. Todas las subtareas de `GB-009` están resueltas; el grupo `GB-009` queda resuelto.

### GB-010.01

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: formularios de registro y login del `GameBook.Frontend`.
- Dependencia verificada: `GB-006` figura `[ RESOLVED ]`.
- Rama: `feature/010-01/auth-forms` en `GameBook.Frontend`.
- Commit: `924d76e` (`feat(frontend): add auth forms`).
- PR: [Frontend #15](https://github.com/CarlosSV923/GameBook.Frontend/pull/15), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 04:41:40 UTC con merge commit `38df05ae6583538a05fd89cdd36cd42dc272dbb4`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene las páginas `/register` y `/login`, campos accesibles, validación de email/contraseña/confirmación, reglas contractuales de contraseña, errores AuthUser traducidos y redirección exitosa a `/login?registered=1` sin iniciar sesión ni almacenar JWT. El PR documenta 14 archivos de pruebas, 37 pruebas, typecheck, lint, build, Prettier, `git diff --check` y revisión visual en claro/oscuro y EN/ES.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-010.01` queda resuelta. `GB-010` permanece activa por las subtareas pendientes.

### GB-010.02

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: gestión de sesión JWT y cliente AuthUser en `GameBook.Frontend`.
- Dependencia verificada: `GB-010.01` figura `[ RESOLVED ]`.
- Rama: `feature/010-02/auth-session` en `GameBook.Frontend`.
- Commit: `56fbff4` (`feat(frontend): manage auth session`).
- PR: [Frontend #16](https://github.com/CarlosSV923/GameBook.Frontend/pull/16), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 04:56:21 UTC con merge commit `5ba67031cef8e9aac4f7b12e1b65b1623114850d`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene `AuthProvider`, almacenamiento exclusivo del JWT opaque de AuthUser en `sessionStorage`, validación mediante `GET /v1/auth/session`, estado de usuario derivado de la respuesta válida, limpieza de sesiones inválidas `401`, redirección de login al catálogo público y Bearer en clientes protegidos de AuthUser/Game. OAuth de IGDB permanece server-only. El PR documenta 15 archivos de pruebas, 40 pruebas, typecheck, lint, build, Prettier y `git diff --check`.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-010.02` queda resuelta. `GB-010` permanece activa por las subtareas pendientes.

### GB-010.03

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: navegación consciente de sesión y perfil autenticado en `GameBook.Frontend`.
- Dependencia verificada: `GB-010.02` figura `[ RESOLVED ]`.
- Rama: `feature/010-03/navigation-profile` en `GameBook.Frontend`.
- Commit: `15ed482` (`feat(frontend): add navigation and profile`).
- PR: [Frontend #17](https://github.com/CarlosSV923/GameBook.Frontend/pull/17), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 05:08:53 UTC con merge commit `fe92d371ed82db8ab36220836ccb90b93e791f5e`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` muestra acciones distintas para visitantes y usuarios autenticados, enlaces a perfil/favoritos, identidad de solo lectura y formulario localizado de cambio de contraseña con contraseña actual y política contractual; un cambio exitoso revoca la sesión local y redirige a login con aviso traducido. El PR documenta 16 archivos de pruebas, 42 pruebas, typecheck, lint, build, Prettier, `git diff --check` y revisión local en navegador.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-010.03` queda resuelta. `GB-010` permanece activa por las subtareas pendientes.

### GB-010.04

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: ciclo de vida de sesión, expiración, revocación y logout en `GameBook.Frontend`.
- Dependencia verificada: `GB-010.03` figura `[ RESOLVED ]`.
- Rama: `feature/010-04/session-lifecycle` en `GameBook.Frontend`.
- Commits: `9311416` (`feat(frontend): handle auth session lifecycle`), `b5b39ba` (`fix(frontend): make home intro full width`) y `bf1540e` (`fix(frontend): expand home intro heading`).
- PR: [Frontend #18](https://github.com/CarlosSV923/GameBook.Frontend/pull/18), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 05:26:35 UTC con merge commit `b7b49bfcdc3ef1ee06d8fcd04b1247e50cff14a9`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` revalida la sesión cada 30 segundos, limpia el estado local ante `401`, redirige logout al catálogo público, propaga callbacks de no autorizado desde AuthUser/Game y limpia la sesión después del cambio de contraseña. Los errores `5xx` durante la comprobación no se convierten en credenciales inválidas. El PR documenta pruebas, typecheck, lint, Prettier y build.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-010.04` queda resuelta. `GB-010` permanece activa por las subtareas pendientes.

### GB-010.05

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: integración local real entre `GameBook.Frontend` y `GameBook.Microservice.AuthUser`.
- Dependencias verificadas: `GB-007` y `GB-010.04` figuran `[ RESOLVED ]`.
- Rama: `feature/010-05/authuser-local-integration` en `GameBook.Frontend`.
- Commit: `75a7cbd` (`docs(frontend): document local AuthUser integration`).
- PR: [Frontend #19](https://github.com/CarlosSV923/GameBook.Frontend/pull/19), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 05:37:54 UTC con merge commit `40b18e55f871997eeb6030e680ce1500a929c37e`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` documenta AuthUser en `http://localhost:3001`, Frontend en `http://localhost:3000`, `CORS_ALLOWED_ORIGINS` exacto, `NEXT_PUBLIC_AUTHUSER_URL` y reinicio requerido de Next.js. El PR acredita Swagger `200`, preflight permitido `204`, flujo real registro `201`→login `200`→sesión `200`→cambio `204`→token anterior rechazado `401`→nuevo login `200`, además del flujo de navegador hasta logout. La validación registra 16 archivos de pruebas, 44 pruebas, typecheck, lint, build, Prettier y `git diff --check`; las diferencias preexistentes de formato fuera del alcance no se modificaron.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y la integración local real, CORS y revocación están acreditados. `GB-010.05` queda resuelta. `GB-010` permanece activa por `GB-010.06` pendiente.

### GB-010.06

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: pruebas y revisión final de la experiencia de cuenta en `GameBook.Frontend`.
- Dependencia verificada: `GB-010.05` figura `[ RESOLVED ]`.
- Rama: `feature/010-06/account-review` en `GameBook.Frontend`.
- Commit: `468aeb4` (`test(frontend): review account experience`).
- PR: [Frontend #20](https://github.com/CarlosSV923/GameBook.Frontend/pull/20), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 05:44:31 UTC con merge commit `ec049a60d67147cfb47bee8a63946318a740cd95`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene cobertura de navegación autenticada para perfil/favoritos/logout, contratos estables de almacenamiento de tema/idioma y verificación de que todas las hojas de mensajes EN/ES existen y no están vacías. El PR documenta revisión desktop en claro/oscuro y EN/ES, responsive móvil con controles de 44px, el flujo local AuthUser de `GB-010.05`, 17 archivos de pruebas, 49 pruebas, typecheck, lint, build, Prettier y `git diff --check`. Las diferencias preexistentes del formato global quedaron fuera del alcance.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-010.06` queda resuelta. Todas las subtareas de `GB-010` están resueltas; el grupo `GB-010` queda resuelto.

### GB-011.01

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: acciones de favoritos condicionadas por sesión en `GameBook.Frontend`.
- Dependencias verificadas: `GB-009` y `GB-010` figuran `[ RESOLVED ]`.
- Rama: `feature/011-01/favorite-actions` en `GameBook.Frontend`.
- PR: [Frontend #21](https://github.com/CarlosSV923/GameBook.Frontend/pull/21), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 06:02:17 UTC con merge commit `5f2c6c3c0e2c32b9c5f88e4e3fcaf1ed2e3ed2b6`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene acciones de favorito condicionadas por sesión en tarjetas y modal; visitantes reciben botones de login/registro sin llamar a Game; usuarios autenticados pueden guardar/eliminar con estados de carga, éxito, duplicado y error. El PR documenta 17 archivos de pruebas, 58 pruebas, typecheck, lint, build, Prettier y `git diff --check`.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-011.01` queda resuelta. `GB-011` permanece activa por las subtareas pendientes.

### GB-011.02

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: vista personal de favoritos en `GameBook.Frontend`.
- Dependencia verificada: `GB-011.01` figura `[ RESOLVED ]`.
- Rama: `feature/011-02/personal-favorites` en `GameBook.Frontend`.
- PR: [Frontend #22](https://github.com/CarlosSV923/GameBook.Frontend/pull/22), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 06:54:08 UTC con merge commit `902782c1bf37d82be00effd68516e2dc56ee36ae`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene la ruta autenticada `/favorites`, tarjetas personales alimentadas por Game con el JWT de sesión, filtros, sugerencias y paginación incremental, reutilización de tarjetas/modal/estados/scroll infinito sin pedir detalle IGDB por tarjeta y redirección anónima. El PR documenta 56 pruebas, typecheck, lint, build, Prettier, `git diff --check` y smoke test local del estado anónimo.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-011.02` queda resuelta. `GB-011` permanece activa por las subtareas pendientes.

### GB-011.03

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: confirmación, guardado y eliminación de favoritos en `GameBook.Frontend`.
- Dependencias verificadas: `GB-011.01` y `GB-011.02` figuran `[ RESOLVED ]`.
- Rama: `feature/011-03/favorite-management` en `GameBook.Frontend`.
- PR: [Frontend #23](https://github.com/CarlosSV923/GameBook.Frontend/pull/23), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 07:16:46 UTC con merge commit `7a62cbd29cf4fefbb13b0d25d6ba51d99493d6ba`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` conserva el guardado autenticado, muestra el mensaje traducido de duplicado, solicita confirmación contextual antes de eliminar desde tarjeta o modal, elimina mediante Game con el JWT de sesión, actualiza la bandeja visible sin duplicados, cierra el modal afectado y muestra el estado vacío al eliminar el último favorito. El PR documenta 57 pruebas, typecheck, lint, build y `git diff --check`.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-011.03` queda resuelta. `GB-011` permanece activa por las subtareas pendientes.

### GB-011.04

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: sincronización del favorito al abrir el detalle en `GameBook.Frontend`.
- Dependencias verificadas: `GB-011.02` y `GB-009` figuran `[ RESOLVED ]`.
- Rama: `feature/011-04/detail-sync` en `GameBook.Frontend`.
- PR: [Frontend #24](https://github.com/CarlosSV923/GameBook.Frontend/pull/24), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 17:23:58 UTC con merge commit `517e550073bebf1cc329165f16728e9b97fa3724`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` obtiene el detalle completo de IGDB al abrir un favorito y sincroniza únicamente los campos del snapshot propiedad de Game tras una respuesta exitosa; actualiza la bandeja por `igdbId` sin duplicados, mantiene la acción de favorito/eliminación si IGDB falla e informa los estados de detalle y sincronización en el modal. El PR documenta 58 pruebas, typecheck, lint, build, Prettier dirigido y `git diff --check`; el drift de formato preexistente del repositorio queda fuera del alcance y los archivos modificados pasan la comprobación dirigida.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-011.04` queda resuelta. `GB-011` permanece activa por las subtareas pendientes.

### GB-011.05

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: integración local real entre `GameBook.Frontend` y `GameBook.Microservice.Game`.
- Dependencias verificadas: `GB-008`, `GB-011.03` y `GB-011.04` figuran `[ RESOLVED ]`.
- Rama: `feature/011-05/game-local-integration` en `GameBook.Frontend` y `GameBook.Microservice.Game`.
- PRs: [Frontend #25](https://github.com/CarlosSV923/GameBook.Frontend/pull/25), fusionado hacia `develop` el 2026-09-25 17:45:31 UTC con merge commit `a55a12d4991d818e9d864f9b71efbf0af9a366b8`; y [Game #17](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/17), fusionado hacia `develop` el 2026-09-25 17:45:34 UTC con merge commit `fac28b4076065cb76727d2e6f5274f79f7dbc427`. Ambos PRs tienen los dos checks `repository-baseline` en `SUCCESS`.
- Evidencia integrada: Frontend documenta `NEXT_PUBLIC_GAME_URL=http://localhost:3002`, el origen CORS exacto, el reinicio requerido tras cambiar la variable y las operaciones autenticadas con el JWT de AuthUser en README EN/ES. Game permite una lista explícita de orígenes mediante `CORS_ALLOWED_ORIGINS`, conserva `Authorization: Bearer` y `X-Request-Id`, y documenta la configuración local. La validación acredita en Game 65 pruebas, typecheck, lint, build, Swagger/OpenAPI `200`, preflight CORS `204` y flujo AuthUser→Game con JWT, duplicado `409`, filtros, sugerencias, snapshot, eliminación, revocación `401` y caída de AuthUser `503`; Frontend acredita 58 pruebas, typecheck, lint y build.
- Cierre verificado: ambos entregables están fusionados en `develop`, los checks están verdes y `GB-011.05` queda resuelta. `GB-011` permanece activa por `GB-011.06` pendiente.

### GB-011.06

- Estado: `[ RESOLVED ]`.
- Dueño: task separado de Codex, bajo coordinación de Codex (coordinación).
- Recurso: pruebas y revisión final de la experiencia de favoritos en `GameBook.Frontend`.
- Dependencia verificada: `GB-011.05` figura `[ RESOLVED ]`.
- Rama: `feature/011-06/favorites-review` en `GameBook.Frontend`.
- PR: [Frontend #26](https://github.com/CarlosSV923/GameBook.Frontend/pull/26), fusionado hacia `develop`; GitHub confirmó el 2026-09-25 18:11:35 UTC con merge commit `5368e59fba9c06771fd0423a0ce16eadc47217c5`, destino `develop`. Los dos checks `repository-baseline` terminaron `SUCCESS`.
- Evidencia integrada: `origin/develop` contiene cobertura bilingüe de estados visitante y autenticado, verifica que el cliente Game no envía `userId` y usa siempre el Bearer de AuthUser, y documenta la revisión automatizada y responsive en README EN/ES. El PR acredita 22 archivos de pruebas, 61 pruebas, typecheck, lint, build, Prettier dirigido y revisión manual local a 390px y 1440px; el drift de formato global preexistente queda fuera del alcance.
- Cierre verificado: el entregable está fusionado en `develop`, los checks están verdes y `GB-011.06` queda resuelta. Todas las subtareas de `GB-011` están resueltas; el grupo `GB-011` queda resuelto.

### GB-012.10

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios distribuidos en los tres repositorios de aplicaciones.
- Recurso: Docker Compose de integración local para `GameBook.Frontend`, `GameBook.Microservice.AuthUser` y `GameBook.Microservice.Game`.
- Dependencias verificadas: `GB-004.01`, `GB-005.01` y `GB-006.02` figuran `[ RESOLVED ]`.
- Ramas: `feature/012-10/compose-local-integration` en los tres repositorios.
- Commits: Frontend `4af7dae` (`feat(frontend): add local compose integration`), AuthUser `e1aaefc` (`chore(authuser): add compose development image`) y Game `6339d0f` (`chore(game): add compose development image`).
- PRs fusionados hacia `develop`: [Frontend #27](https://github.com/CarlosSV923/GameBook.Frontend/pull/27), fusionado el 2026-09-25 19:01:17 UTC con merge commit `1623eaecb1f0317ca5de7a823fb529d69621f023`; [AuthUser #16](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/16), fusionado el 2026-09-25 19:01:24 UTC con merge commit `2e3b6d5b1d8121cf7a86bae46841f0949130f408`; y [Game #18](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/18), fusionado el 2026-09-25 19:01:21 UTC con merge commit `fa7e700fa5c7b3ef97eb6fd1a33ab6716e7d6413`. Los dos checks `repository-baseline` de cada PR terminaron `SUCCESS`.
- Entregable preparado: Compose en Frontend con los tres clones hermanos, health checks de Swagger/Next.js, dependencias ordenadas, puertos `3000`/`3001`/`3002`, volúmenes de dependencias y sin PostgreSQL alternativo; Dockerfiles de desarrollo en ambos backends; plantillas privadas separadas por servicio; ningún secreto ni URL directa de migración versionado.
- Validaciones locales: `docker compose config --quiet`, `git diff --check`, Frontend 62 pruebas/typecheck/lint/build, AuthUser 51 pruebas/build/typecheck de producción/lint, Game 65 pruebas/typecheck/lint/build y generación Prisma con URLs sintácticas temporales sin conexión.
- Evidencia integrada verificada: `origin/develop` contiene `compose.yaml`, `Dockerfile.dev`, las tres plantillas privadas y la documentación EN/ES en Frontend; además de `Dockerfile.dev` en AuthUser y Game.
- Smoke test local completado con los tres archivos privados configurados sin registrar sus valores: `docker compose config --quiet` pasó; `docker compose up --build -d` reconstruyó las tres imágenes; AuthUser, Game y Frontend terminaron `healthy`; `http://localhost:3001/docs/openapi.json`, `http://localhost:3002/docs/openapi.json` y `http://localhost:3000/` respondieron `200`; Frontend alcanzó desde la red Compose los endpoints de AuthUser y Game; `docker compose down` apagó y retiró limpiamente el stack.
- Cierre verificado: los tres PR están fusionados en `develop`, sus checks están verdes, el entregable está integrado y el smoke test local demuestra reconstrucción, arranque ordenado, health checks, puertos, conectividad básica y apagado. `GB-012.10` queda resuelta; `GB-012` permanece activa por `GB-012.01` y las subtareas posteriores.

### GB-012.01

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: ejecución local de los contratos y flujo transversal sobre Docker Compose en `GameBook.Frontend`.
- Dependencias verificadas: `GB-007`, `GB-008`, `GB-009`, `GB-010`, `GB-011` y `GB-012.10` figuran `[ RESOLVED ]`.
- Inicio: 2026-09-25 (America/Guayaquil).
- Alcance de validación: visita pública sin JWT; registro; login; sesión y JWT compartido; guardado; filtros; detalle/sync; eliminación; cambio de contraseña; revocación; nuevo login; logout; Swagger; CORS; Neon `develop`; IGDB de pruebas y ausencia de migraciones/secretos en Compose.
- Evidencia de ejecución: `docker compose up -d` levantó AuthUser, Game y Frontend con sus health checks; los documentos OpenAPI de AuthUser y Game respondieron `200`.
- Flujo transversal completado con un usuario de prueba temporal y sin imprimir credenciales: catálogo público IGDB y detalle IGDB respondieron `200`; registro `201`; login `200` con JWT Bearer de una hora; sesión `200`; Game rechazó una consulta sin token con `401`; favorito creado `201`, listado filtrado `200`, sugerencias `200`, snapshot sincronizado `200`, eliminación `204` y ausencia confirmada en el listado posterior.
- Revocación y recuperación completadas: cambio de contraseña `204`; el JWT anterior fue rechazado por AuthUser y Game con `401`; nuevo login `200`; nueva sesión `200`. CORS desde `http://localhost:3000` respondió `204` y devolvió el origen exacto en AuthUser y Game.
- Frontend: `signOut` elimina el token de `sessionStorage`, limpia el usuario/estado y la navegación vuelve a `/`; la suite local pasó `22` archivos y `62` pruebas. `docker compose down` apagó y retiró limpiamente el stack. No se registraron secretos ni se ejecutaron migraciones desde Compose.
- Cierre verificado: el contrato transversal local quedó demostrado en `develop`, sin discrepancias de integración. `GB-012.01` queda resuelta; `GB-012` permanece activa por `GB-012.02` y las subtareas posteriores.

### GB-012.02

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Recurso: auditoría local de calidad, seguridad y aceptación del MVP sobre Docker Compose.
- Dependencia verificada: `GB-012.01` figura `[ RESOLVED ]`.
- Inicio: 2026-09-25 (America/Guayaquil).
- Alcance de auditoría: CORS y secretos locales; aislamiento por `sub`; fallo cerrado de AuthUser; Twitch/IGDB caído o limitado y renovación OAuth; accesibilidad, temas e idiomas; logs; Swagger; pruebas automatizadas; y arranque Compose desde cero.
- Seguridad estática: no hay archivos `.env`, PEM, claves ni secretos rastreados; no hay asignaciones de `JWT_PRIVATE_KEY`, `JWT_PUBLIC_KEY`, `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET`, `AUTH_DATABASE_URL` o `GAME_DATABASE_URL` en archivos versionados. Compose no define PostgreSQL alternativo, no recibe URLs directas de migración ni ejecuta migraciones.
- CORS y aislamiento live: el origen `http://localhost:3000` fue aceptado por AuthUser y Game; un origen no autorizado no recibió `Access-Control-Allow-Origin`. Con dos usuarios temporales, Game permitió crear/listar el favorito solo al propietario, no expuso datos al segundo usuario y devolvió `404` al intentar eliminarlo desde la cuenta ajena; la limpieza del favorito de prueba terminó con `204`.
- Fallo cerrado y externos: las suites cubren AuthUser no disponible (`503`), sesión revocada/incorrecta (`401`), Twitch/IGDB `401`/`403`/`429`/`5xx`, respuestas malformadas y renovación del token OAuth; todos los casos pasaron. El flujo live confirmó que la revocación de AuthUser bloquea también a Game.
- Calidad automatizada: Frontend `22` archivos/`62` pruebas, AuthUser `14`/`51` y Game `13`/`65`, todos verdes. Frontend y Game pasaron lint, typecheck y build; AuthUser pasó lint, build y `pnpm exec tsc --noEmit -p tsconfig.build.json`. AuthUser conserva únicamente advertencias existentes `unbound-method` en tests, sin fallos.
- Accesibilidad y preferencias: las pruebas de mensajes EN/ES, preferencias de tema/idioma, controles, sesión y revisión de favoritos pasaron; la revisión estática confirma etiquetas ARIA, estados live, foco/modal y HTML inicial `lang="en"`.
- Swagger, logs y Compose desde cero: `/docs` y `/docs/openapi.json` respondieron `200` en ambos backends; OpenAPI `3.0.0`/`3.0.3` expone `BearerAuth`. `docker compose down -v` eliminó los volúmenes del proyecto, `docker compose up --build -d` reconstruyó y levantó los tres servicios `healthy`, y los logs revisados no filtraron credenciales; el apagado final fue limpio.
- Cierre verificado: la auditoría local de calidad, seguridad y aceptación quedó completada sin incidencias bloqueantes. `GB-012.02` queda resuelta; `GB-012` permanece activa por `GB-012.03` y las subtareas posteriores.

### GB-012.03

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios distribuidos en los tres repositorios de aplicaciones.
- Recurso: `README.md` en inglés y `README.es.md` en español de `GameBook.Frontend`, `GameBook.Microservice.AuthUser` y `GameBook.Microservice.Game`.
- Dependencia verificada: `GB-012.02` figura `[ RESOLVED ]`.
- Inicio: 2026-09-25 (America/Guayaquil).
- Alcance: propósito, arquitectura realmente implementada, setup con `pnpm`, variables sin valores, pruebas, uso local, enlaces EN/ES y enlaces cruzados entre los tres repositorios; las URLs de producción no se inventan y quedan para `GB-012.09`.
- Avance comprobado: se actualizaron los seis README en ramas `feature/012-03/bilingual-readmes`, con commits iniciales `1cba31c` (Frontend), `afab8fa` (AuthUser) y `9300d7c` (Game), ajustes de ejecución individual `66db667`, `0fbed9b` y `f992aa4`, y los commits finales `4901c83`, `3955416` y `11a5d54`, respectivamente. Por decisión del autor, se eliminaron los artefactos versionados de Compose/Docker de los tres repositorios y se documentó únicamente la ejecución individual con `pnpm`; los archivos locales privados `compose.*.env` no se tocaron. Cada repositorio ahora incluye `.env.example` con nombres de variables y valores vacíos, incluyendo las variables directas de Prisma marcadas como exclusivas de migración; `.gitignore` permite publicar el ejemplo y mantiene privados los `.env` reales. Las validaciones pasaron `git diff --check`, nombres esperados, asignaciones vacías, referencias desde ambos README, enlaces de idioma, marcadores requeridos, ausencia de referencias Docker/Compose en archivos no secretos y ausencia de URLs de producción. Los tres PR fueron fusionados y verificados en `develop`; su CI `repository-baseline` está en `SUCCESS`.
- Cierre verificado (2026-09-25 UTC): el autor notificó los merges y GitHub confirmó `MERGED` hacia `develop` para Frontend PR #28 (merge `fae380bf95becc56f2d4b11aef7dd9f87945d27c`), AuthUser PR #17 (merge `a633aacf8996002c6c522814d2cfe8da1aa31ead`) y Game PR #19 (merge `ecee47b40d337ed29da81e3a92e828feb6dbf629`). Los CI `repository-baseline` de los tres PR terminaron `SUCCESS`. Tras actualizar las ramas locales, `develop` contiene los seis README bilingües, los tres `.env.example` sin valores y no contiene archivos versionados ni referencias Docker/Compose. GB-012.03 queda resuelta.

### GB-012.04

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios distribuidos en los tres repositorios de aplicaciones.
- Recurso: workflows de GitHub Actions, configuración de release-please y política de despliegue Git de Vercel en `GameBook.Frontend`, `GameBook.Microservice.AuthUser` y `GameBook.Microservice.Game`.
- Dependencia verificada: `GB-012.03` figura `[ RESOLVED ]` con los tres merges comprobados en `develop`.
- Inicio: 2026-09-25 (America/Guayaquil).
- Alcance: revisar y completar release-please apuntando a `main`, conservar CI con commits Conventional Commits, verificar checks de PR/release y permisos mínimos, y versionar la política `git.deploymentEnabled` de Vercel para permitir solo `main`, sin crear proyectos ni desplegar.
- Avance comprobado: los tres repositorios incluyen configuración manifest de release-please con la versión actual de `package.json`, workflow sobre pushes a `main`/ejecución manual usando `GITHUB_TOKEN` con `contents: write`, `issues: write` y `pull-requests: write`, validación de estos archivos dentro del CI y `vercel.json` con `**: false` y `main: true`. La validación local de JSON/YAML, versiones, política Vercel y `git diff --check` pasó; no se crearon proyectos ni deployments Vercel.
- Cierre verificado (2026-09-25 UTC): el autor notificó los merges y GitHub confirmó `MERGED` hacia `develop` para Frontend PR #29 (2026-09-25 20:41:08 UTC, merge `16631ecdedae09e5c614a0de74be8aab5d744aaa`), AuthUser PR #18 (2026-09-25 20:40:20 UTC, merge `49726539f105a93b38f4d2a27666a59117d80099`) y Game PR #20 (2026-09-25 20:40:56 UTC, merge `043f178503cd9dc9698d6f6e4623cce12a22a7dd`). Los checks `repository-baseline` de los tres PR terminaron `SUCCESS` (runs `36186331836`, `36186368578`; `36186336551`, `36186373561`; y `36186340736`, `36186376634`, respectivamente). Tras actualizar las ramas locales, `develop` contiene los workflows/configuraciones esperados en los tres repositorios; las comprobaciones de existencia, JSON, versiones, permisos/token y política Vercel pasaron. GB-012.04 queda resuelta; GB-012 permanece activa por las subtareas posteriores.

### GB-012.11

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), sin cambios de código ni creación, eliminación o despliegue de proyectos Vercel.
- Dependencias verificadas: `GB-012.02` y `GB-012.04` figuran `[ RESOLVED ]`.
- Inicio: 2026-09-25 (America/Guayaquil).
- Alcance: comprobar la eliminación manual de los tres proyectos Vercel históricos y preparar el inventario productivo sin imprimir valores, crear proyectos ni desplegar.
- Vercel verificado: `vercel whoami` confirmó la cuenta `carlossv923`; `vercel project ls` mostró únicamente `portafolio-personal` (`https://portafolio-personal-ecru-pi.vercel.app`), que no corresponde a GameBook. No aparecen `gamebook-microservice-authuser`, `gamebook-microservice-game` ni `gamebook-frontend`.
- Neon verificado: la CLI confirmó el proyecto `GameBook-db` (`little-shape-45554749`) y las ramas `develop` (`ready`) y `production` (`ready`), sin recuperar cadenas de conexión.
- Inventario productivo preparado por aplicación: AuthUser requiere `AUTH_DATABASE_URL`, `JWT_PRIVATE_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE` y `CORS_ALLOWED_ORIGINS`; Game requiere `GAME_DATABASE_URL`, `JWT_PUBLIC_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE`, `AUTHUSER_URL` y `CORS_ALLOWED_ORIGINS`; Frontend requiere `NEXT_PUBLIC_AUTHUSER_URL`, `NEXT_PUBLIC_GAME_URL`, `IGDB_CLIENT_ID` e `IGDB_CLIENT_SECRET`. `AUTH_DATABASE_DIRECT_URL` y `GAME_DATABASE_DIRECT_URL` quedan exclusivamente en GitHub Actions, nunca en Vercel.
- Herramienta local actualizada: `secrets/generate-jwt-keys.ps1` ahora solicita `develop` o `prod`, rechaza cualquier otra selección, funciona desde cualquier directorio y pide confirmación antes de sobrescribir archivos existentes. La sintaxis pasó validación sin ejecutar OpenSSL ni modificar llaves.
- Presets y política preparados: AuthUser/Game usarán preset NestJS; Frontend usará preset Next.js; los tres `vercel.json` integrados en `develop` habilitan despliegues Git únicamente para `main` (`**: false`, `main: true`). No hay proyectos Vercel activos de GameBook, por lo que no existe despliegue automático ni preview vigente.
- Cierre verificado (2026-09-25 UTC): el autor confirmó que las credenciales productivas JWT e IGDB/Twitch están generadas y separadas de las de desarrollo. `secrets/prod` contiene `jwt-private.pem`, `jwt-public.pem`, `jwt-keys.env` e `igdb-keys.env`; se comprobaron únicamente nombres y presencia de `JWT_PRIVATE_KEY`, `JWT_PUBLIC_KEY`, `IGDB_CLIENT_ID` e `IGDB_CLIENT_SECRET`, sin imprimir valores. La clave pública derivada de la privada coincide con `jwt-public.pem`. Neon `production` está disponible y lista; los tres proyectos Vercel históricos no aparecen en la cuenta, que conserva únicamente el proyecto ajeno `portafolio-personal`. La política versionada mantiene despliegues Git solo en `main`, con presets NestJS para los backends y Next.js para Frontend. No se crearon proyectos ni se ejecutaron despliegues. GB-012.11 queda resuelta; GB-012 permanece activa por las subtareas de publicación posteriores.

### GB-012.05

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con publicación de AuthUser condicionada al merge manual del autor.
- Dependencias verificadas: `GB-003`, `GB-004.07`, `GB-007` y `GB-012.11` figuran `[ RESOLVED ]`.
- Inicio: 2026-09-26 (America/Guayaquil).
- Alcance vigente: promover `develop` a `main`, ejecutar y verificar la migración `auth` de producción desde GitHub Actions, y publicar AuthUser en Render sin incluir configuración Vercel.
- Rama sincronizada: `develop` en `fb1bed7ccc5856f2e3652cb88c381f86d89a4e6c` (incluye la corrección de build); `main` permanece en `f0a16b00d91cc69b57300d629f955ce159220f2c`.
- Promoción verificada: [AuthUser #19](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/19), `develop → main`, fue fusionado el 2026-09-26 01:01:04 UTC con merge commit `fbc6b496ad2dd40d15908f9da3fafb35a15b647e`; su CI `repository-baseline` terminó `SUCCESS` en el run `36206826206`. El PR de release-please [AuthUser #20](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/20) fue fusionado el 2026-09-26 01:02:28 UTC con merge commit `190adbe9cf306036c9f60a9b15f28468690ad68a`; su CI terminó `SUCCESS` en el run `36206985185`. `origin/main` quedó verificado en `190adbe`.
- Migración productiva diagnosticada: el primer intento del workflow [run `36207150045`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/actions/runs/36207150045) validó que `AUTH_DATABASE_DIRECT_URL` está presente y usa `gamebook_auth_migrator_limited`, conectó a Neon `production` y falló con `permission denied for schema public` al inicializar `_prisma_migrations`. Tras añadir `schema=auth` al secreto y repetir el Action, Prisma pasó esa etapa, creó `auth._prisma_migrations` y falló al aplicar `20260924030000_init_auth_user` con `permission denied for database neondb`.
- Evidencia Neon del segundo fallo: `gamebook_auth_migrator_limited` tiene `USAGE`/`CREATE` en `auth`, no tiene `CREATE` sobre la base `neondb`, el esquema `auth` ya existe y `auth._prisma_migrations` contiene una fila de la migración con `finished=false` y `rolled_back=false`. `auth.User` y su índice no existen; no hay tablas de aplicación parcialmente creadas. No se modificaron grants ni datos desde la CLI.
- Diagnóstico y siguiente corrección: `schema=auth` resolvió el problema de ubicación del historial, pero la migración versionada todavía contiene `CREATE SCHEMA IF NOT EXISTS "auth"`, una operación que requiere `CREATE` en la base y contradice el migrador limitado. Debe retirarse esa sentencia porque `auth` se provisiona fuera de Prisma; después habrá que publicar la corrección y ejecutar `prisma migrate resolve --rolled-back 20260924030000_init_auth_user` antes de repetir `migrate deploy`. GB-012.05 permanece `[ ACTIVE ]`; no se concederá `CREATE` sobre `neondb`, no se repetirá el Action a ciegas y no se creará Vercel hasta recuperar la migración.
- Corrección integrada: [AuthUser #21](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/21) eliminó únicamente `CREATE SCHEMA IF NOT EXISTS "auth"` desde `fix/012-05/remove-auth-schema-create`, commit `1b602645c5d6cad448f88a9b3ebf6d1174dd2fa5`. GitHub confirmó el merge hacia `develop` el 2026-09-26 01:25:20 UTC con merge commit `c5a8b15a13e501fe1f5c942a867e9e11e3509eb6`; su CI `repository-baseline` terminó `SUCCESS` en el run `36208314268`. `pnpm db:validate` pasó con URL sintética y `git diff --check` pasó.
- Promoción verificada: [AuthUser #22](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/22), `develop → main`, fue fusionado el 2026-09-26 01:27:06 UTC con merge commit `2b950255201847bebbf6de8a6afc5dc3bc6e75d0`; el CI de promoción terminó `SUCCESS` en el run `36208417351` y la migración de `develop` terminó `SUCCESS` en el run `36208364226`. Release-please [AuthUser #23](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/23) fue fusionado el 2026-09-26 01:27:53 UTC con merge commit `f0a16b00d91cc69b57300d629f955ce159220f2c` y dejó `main` en `0.1.1`.
- Estado posterior: el Action productivo [run `36208582573`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/actions/runs/36208582573) falló con `P3009`, porque detectó la migración fallida anterior y bloqueó nuevas aplicaciones. Neon CLI confirmó una sola fila en `auth._prisma_migrations` para `20260924030000_init_auth_user`, con `finished_at=NULL`, `rolled_back_at=NULL` y `applied_steps_count=0`; `auth.User` aún no existe. No se modificaron datos ni permisos.
- Recuperación ejecutada (2026-09-26 01:33:05 UTC): se ejecutó de forma controlada `prisma migrate resolve --rolled-back 20260924030000_init_auth_user` contra Neon `production` usando `gamebook_auth_migrator_limited` y `schema=auth`; Prisma terminó con código `0`. Neon CLI confirmó `rolled_back_at` establecido, `applied_steps_count=0` y `auth.User` aún inexistente. No se concedieron permisos ni se modificaron datos de aplicación.
- Migración productiva completada (2026-09-26 UTC): el Action [run `36208582573`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/actions/runs/36208582573) terminó `SUCCESS`; `Validate migration inputs`, `Validate Prisma schema`, `Apply AuthUser migrations` y `Verify migration idempotence` pasaron sobre `main` (`f0a16b00d91cc69b57300d629f955ce159220f2c`). Neon CLI confirmó `auth.User` y `auth._prisma_migrations` existentes, la fila aplicada con `finished=true`, `rolled_back=false` y `applied_steps_count=1`, y la ejecución idempotente completada. La fila histórica revertida permanece registrada con `rolled_back=true`; no hay una tabla de aplicación duplicada.
- Incidencia de build Vercel diagnosticada: `nest build` falló porque el cliente generado en `src/infrastructure/persistence/prisma/generated` no se crea automáticamente, y `prisma.config.ts` exigía `AUTH_DATABASE_DIRECT_URL`, una variable exclusiva de GitHub Actions que no debe configurarse en Vercel.
- Corrección integrada: [AuthUser #24](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/24) añadió `pnpm db:generate` al script `build` y cambió Prisma a lectura opcional de `AUTH_DATABASE_DIRECT_URL`/`AUTH_DATABASE_URL`, permitiendo generar el cliente sin URL real durante el build. GitHub confirmó el merge hacia `develop` el 2026-09-26 01:49:54 UTC con merge commit `fb1bed7ccc5856f2e3652cb88c381f86d89a4e6c`; su CI `repository-baseline` terminó `SUCCESS` en el run `36209660959`. La validación local pasó `db:generate` sin variables de base, build, `db:validate` con URL sintética, lint, 14 archivos/51 pruebas y `git diff --check`.
- Promoción verificada: [AuthUser #25](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/25), `develop → main`, fue fusionado el 2026-09-26 01:52:51 UTC con merge commit `b1555661bae02640fce1522868424df51d7faaab`; su CI `repository-baseline` terminó `SUCCESS` en el run `36209811241`. La migración y el CI de `develop` también terminaron `SUCCESS` en los runs `36209727672` y `36209727721`.
- Vercel creado por el autor: proyecto `gamebook-ms-authuser`, ID `prj_Qw6RZr5LPBflFLjhk8AZJy6BCEd0`, preset NestJS, raíz `.`, Node `24.x`, sin comando de build/output explícitos (detección cero-config). Variables presentes solo por nombre y ámbito: `AUTH_DATABASE_URL`, `JWT_PRIVATE_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE` y `CORS_ALLOWED_ORIGINS` en `Production` y `Preview`; no aparece `AUTH_DATABASE_DIRECT_URL` en Vercel. El deployment Production `dpl_3yKe6QCKeNUEjxMAEnJHBNTDPLnY` y el Preview `dpl_Ggeaciz1i7PgzdfQsN3CedDgGw5c` figuran `READY`; sus alias confirman `main` y `develop`, respectivamente.
- Incidencia de runtime detectada: aunque el build y ambos deployments figuran `READY`, los logs de Vercel registran HTTP `500` en Production para `/`, `/docs`, `/docs/openapi.json` y `/v1/auth/register`; una solicitud directa a `/docs` no recibió respuesta dentro de 5 segundos. El deployment no se considera funcional hasta identificar y corregir la causa del `500`, y repetir Swagger, función y flujo de autenticación.
- Revalidación posterior (2026-09-26 UTC): el nuevo deployment Production `dpl_HPUwcgrKxpr23gjVMtQ2gKC8AL1e` figura `READY`, conserva preset NestJS/Node 24 y aliases de `main`, pero sus logs vuelven a registrar HTTP `500` para `/` y `/favicon.*`. Las variables requeridas mantienen los cinco nombres en `Production` y `Preview`, sin `AUTH_DATABASE_DIRECT_URL`; el Preview de `develop` responde con redirección SSO de protección. El runtime continúa sin aceptación.
- Causa confirmada por logs del runtime (2026-09-26 UTC): `ConfigModule.forRoot` aborta el arranque con `Missing required AuthUser configuration: AUTH_DATABASE_URL`. `vercel env ls` muestra la variable en `Production`, pero Vercel no permite extraer valores sensibles para verificar su contenido; por tanto, debe confirmarse en el panel que el valor no esté vacío y que esté asignado a `Production`, y crear un nuevo deployment después de guardarlo. Vercel aplica cambios de variables únicamente a deployments nuevos.
- Revalidación tras actualizar `AUTH_DATABASE_URL` (2026-09-26 UTC): Vercel creó el deployment Production `dpl_9n9Br2TBxmSwQyTs3GQLBRpMMufQ` (`READY`, alias `main`), pero sus logs continúan registrando HTTP `500` en `/`. `vercel env ls` sigue mostrando `AUTH_DATABASE_URL` en `Production`, aunque el valor permanece oculto; el runtime aún no recibe una configuración válida. GB-012.05 continúa bloqueada para cierre hasta confirmar el ámbito/valor efectivo del deployment y obtener respuestas exitosas.
- Comparación Preview/develop (2026-09-26 UTC): el deployment Preview `dpl_Ggeaciz1i7PgzdfQsN3CedDgGw5c` (`READY`, alias `develop`) también registra HTTP `500`. En el deployment Preview anterior `dpl_5EAMdS17q42iK9qejAS6PWuQxFhs`, Vercel expone el mismo error exacto `Missing required AuthUser configuration: AUTH_DATABASE_URL`; la protección SSO externa devuelve `302`, pero no corrige el fallo interno. Production y Preview/develop comparten la incidencia de configuración.
- Cambio de proveedor (2026-09-26 UTC): el autor decidió retirar Vercel del alcance de despliegue de AuthUser. Render está operativo y documentado en [`Swagger de AuthUser`](https://gamebook-microservice-authuser.onrender.com/docs); `/docs` y `/docs/openapi.json` devolvieron HTTP `200`. La raíz devuelve `404` porque no define una ruta. La evidencia Vercel anterior queda histórica y no es criterio de cierre vigente.
- Limpieza de proveedor integrada (2026-09-26 UTC): AuthUser eliminó `vercel.json`, la carpeta local `.vercel`, la validación Vercel del CI y las referencias Vercel de `README.md`/`README.es.md`. El PR [AuthUser #27](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/27), `fix/012-05/render-deployment` → `develop`, fue fusionado por el autor con merge commit `482c6718ca4093683652545980f0876bce051035`; su CI `repository-baseline` terminó `SUCCESS` (run `36215188266`). Las validaciones locales `lint`, `test` (14 archivos/51 pruebas), `test:e2e` (7 archivos/27 pruebas), `db:generate`, `build` y `git diff --check` pasaron.
- Promoción a producción integrada (2026-09-26 UTC): el PR [AuthUser #28](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/28), `develop` → `main`, fue fusionado por el autor con merge commit `d7f750cfa78f997c6cc23b8503f0f8278b54c0a5`; su CI `repository-baseline` terminó `SUCCESS` en los runs `36215363698` y `36215300526`. El workflow de release-please [run `36215394385`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/actions/runs/36215394385) también terminó `SUCCESS`, pero no creó PR: encontró `v0.1.2` como último release y clasificó los cambios posteriores (`chore` y merges) como no orientados al usuario, por lo que los omitió correctamente. No existe una nueva versión liberable pendiente.
- Render verificado por CLI y HTTP (2026-09-26 UTC): el servicio `GameBook.Microservice.AuthUser` está en el workspace `My Workspace`, rama `main`, auto-deploy por commit y URL `https://gamebook-microservice-authuser.onrender.com`. El deploy `dep-darjs2s9v7es73e8d8a0` está `live` y corresponde al commit `d7f750c`; los logs muestran `Build successful`, cliente Prisma generado, `Nest application successfully started` y `Your service is live`. `/docs` y `/docs/openapi.json` devolvieron `200`; `/v1/auth/session` sin token devolvió `401`; un `Origin` no permitido no recibió `Access-Control-Allow-Origin`. Los `404` de `/` corresponden a una ruta no definida y no a un fallo de arranque.
- Flujo productivo verificado (2026-09-26 UTC): se creó una cuenta temporal con datos sintéticos de prueba en Render Production; registro devolvió `201`, login `200` y `GET /v1/auth/session` con el Bearer emitido devolvió `200`. La respuesta indicó `tokenType=Bearer`, `expiresIn=3600` y el UUID de sesión coincidió con el usuario registrado. El preflight con un origen no permitido no devolvió `Access-Control-Allow-Origin`, por lo que el navegador lo bloquearía como exige la política vigente antes de `GB-012.07`. No se imprimieron contraseña ni JWT. La cuenta permanece en producción porque AuthUser no expone eliminación de usuarios.
- Cierre verificado: build, arranque, logs, función, migración productiva, Swagger/OpenAPI, login/sesión, JWT y denegación CORS quedaron comprobados en Render; el PR de limpieza Vercel y la promoción a `main` están fusionados. `GB-012.05` queda `[ RESOLVED ]`. La configuración del origen Frontend y el redeploy posterior pertenecen a `GB-012.07`.

### GB-012.06

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con merge manual del autor verificado.
- Dependencias verificadas: `GB-003`, `GB-005.07`, `GB-008` y `GB-012.05` figuran `[ RESOLVED ]`.
- Inicio: 2026-09-26 (America/Guayaquil).
- Alcance vigente: ejecutar y verificar la migración `game` de producción desde GitHub Actions, publicar Game en Render desde `main`, configurar sus variables runtime sin `GAME_DATABASE_DIRECT_URL` y probar JWT/AuthUser, Swagger, CORS y función.
- Rama sincronizada antes de iniciar: `GameBook.Microservice.Game/develop` limpia y alineada con `origin/develop` en `768ff6e`.
- PR de limpieza integrado (2026-09-26 UTC): [Game #21](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/21), `fix/012-06/render-deployment` → `develop`, fue fusionado por el autor con merge commit `0cb2960e4ed8b5b2701e6bf68219038c75c5ffcc`; su CI `repository-baseline` terminó `SUCCESS` (run `36216574891`). `vercel.json` está ausente de `origin/develop` y la validación Vercel del CI fue retirada.
- Promoción verificada (2026-09-26 UTC): [Game #22](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/22), `develop` → `main`, fue fusionado por el autor con merge commit `fd510e359cdd2f776e16999d12a0dc54eae0577f`; su CI `repository-baseline` terminó `SUCCESS` (run `36216722613`). Release-please [Game #23](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/23) también fue fusionado a `main` con release `0.1.0` y commit `9c061090f9b886bce9aa1f90858404cf74fe7f1d`.
- Migración productiva diagnosticada: el Action [run `36216937328`](https://github.com/CarlosSV923/GameBook.Microservice.Game/actions/runs/36216937328), sobre `main`/`9c061090f9b886bce9aa1f90858404cf74fe7f1d`, pasó validación de secretos y Prisma, pero falló en `Recover failed Game migration` antes de `migrate deploy` con `42P01: relation public._prisma_migrations does not exist`; los pasos de aplicación e idempotencia quedaron omitidos. Neon confirmó que `production` no contiene tablas ni historial parcial, mientras `develop` tiene `public._prisma_migrations` y las tablas `game.Favorite`/`game.FavoritePlatform`. La causa es que el workflow asume que el historial existe en el primer despliegue; debe tratar la ausencia de la tabla como “sin migración fallida” y continuar con `migrate deploy`. No se ejecutó recuperación ni se modificaron datos/permisos.
- Corrección integrada: [Game #24](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/24), `fix/012-06/recover-empty-migration-history` → `develop`, fue fusionado por el autor con merge commit `768ff6e314b5b62ecb28142a1e76da51f8a3eebd`; su CI `repository-baseline` terminó `SUCCESS` (run `36217443459`). La migración automática de `develop` [run `36217473171`](https://github.com/CarlosSV923/GameBook.Microservice.Game/actions/runs/36217473171) terminó `SUCCESS` y confirmó que la corrección no rompe el flujo con historial existente.
- Promoción verificada (2026-09-26 UTC): [Game #26](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/26), `develop` → `main`, fue fusionado por el autor con merge commit `a5b5e50833ec7b5426463788392d300ff2d1855f`; su CI `repository-baseline` terminó `SUCCESS` (run `36217531342`). Release-please [Game #27](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/27) también fue fusionado a `main`, dejando el release `0.1.1` en el commit `469f7ee`.
- Ajuste integrado en `develop`: [Game #28](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/28), `fix/012-06/prisma-generate-build` → `develop`, fue fusionado por el autor con merge commit `f54572d78cfdcdce24458fc2a99241814463eb45`; su CI `repository-baseline` terminó `SUCCESS` (run `36217858458`). El cambio añade `pnpm run db:generate && nest build` y actualiza `prisma.config.ts` para usar `GAME_DATABASE_DIRECT_URL` con respaldo en `GAME_DATABASE_URL`; `prisma validate` usando solo la variable runtime y `pnpm run build` terminaron `SUCCESS`.
- Promoción verificada: [Game #29](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/29), `develop` → `main`, fue fusionada por el autor con merge commit `30d0c65ca9e2de97d6da385fc40625fa42b3eed7`; CI y Release Please terminaron `SUCCESS` (runs `36218091679` y `36218091731`). [Game #30](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/30) de release-please fue fusionado a `main` con release `0.1.2`, commit `57a07f7a39b0a91a5e0ed62129428edf8145bb33`; CI y Release Please terminaron `SUCCESS` (runs `36218148948` y `36218148965`).
- Migración productiva final verificada en el `main` vigente `57a07f7`: el Action [run `36218546970`](https://github.com/CarlosSV923/GameBook.Microservice.Game/actions/runs/36218546970) terminó `SUCCESS`; `Validate migration inputs`, `Validate Prisma schema`, `Recover failed Game migration`, `Apply Game migrations` y `Verify migration idempotence` terminaron correctamente.
- Render verificado: servicio `GameBook.Microservice.Game`, rama `main`, documentación en [`Swagger de Game`](https://gamebook-microservice-game.onrender.com/docs), deploy `dep-darknagjo6nc738cgr60` `live` sobre el commit `57a07f7`; previews desactivadas. El build ejecutó `pnpm run db:generate && nest build`, generó Prisma Client `7.10.0`, terminó correctamente y los logs registraron `Nest application successfully started` y `Your service is live`. La raíz del servicio devuelve `404` porque no define una ruta.
- Smoke test productivo verificado sin exponer credenciales: AuthUser registro `201`, login `200` y sesión `200`; Game lista inicial `200`, crea favorito `201`, lista `200` con total `1`, elimina `204` y lista final `200` con total `0`. Swagger `/docs` y `/docs/openapi.json` respondieron `200`; solicitudes sin JWT respondieron `401`; un origen CORS no configurado no recibió `Access-Control-Allow-Origin`. La primera cuenta sintética del smoke test permaneció en producción porque AuthUser no expone eliminación de usuarios; la segunda se usó para completar el flujo y tampoco se imprimieron sus datos. `GB-012.06` queda `[ RESOLVED ]`; la configuración del origen Frontend pertenece a `GB-012.07`.

### GB-012.07

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con publicación y configuración productiva condicionadas al merge manual del autor.
- Dependencias verificadas: `GB-011` y `GB-012.06` figuran `[ RESOLVED ]`.
- Inicio: 2026-09-25 (America/Guayaquil).
- Alcance vigente: publicar `GameBook.Frontend` en Vercel desde `main`, configurar únicamente variables `Production`, fijar CORS exacto en AuthUser y Game con el dominio real de Frontend y verificar el flujo navegador→AuthUser/Game sin exponer secretos.
- Restricciones vigentes: no crear Vercel Preview, no configurar URLs de migración en Vercel, no imprimir valores de IGDB/Twitch/JWT/Neon y no avanzar `GB-012.08`.
- Promoción verificada (2026-09-26 UTC): [Frontend #30](https://github.com/CarlosSV923/GameBook.Frontend/pull/30), `develop` → `main`, fue fusionada por el autor con merge commit `80a9e365082f61b0d91b877235c86f0786849014`; su CI `repository-baseline` terminó `SUCCESS` (run `36219234840`). Las validaciones locales de tests (`62`), typecheck, lint y build terminaron correctamente; el `format:check` local solo refleja finales CRLF del checkout Windows y no una diferencia de contenido.
- Release verificado (2026-09-26 UTC): [Frontend #31](https://github.com/CarlosSV923/GameBook.Frontend/pull/31), release-please → `main`, fue fusionado con release `0.2.0` y merge commit `b44a527d36230c937b08373177aa4d2cf516007a`; CI y Release Please terminaron `SUCCESS` (runs `36219569308` y `36219569327`).
- Vercel verificado sin exponer valores: proyecto `gamebook-frontend`, URL [`https://gamebook-frontend.vercel.app`](https://gamebook-frontend.vercel.app), deployment Production `dpl_HHHgWAamD8fYqHA5wReSy6f4eVyZ` en estado `READY`, aliases de `main` activos y política `vercel.json` sin previews. Las variables `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET`, `NEXT_PUBLIC_AUTHUSER_URL` y `NEXT_PUBLIC_GAME_URL` existen únicamente en ámbito `Production`; los ámbitos `Preview` de las URLs de backend fueron retirados. La portada, `/login`, `/favorites`, `/api/igdb/games?page=1&pageSize=1` y `/api/igdb/platforms` respondieron `200`.
- CORS y flujo autenticado verificados (2026-09-26 UTC) desde el origen real `https://gamebook-frontend.vercel.app`: AuthUser y Game están `live` en Render sobre `main`; el preflight `OPTIONS` de `/v1/auth/session` y `/v1/favorites` respondió `204` con `Access-Control-Allow-Origin` exacto y headers/métodos contractuales. Una cuenta sintética temporal permitió comprobar registro `201`, login `200`, sesión AuthUser `200` y favoritos Game `200`; ambas respuestas autenticadas incluyeron `Access-Control-Allow-Origin` exacto. La cuenta sintética adicional permanece en producción porque AuthUser no expone eliminación de usuarios; no se imprimieron sus datos ni el JWT.
- Cierre verificado: merge de promoción, release, deployment Production `READY`, variables productivas sin Preview, CORS exacto y smoke test autenticado navegador→AuthUser/Game comprobados. `GB-012.07` queda `[ RESOLVED ]`; `GB-012.08` no se inició.

### GB-012.08

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación).
- Dependencias verificadas: `GB-012.07` figura `[ RESOLVED ]`; `GB-012.09` no se inició durante esta aceptación.
- Inicio: 2026-09-26 (America/Guayaquil).
- Alcance vigente: ejecutar la aceptación de producción contra Frontend en Vercel, AuthUser y Game en Render, demostrando los 16 criterios de `specs/mvp.md` con datos sintéticos y sin exponer secretos.
- Estrategia de evidencia: combinar recorrido funcional productivo, comprobaciones HTTP/OpenAPI/Swagger, pruebas de aislamiento y revocación, fallos controlados de IGDB/Twitch/AuthUser, comprobación visual de tema/idioma y revisión de logs/variables sin imprimir valores.
- Restricciones de aquella aceptación: no cambiar contratos ni código salvo que una incidencia reproducible lo exigiera, no modificar secretos, no ejecutar migraciones, mantener `GB-012.09` fuera del alcance y no publicar documentación final de `GameBook.System`.
- Aceptación productiva verificada (2026-09-26 UTC): Frontend Production [`https://gamebook-frontend.vercel.app`](https://gamebook-frontend.vercel.app) respondió `200` en `/`, `/login`, `/register`, `/favorites` y `/profile`; sus rutas server-side IGDB devolvieron catálogo, detalle, filtros por plataforma/año y sugerencias de plataforma. La UI mostró el catálogo ordenado, filtros, botón de carga adicional, detalle con atribución IGDB, redirección a login para guardar sin sesión y redirección de `/favorites` sin sesión.
- Criterios `1–2`: catálogo público, filtro por nombre en la UI, detalle y control de carga verificados; un visitante sin sesión fue enviado a `/login` al intentar guardar y al abrir `/favorites`.
- Criterios `3–8`: flujo sintético en producción AuthUser/Frontend verificado con registro `201`, login `200`, sesión `200`, cambio de contraseña `204`, y limpieza de sesión por revocación; Game confirmó creación `201`, duplicado `409`, snapshot `200`, eliminación `204` y lista final vacía. La implementación y pruebas de logout/limpieza de `sessionStorage` quedaron cubiertas por la suite Frontend `22` archivos/`62` pruebas.
- Criterios `9–10`: dos cuentas sintéticas demostraron aislamiento: la cuenta B vio `total=0` y no pudo eliminar el favorito de A (`404`); una petición sin JWT devolvió `401`. Tras cambiar la contraseña, el JWT anterior devolvió `401` en AuthUser y Game; los casos de JWT expirado están cubiertos por las suites unitarias de AuthUser/Game.
- Criterios `11–13`: catálogo, detalle y datos externos IGDB respondieron en producción; la UI cambió a tema claro y español, y ambos valores persistieron después de recargar. Las rutas y suites Frontend cubren las respuestas seguras de proveedor no disponible/`429`, sin provocar deliberadamente una cuota real en producción.
- Criterios `14–15`: `/docs` y `/docs/openapi.json` de AuthUser y Game respondieron `200`; ambos OpenAPI declaran `BearerAuth` y operaciones protegidas. Game aceptó el mismo JWT emitido por AuthUser y limitó la consulta al usuario autenticado.
- Criterio `16`: cambio de contraseña productivo invalidó inmediatamente el JWT anterior en AuthUser y Game; un nuevo login emitió un JWT funcional y pudo consultar Game.
- Seguridad y operación: CORS permitió únicamente `https://gamebook-frontend.vercel.app` (`OPTIONS 204`) y rechazó un origen externo sin `Access-Control-Allow-Origin`; las variables Vercel permanecen solo en `Production` y ocultas; el HTML público no contiene marcadores de `IGDB_CLIENT_SECRET`, token OAuth ni `JWT_PRIVATE_KEY`; Render no registró respuestas 5xx recientes.
- Calidad: `pnpm test` fuera del sandbox pasó en AuthUser (`14` archivos/`51` pruebas), Game (`13` archivos/`65` pruebas) y Frontend (`22` archivos/`62` pruebas`). CI/Release Please de `main` y las migraciones productivas vigentes están `SUCCESS` en GitHub; no se modificó código ni configuración durante la aceptación.
- Datos de prueba: se crearon cuentas sintéticas adicionales para validación; los favoritos de las pruebas se eliminaron. AuthUser no expone eliminación de usuarios, por lo que esas cuentas permanecen sin credenciales ni identificadores registrados en este archivo.
- Cierre verificado: los 16 criterios tienen evidencia productiva o cobertura automatizada de fallos controlados, no quedaron incidencias abiertas y `GB-012.09` no se inició. `GB-012.08` queda `[ RESOLVED ]`.

### GB-012.09

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios distribuidos en los tres repositorios de aplicaciones.
- Dependencia verificada: `GB-012.08` figura `[ RESOLVED ]`.
- Inicio: 2026-09-26 (America/Guayaquil).
- Alcance vigente: cerrar la versión y las notas de release de cada aplicación, sincronizar `main` con `develop`, incorporar las URLs productivas reales en `README.md` y `README.es.md` de cada aplicación, e inventariar la evidencia factual necesaria para la futura documentación Archify.
- Restricciones vigentes: no generar diagramas ni HTML Archify, no crear `GameBook.System`, no iniciar `GB-013`, no exponer secretos, y no alterar contratos o funcionalidades fuera de este cierre.
- Avance inicial: dependencias y seis README revisados; las URLs productivas de Frontend, AuthUser y Game quedaron incorporadas en ambos idiomas.
- Actualización Frontend (2026-09-26 UTC): el PR #32 incorpora el ajuste visual del autocompletado para que quede por encima de las tarjetas enfocadas, los botones `Search`/`Clear` y `Buscar`/`Limpiar`, y el selector de idioma mediante las iniciales del idioma destino (`ES`/`EN`) conservando etiquetas accesibles. El commit `c7b3ab1` fue publicado en el branch del PR; las validaciones locales (`pnpm typecheck`, `pnpm lint`, Prettier, `git diff --check` y `pnpm test`: 22 archivos/63 pruebas) pasaron y los checks `repository-baseline` de GitHub terminaron `SUCCESS` en los runs `36265521399` y `36265524002`. La comprobación visual local confirmó el comportamiento del dropdown en inglés y español. El commit posterior `854f379` hace que las sugerencias se oculten al perder su foco, conserva la selección por clic y añade el control reutilizable de mostrar/ocultar contraseña en acceso, registro y perfil, con etiquetas accesibles EN/ES. La validación local volvió a pasar (`pnpm typecheck`, `pnpm lint`, Prettier, `git diff --check` y `pnpm test`: 22 archivos/64 pruebas); la comprobación local confirmó el cierre al cambiar de campo y el cambio de `password` a `text` por teclado. GitHub confirmó `repository-baseline` `SUCCESS` en los runs `36266096197` y `36266098828`. El commit `6a028ce` retira los límites internos de `760px` y `52ch` de la introducción para que el titular y la descripción aprovechen todo el ancho editorial; la verificación visual local a `1163px` confirmó el resultado y las mismas validaciones pasaron. GitHub confirmó `repository-baseline` `SUCCESS` en los runs `36266438947` y `36266442613`. El commit `133db88` añade dentro del desplegable un indicador circular y el texto traducido `Searching suggestions…`/`Buscando opciones…` desde el debounce hasta que llega la respuesta; respeta `prefers-reduced-motion`. La validación local confirmó el estado pendiente y su sustitución por opciones; `pnpm typecheck`, `pnpm lint`, Prettier, `git diff --check` y `pnpm test` (22 archivos/65 pruebas) pasaron. GitHub confirmó `repository-baseline` `SUCCESS` en los runs `36266929034` y `36266930818`. El cambio quedó incluido en el PR #32, fusionado hacia `develop`.
- Actualización Frontend (2026-09-26 UTC): el commit `39cda98` limita el catálogo público y las sugerencias de nombre a juegos con el conjunto completo de metadatos que muestra cada tarjeta: título, portada, fecha de lanzamiento, valoración y plataformas. El adaptador de IGDB solicita esas condiciones al origen y vuelve a filtrarlas de forma defensiva antes de devolver resultados. Las validaciones locales `pnpm typecheck`, `pnpm lint`, Prettier, `git diff --check` y `pnpm test` (22 archivos/66 pruebas) pasaron; la prueba de regresión cubre registros incompletos en catálogo y sugerencias. GitHub confirmó `repository-baseline` `SUCCESS` en los runs `36267252871` y `36267256603`. El cambio quedó incluido en el PR #32, fusionado hacia `develop`.
- Actualización Frontend (2026-09-26 UTC): el commit `56da45c` añade indicadores animados y estados accesibles de espera en autenticación, cambio de contraseña, favoritos, borrado, detalle, paginación y carga de catálogo, respetando `prefers-reduced-motion`. El cliente HTTP reintenta hasta dos veces las lecturas `GET` ante errores de red, `408`, `425`, `429` o `5xx`, con backoff de `250 ms` y `750 ms`; no reintenta `POST`, `PATCH` ni `DELETE` para evitar duplicar mutaciones. Las validaciones locales `pnpm typecheck`, `pnpm lint`, Prettier, `git diff --check` y `pnpm test` (23 archivos/68 pruebas) pasaron, incluida cobertura de retry y de no-retry para mutaciones. GitHub confirmó `repository-baseline` `SUCCESS` en los runs `36268440968` y `36268443051`. El cambio quedó incluido en el PR #32, fusionado hacia `develop`.
- Promoción a `main` (2026-09-26 UTC): GitHub confirmó Frontend #32, AuthUser #29 y Game #31 fusionados hacia `develop` con commits `fd369da158707efebd7be7326b28d94bad5cf5d0`, `e233ed2277c5f3eb438d3c9c978f46d306fd9109` y `4625ef051a6e576409a58760a6817afec673ce83`. Los PR [Frontend #33](https://github.com/CarlosSV923/GameBook.Frontend/pull/33), [AuthUser #30](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/30) y [Game #32](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/32), todos `develop → main`, también quedaron `MERGED` con commits `3fa13bcfeed21551e252330f65309b44d3335c32`, `a2c62060778af962fb96a74f11ecdf27fd7a9f4d` y `8374c901fd8480e4537e8d63b80da6f4eae9fc93`; sus checks `repository-baseline` terminaron `SUCCESS` en los runs `36268998712`, `36268999728` y `36269002235`.
- Releases productivos ya verificados en GitHub: Frontend `v0.2.0` / tag `gamebook-frontend-v0.2.0` ([notas](https://github.com/CarlosSV923/GameBook.Frontend/releases/tag/gamebook-frontend-v0.2.0)), AuthUser `v0.1.2` / tag `gamebook-microservice-authuser-v0.1.2` ([notas](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/releases/tag/gamebook-microservice-authuser-v0.1.2)) y Game `v0.1.2` / tag `gamebook-microservice-game-v0.1.2` ([notas](https://github.com/CarlosSV923/GameBook.Microservice.Game/releases/tag/gamebook-microservice-game-v0.1.2)); las notas públicas fueron generadas por release-please y publicadas el 2026-09-26 UTC.
- Inventario factual para la futura GB-013, sin generar diagramas: Frontend — `src/app/api/igdb/` (rutas server-side), `src/server/igdb/` (Twitch/IGDB, límites y errores), `src/features/api/` (clientes AuthUser/Game), `src/features/` (UI y sesión) y `src/shared/`; AuthUser — `src/api/` (HTTP/OpenAPI), `src/application/`, `src/domain/`, `src/infrastructure/persistence/prisma/` (esquema/migración `auth`) y `src/infrastructure/cryptography/` (JWT RS256); Game — `src/api/` (favoritos/OpenAPI/JWT), `src/application/`, `src/domain/`, `src/infrastructure/auth/` (consulta de sesión AuthUser), `src/infrastructure/persistence/prisma/` (esquema/migración `game`) y `src/infrastructure/cryptography/` (verificación JWT). Evidencia transversal: `.github/workflows/ci.yml`, `release-please.yml` y los workflows de migración de cada backend; contratos `contracts/contract-baseline-v3.md` y `contracts/persistence-access-amendment-v2.md`; despliegue real Frontend/Vercel, AuthUser/Render, Game/Render, Neon runtime por esquema y Twitch OAuth→IGDB según `environments/deployment-policy-local-first.md` y la aceptación de `GB-012.08`.
- Cierre verificado (2026-09-26 UTC): los seis README (`README.md` y `README.es.md`) de los tres repositorios en `main` contienen las URLs productivas de Frontend/Vercel, AuthUser/Render y Game/Render; `main` contiene la promoción fusionada desde `develop`; las releases y notas públicas ya estaban verificadas; y el inventario factual para la futura documentación Archify quedó registrado sin generar diagramas ni iniciar `GB-013`. `GB-012.09` queda `[ RESOLVED ]`; `GB-012.12` se inició después y queda cerrada por separado.

### GB-012.12

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios previstos en AuthUser, Game y Frontend.
- Dependencia: `GB-012.09`, que debe figurar `[ RESOLVED ]` antes de iniciar esta subtarea.
- Alcance solicitado: implementar desactivación lógica desde Perfil; conservar la cuenta con un indicador booleano; invalidar sus JWT; bloquear login y registro con su correo con mensajes EN/ES; y hacer que Game rechace operaciones protegidas de la cuenta deshabilitada.
- Restricciones: no iniciar mientras `GB-012.09` siga abierta; no iniciar `GB-013`; no reactivar cuentas, purgar datos ni fijar nombres contractuales sin decisión explícita del autor.
- Decisiones registradas: el indicador booleano persistente se llamará `isDisabled`; no habrá reactivación en el MVP; la cuenta y sus favoritos se conservarán indefinidamente, sin purga automática ni física. La cuenta deshabilitada no podrá iniciar sesión ni registrar de nuevo su correo y recibirá el mensaje EN/ES previsto.
- Implementación local completada en AuthUser, Game y Frontend. La migración Prisma añade `auth.User.isDisabled`; `DELETE /v1/users/me` actualiza el indicador y `sessionVersion` atómicamente; login/registro y validación de sesión clasifican `ACCOUNT_DISABLED`; Game propaga el rechazo; Perfil ofrece confirmación bilingüe y cierre de sesión tras `204`.
- Evidencia local: AuthUser `pnpm build`, `pnpm db:validate`, lint y 59 pruebas pasan; Game typecheck, `pnpm db:validate`, lint y 66 pruebas pasan; Frontend build, typecheck, lint y 62 pruebas pasan. La comprobación dirigida de Prettier y `git diff --check` pasan en los cambios de la subtarea; el check global de Prettier conserva advertencias preexistentes fuera del alcance.
- PR fusionados hacia `develop`: AuthUser [#31](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/31) con merge `00a4072ef795581c2fa04499e059ae5b53c1976f`, Game [#33](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/33) con merge `d408e620bf71c50f6cd00315c6e6402547369f75` y Frontend [#35](https://github.com/CarlosSV923/GameBook.Frontend/pull/35) con merge `ca937b7a8475b997c292890f4aa0f57144ea56d2`; sus checks `repository-baseline` pasaron.
- Cierre verificado: con autorización del autor se transfirió en Neon `develop` la propiedad de `auth.User` al rol `gamebook_auth_migrator_limited`, usando temporalmente el rol administrativo histórico y retirando después ese otorgamiento. La migración fallida se marcó `rolled_back`, se aplicó de nuevo y el segundo `migrate deploy` fue idempotente; el run [`36270952454`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/actions/runs/36270952454) terminó `SUCCESS` en `Apply AuthUser migrations` y `Verify migration idempotence`. La columna `isDisabled BOOLEAN NOT NULL DEFAULT false` y el estado final de Prisma fueron verificados con el migrador limitado en `develop`. `GB-012.12` queda `[ RESOLVED ]`.

### GB-012.13

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios previstos únicamente en `GameBook.Frontend`.
- Dependencia: `GB-012.12`, que figura `[ RESOLVED ]`.
- Alcance solicitado: corregir la selección de sugerencias en los inputs de filtrado para que el clic o teclado no pierda la opción cuando el input deja de tener foco; conservar el nombre o plataforma elegidos y cerrar el desplegable.
- Restricciones: no iniciar `GB-013`, no modificar AuthUser ni Game y no alterar el contrato de filtros más allá de recuperar el comportamiento especificado.
- Causa reproducida en el código promovido: `SuggestionField` programa `setIsFocused(false)` en `onBlur` con `setTimeout(0)`, por lo que el desplegable puede desmontarse antes del `click` de su botón y `onSelect` nunca se ejecuta.
- Evidencia de cierre (2026-09-27 UTC): PR #38 fusionado a `develop`; PR #39 fusionado a `main` con commit `2cd4b0badb89265d3ebfdf1ca039acccb9ba6731`; release-please #40 fusionado con commit `6de4f55aa439440c3fc565dce6c901fe1d426a61` y tag `gamebook-frontend-v0.4.1`. CI `repository-baseline` pasó en los PR #39 y #40; el contenido final de `origin/main` fue comprobado tras el aviso del autor.

### GB-012.14

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios previstos únicamente en `GameBook.Microservice.AuthUser` y `GameBook.Frontend` si la evidencia lo exige.
- Dependencia: `GB-012.13`, que figura `[ RESOLVED ]`.
- Alcance solicitado: corregir el proceso de actualización de contraseña que en producción registra `401` para `PATCH /v1/users/me/password`, sin cambiar el contrato de autenticación ni debilitar la revocación de sesiones.
- Restricciones: no usar, copiar ni registrar credenciales reales; no iniciar `GB-013`; no modificar Game salvo evidencia contractual imprescindible; no ocultar un `401` convirtiéndolo en éxito.
- Evidencia inicial: Render registra una respuesta `401` en `PATCH /v1/users/me/password`; el diagnóstico debe distinguir token ausente/inválido/expirado/revocado de contraseña actual incorrecta y revisar el envío del Bearer desde Frontend.
- Diagnóstico realizado (2026-09-27 UTC): AuthUser local mapea correctamente `SessionValidationError` y `InvalidCurrentPasswordError` a `401`; la prueba dirigida del endpoint y del caso de uso pasó (5 pruebas), Frontend pasó sus pruebas dirigidas (7 pruebas) y la petición segura a Render con un Bearer sintético inválido devolvió `401 TOKEN_INVALID`. El cuerpo productivo compartido confirma `INVALID_CREDENTIALS`, por lo que AuthUser está rechazando la contraseña actual y no existe evidencia para convertir ese `401` en éxito. El cierre de sesión era un defecto del Frontend: trataba cualquier `401` como token inválido.
- Corrección implementada (2026-09-27 UTC): Frontend conserva la sesión cuando `PATCH /v1/users/me/password` devuelve `INVALID_CREDENTIALS`, permitiendo que el formulario muestre el error de contraseña actual; mantiene el cierre para `TOKEN_MISSING`, `TOKEN_INVALID`, `TOKEN_EXPIRED`, `SESSION_REVOKED` y `ACCOUNT_DISABLED`. Se añadieron pruebas de regresión para ambos grupos. Validación local: Vitest completo 24 archivos/71 pruebas, `typecheck`, `lint` y `git diff --check` correctos.
- Ajuste visual incluido (2026-09-27 UTC): después de un cambio exitoso, Frontend muestra en `/login?passwordChanged=1` un modal accesible EN/ES con confirmación, explicación del cierre de sesión, CTA para volver a iniciar sesión, foco inicial, Escape, bloqueo de scroll y animación reducida según preferencia del sistema. El CTA desmonta el modal y limpia la URL a `/login`; se añadió una prueba de renderizado. Validación local actualizada: Vitest completo 25 archivos/72 pruebas, `typecheck`, `lint`, `git diff --check` y comprobación visual local correctos.
- Evidencia de cierre (2026-09-27 UTC): PR Frontend [#41](https://github.com/CarlosSV923/GameBook.Frontend/pull/41), `feature/012-14/fix-password-change-production` → `develop`, fue fusionado con commit `9eb72a032320da81c04c6af93e537d34999ca532`; PR [#42](https://github.com/CarlosSV923/GameBook.Frontend/pull/42), `develop` → `main`, fue fusionado con commit `ebc94398c960ba579c13a1c2dd603fedbbb82f9a`. Sus checks `repository-baseline` terminaron `SUCCESS` en los runs `36343200701`, `36343203117`, `36343462285` y `36343502210`. El agente verificó `origin/main` en `d77c5d9` y confirmó la lógica de sesión para `INVALID_CREDENTIALS` y el modal de cambio exitoso. GB-012.14 queda `[ RESOLVED ]`.

### GB-012.15

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios previstos en `GameBook.Microservice.AuthUser`, `GameBook.Microservice.Game` y `GameBook.Frontend`.
- Dependencia: `GB-012.14`, que figura `[ RESOLVED ]`; `GB-013` no se inicia.
- Alcance solicitado: mitigar la espera de arranque en Render mediante un endpoint público `GET /health` en AuthUser y Game y una consulta previa desde Frontend antes de cada llamada a esos servicios.
- Decisiones registradas: cada intento del healthcheck esperará hasta 15 segundos; se permitirán como máximo 15 reintentos adicionales después del intento inicial; solo se reintentará el `GET /health`; la operación prevista no se ejecutará si no se obtiene HTTP `200`; el usuario no verá una interacción específica del healthcheck.
- Validación local (2026-09-27 UTC): AuthUser e2e 6 pruebas, suite completa 59 pruebas, build, lint y formato pasan; Game e2e 5 pruebas, suite completa 65 pruebas, typecheck, build, lint y formato pasan; Frontend suite completa 26 archivos/74 pruebas, typecheck, build, lint y formato pasan. Las pruebas de clientes comprueban el healthcheck previo, las de healthcheck comprueban HTTP `200` y el límite de 15 reintentos adicionales; los e2e comprueban endpoint público y OpenAPI sin seguridad.
- Evidencia de cierre (2026-09-27 UTC): AuthUser [#34](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/34) se fusionó en `develop` con `33a7c6438478ae542c9f8fc36afbf2cad3ff0b49`; Game [#36](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/36) con `c0ce96a7be59c5cec37607754eae3552f77c8ce7`; Frontend [#44](https://github.com/CarlosSV923/GameBook.Frontend/pull/44) con `c37976345612e4516c76c481c7ebec43762d081f`. Sus checks `repository-baseline` terminaron `SUCCESS`.
- Promoción verificada en GitHub: AuthUser [#35](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/35) se fusionó a `main` con `637c50574b754f7d7a514ef144c686ab3c7a1836`; Game [#37](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/37) con `095cacd14a824a8de2a5c464ed290d9409c9c947`; Frontend [#45](https://github.com/CarlosSV923/GameBook.Frontend/pull/45) con `ca7ae7efbb81732aed308acdaff36a0101689c12`. Los checks finales fueron `SUCCESS`; `origin/main` fue actualizado y comprobado en AuthUser `cba3b2a`, Game `23d9ed9` y Frontend `5fd3020`.
- Entregable final verificado en `main`: AuthUser y Game exponen `GET /health` público con HTTP `200` y documentación OpenAPI; Frontend consulta el healthcheck antes de todas las operaciones AuthUser/Game, aplica 15 segundos por intento y hasta 15 reintentos adicionales, y desactiva los reintentos genéricos de la operación original. `GB-012.15` queda `[ RESOLVED ]`; `GB-013` permanece `[ NEW ]`.

### GB-012.16

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios previstos únicamente en `GameBook.Frontend`.
- Dependencia: `GB-012.15`, que figura `[ RESOLVED ]`; `GB-013` no se inicia.
- Alcance solicitado: migrar toda la comunicación HTTP del Frontend desde el `fetcher` basado en Fetch hacia Axios como transporte y RxJS como capa interna de composición, incluyendo AuthUser, Game, healthchecks y las llamadas server-side de Next.js hacia IGDB/Twitch.
- Decisiones registradas: se preservarán contratos, URLs, métodos, cabeceras, JWT, cuerpos, mapeos de errores, cancelación, caché, estados de carga, secretos server-only y políticas de reintento. Los límites actuales podrán continuar exponiendo `Promise`; los componentes no tendrán que adoptar Observables. El healthcheck conservará 15 segundos por intento y hasta 15 reintentos adicionales; las mutaciones no se reintentarán por defecto.
- Restricciones: no modificar AuthUser ni Game; no alterar el comportamiento visible ni el contrato de IGDB; no exponer `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET`, tokens OAuth o JWT al navegador, logs, pruebas o repositorios; no iniciar `GB-013`.
- Implementación local: `package.json` y `pnpm-lock.yaml` incorporan Axios `1.20.0` y RxJS `7.8.2`; el adaptador común usa Axios + operadores RxJS y los clientes AuthUser, Game, healthcheck, proxy browser de IGDB y clientes server-side IGDB/Twitch ya no usan `fetcher`/Fetch. Los límites actuales siguen exponiendo Promises y los componentes no cambian.
- Validación local (2026-09-27 UTC): suite completa Frontend 26 archivos/75 pruebas, typecheck, lint, build, `git diff --check`, auditoría sin `fetcher`/`Fetcher`/`fetch(` en `src` y Prettier de archivos modificados pasan. El check global de Prettier conserva advertencias preexistentes fuera del alcance.
- Evidencia de cierre (2026-09-27 UTC): Axios y RxJS quedaron instalados en el Frontend; todas las rutas HTTP migradas y las pruebas de AuthUser/Game/healthcheck/IGDB/Twitch y cancelación/reintentos actualizadas. La suite local, typecheck, build, lint, `git diff --check`, auditoría de transporte y Prettier de archivos modificados pasan. El PR de implementación [#47](https://github.com/CarlosSV923/GameBook.Frontend/pull/47) se fusionó en `develop` con `b6091179b47d2ece99dfa26a34e5bfec053e8712` y `repository-baseline` `SUCCESS` (runs `36347740347`, `36347757705`). La promoción [#48](https://github.com/CarlosSV923/GameBook.Frontend/pull/48) se fusionó en `main` a las `20:27:19 UTC` con `f9735f1d4b4b6fe0dd983e63538af5da9b2b6eb4`; ambos checks `repository-baseline` terminaron `SUCCESS` (runs `36347859235`, `36347955065`) y `origin/main` fue verificado con ese commit.

### GB-012.17

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios previstos únicamente en `GameBook.Frontend` salvo que la evidencia contractual requiera otra cosa.
- Dependencia: `GB-012.16`, que figura `[ RESOLVED ]`; `GB-013` no se inicia.
- Alcance solicitado: cuando una persona autenticada cargue el catálogo completo, identificar visualmente cuáles juegos ya están guardados en sus favoritos. El mismo estado debe reflejarse en cada tarjeta del catálogo y en el modal de detalle del juego.
- Comportamiento de mutación: después de agregar un juego desde el catálogo, al intentar eliminarlo se debe solicitar confirmación. El modal de detalle debe mostrarlo como ya agregado y, al intentar eliminarlo desde allí, debe solicitar la misma confirmación de eliminación vigente.
- Decisión registrada: reutilizar exactamente el modal existente de confirmación, incluidos sus textos EN/ES y comportamiento, tanto desde el catálogo como desde el detalle; no crear un segundo patrón de confirmación.
- Criterios de cierre: estado de favoritos coherente entre catálogo y detalle; eliminación bloqueada hasta confirmar y sin ejecutarse al cancelar; actualización visual posterior a agregar/eliminar; usuarios no autenticados conservan el catálogo público sin estado privado de favoritos; pruebas, typecheck, lint y build del Frontend pasan; cambios promovidos y verificados en los destinos que correspondan.
- Restricciones: no iniciar `GB-013`; no modificar la semántica del modal existente ni inventar mensajes nuevos; no exponer JWT, secretos ni datos privados; no cambiar el contrato de favoritos sin documentar primero la decisión correspondiente.
- Implementación en progreso: el Frontend carga todos los IDs favoritos mediante páginas de `GET /v1/favorites` con el límite contractual de 1000, bloquea las acciones de guardado mientras se sincroniza, marca el corazón activo en las tarjetas y reutiliza el mismo estado en `GameDetailModal`. Guardar/quitar actualiza el conjunto local para mantener catálogo y detalle sincronizados; los usuarios anónimos conservan el botón para iniciar sesión. Se añadieron mensajes EN/ES de sincronización y error sin alterar la confirmación de eliminación existente.
- Validación local (2026-09-27 UTC): suite completa `pnpm test` (27 archivos/78 pruebas), `pnpm typecheck`, `pnpm lint`, `pnpm build`, `git diff --check` y Prettier de los archivos modificados pasan. El check global de Prettier conserva 59 advertencias preexistentes fuera del alcance. La inspección del catálogo local confirmó que los visitantes anónimos mantienen acciones accesibles de inicio de sesión.
- PR de implementación verificado: [#50](https://github.com/CarlosSV923/GameBook.Frontend/pull/50), `feature/012-17/catalog-favorite-state` → `develop`, fusionado como `ed28a7aa34aa9e6a14c9489b6cfd10ab0f953289`; ambos checks `repository-baseline` en `SUCCESS` (runs `36351191690` y `36351222659`).
- Promoción verificada: [#51](https://github.com/CarlosSV923/GameBook.Frontend/pull/51), `develop` → `main`, fusionada el 2026-09-27 a las 21:22:20 UTC con merge `7a633bcb052fbb8e54da47e176a892748952b360`; ambos checks `repository-baseline` en `SUCCESS` (runs `36351428872` y `36351366363`). El `main` actual `b5c66f646141a9b360a958f7bd3faa6a6dfb7bbe` fue verificado y contiene la promoción. `GB-012.17` queda `[ RESOLVED ]`.

### GB-012.18

- Estado: `[ RESOLVED ]`.
- Dueño: Codex (coordinación), con cambios previstos únicamente en `GameBook.Frontend` salvo que la evidencia contractual requiera otra cosa.
- Dependencia: `GB-012.17`, que figura `[ RESOLVED ]`; `GB-013` no se inicia.
- Alcance solicitado: reemplazar el `do...while` de `listAllFavoriteIds` por composición RxJS con `expand`, sin cambiar el contrato del cliente ni la experiencia de usuario.
- Criterios de cierre: páginas solicitadas en orden, la secuencia termina cuando `hasNext` es falso, la cancelación se propaga como abort error, los IDs se deduplican y se conserva `Promise<Set<number>>`; pruebas de paginación/cancelación, typecheck, lint y build del Frontend pasan; cambios promovidos y verificados en los destinos que correspondan.
- Restricciones: no modificar AuthUser ni Game, no cambiar contratos HTTP ni políticas de reintento, no introducir solicitudes paralelas, no exponer secretos ni iniciar `GB-013`.
- Implementación local: `listAllFavoriteIds` usa `defer` + `expand` con concurrencia `1` para solicitar las páginas secuencialmente, `EMPTY` para terminar por `hasNext`, `reduce` para deduplicar IDs y `firstValueFrom` para conservar `Promise<Set<number>>`. La cancelación previa o durante la expansión propaga `AbortError`; no se modificaron los contratos HTTP ni las políticas de reintento.
- Validación local (2026-09-27 America/Guayaquil / 2026-09-28 UTC): prueba dirigida y suite completa `pnpm test` (27 archivos/80 pruebas), `pnpm typecheck`, `pnpm lint`, `pnpm build`, Prettier de los archivos modificados y `git diff --check` pasan.
- PR de implementación verificado: [#53](https://github.com/CarlosSV923/GameBook.Frontend/pull/53), `feature/012-18/rxjs-favorite-pagination` → `develop`, fusionado el 2026-09-28 a las 01:45:11 UTC con merge `fb206863365246b4eca071615de844b8da1c7fdc`; ambos checks `repository-baseline` en `SUCCESS` (runs `36366144107` y `36366127805`).
- Promoción verificada: [#54](https://github.com/CarlosSV923/GameBook.Frontend/pull/54), `develop` → `main`, fusionada el 2026-09-28 a las 01:47:33 UTC con merge `f25c1ed74672d0732f654ee995d82b1f7ffa22ec`; ambos checks `repository-baseline` en `SUCCESS` (runs `36367183709` y `36367237803`). `main` fue verificado en ese mismo commit y contiene `src/features/catalog/catalog-favorites.ts` con la implementación RxJS. `GB-012.18` queda `[ RESOLVED ]`.

### GB-012.19

- Estado: `[ RESOLVED ]` (2026-09-29 UTC).
- Dueño: Codex (coordinación), con cambios limitados a `GameBook.Microservice.AuthUser` y su configuración/pruebas directamente afectadas.
- Dependencia: `GB-012.18`, que figura `[ RESOLVED ]`; `GB-012.20` se crea pero no se ejecuta en esta subtarea.
- Alcance: normalizar las extensiones de los imports relativos TypeScript de AuthUser desde `.js` a `.ts`, sin cambiar rutas de paquetes, contratos HTTP, lógica de negocio, configuración de runtime ni comportamiento funcional.
- Evidencia de cierre: inventario completo de imports, tests unitarios/E2E, lint, typecheck, build, formato, `git diff --check`, PR hacia `develop`, promoción a `main` y comprobación del contenido final. No resolver si alguna verificación falla o si el cambio exige alterar la funcionalidad.
- Implementación local: se normalizaron 182 imports relativos en 50 archivos de `src/` y `tests/`; no quedaron imports relativos `.js`. Se añadió `rewriteRelativeImportExtensions: true` en `tsconfig.json` para que el código fuente use `.ts` y el JavaScript emitido conserve `.js` para Node ESM; `dist/` fue comprobado sin imports `.ts`.
- Validación local (2026-09-28 UTC): `pnpm test` (15 archivos/59 pruebas), `pnpm test:e2e` (7 archivos/32 pruebas), `pnpm lint`, `pnpm exec tsc --noEmit -p tsconfig.build.json`, `pnpm build` y `git diff --check` pasan. Lint conserva advertencias `unbound-method` existentes. Prettier global reporta 37 archivos preexistentes fuera del alcance; el typecheck completo incluyendo tests conserva el problema preexistente de resolución `supertest/types`, mientras las suites Vitest pasan.
- PR de implementación: [AuthUser #39](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/39), `feature/012-19/authuser-ts-import-extensions` → `develop`, fusionado el 2026-09-29 a las 01:10:51 UTC con merge `6032519541afa8413b8c85cebeef0a919a9633d7`; ambos checks `repository-baseline` terminaron `SUCCESS` (runs `36506417500` y `36506447731`). La promoción [AuthUser #40](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/40), `develop` → `main`, fue fusionada el 2026-09-29 a las 01:13:40 UTC con merge `fb522e0b7172d9afa60409a8fa638b22c3a56983`; ambos checks `repository-baseline` terminaron `SUCCESS`. `main` fue verificado en ese commit. GB-012.19 queda `[ RESOLVED ]`.

### GB-012.20

- Estado: `[ RESOLVED ]` (2026-09-29 UTC).
- Dueño previsto: Codex (coordinación), con cambios limitados a `GameBook.Microservice.Game` y su configuración/pruebas directamente afectadas.
- Dependencia: `GB-012.18`, que figura `[ RESOLVED ]`; GB-012.19 también quedó `[ RESOLVED ]` antes de iniciar esta subtarea.
- Alcance: normalizar las extensiones de los imports relativos TypeScript de Game desde `.js` a `.ts`, sin cambiar rutas de paquetes, contratos HTTP, lógica de negocio, configuración de runtime ni comportamiento funcional.
- Evidencia de cierre: inventario completo de imports, tests unitarios/E2E, lint, typecheck, build, formato, `git diff --check`, PR hacia `develop`, promoción a `main` y comprobación del contenido final.
- Implementación local: se normalizaron 135 imports relativos en 40 archivos de `src/` y `tests/`; no quedaron imports relativos `.js`. Se añadió `rewriteRelativeImportExtensions: true` en `tsconfig.json` para que el código fuente use `.ts` y el JavaScript emitido conserve `.js` para Node ESM; `dist/` fue comprobado sin imports `.ts`.
- Validación local (2026-09-29 UTC): `pnpm test` (13 archivos/66 pruebas), `pnpm test:e2e` (3 archivos/22 pruebas), `pnpm lint`, `pnpm typecheck`, `pnpm exec tsc --noEmit -p tsconfig.build.json`, `pnpm build` y `git diff --check` pasan. `pnpm format:check` reporta 49 archivos preexistentes fuera del alcance; no se reformatearon para esta subtarea.
- PR de implementación: [Game #41](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/41), `feature/012-20/game-ts-import-extensions` → `develop`, fusionado el 2026-09-29 a las 01:21:50 UTC con merge `c5a666e91d90b134766002be7d7e7a01543d0e07`; ambos checks `repository-baseline` terminaron `SUCCESS` (runs `36507370178` y `36507387929`). La promoción [Game #42](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/42), `develop` → `main`, fue fusionada el 2026-09-29 a las 01:25:07 UTC con merge `b825daa6629ae83b9a2d1c801e7c0e086e59dfea`; ambos checks `repository-baseline` terminaron `SUCCESS`. `main` fue verificado en ese commit. GB-012.20 queda `[ RESOLVED ]` y GB-012 se cierra.

### GB-012.21

- Estado: `[ RESOLVED ]` (2026-09-29 UTC).
- Dueño previsto: Codex (coordinación), con cambios limitados a `GameBook.Frontend`.
- Dependencia: `GB-012.20`, que figura `[ RESOLVED ]`.
- Alcance: incorporar un alert bilingüe para el primer fallo de cualquier healthcheck de AuthUser o Game durante una interacción que requiera esos servicios. El aviso debe aparecer inmediatamente después del primer fallo; si el siguiente intento del mismo flujo responde correctamente, debe cerrarse con una transición gradual sin esperar 30 segundos; si no se recupera antes, debe cerrarse a los 30 segundos o por acción manual del usuario. Un primer healthcheck exitoso no debe generar alerta.
- Diseño: aplicar el sistema visual existente del Frontend (`.interface-design/system.md`): superficies y paleta semántica, tono editorial y tranquilo, claro/oscuro, textos EN/ES, responsive, botón de cierre accesible, foco/teclado/ARIA y `prefers-reduced-motion`. Reutilizar `CatalogState` o patrones existentes si encaja, sin duplicar primitivas.
- Restricciones: no cambiar los endpoints ni la política de reintentos del healthcheck, no mostrar alerta cuando el primer intento es exitoso, no bloquear el flujo ni exponer detalles técnicos al usuario, no iniciar otras subtareas.
- Evidencia de cierre: tests de estados primer fallo/recuperación/timeout/cierre manual/primer éxito, verificación visual EN/ES claro/oscuro y responsive, `pnpm typecheck`, `pnpm lint`, `pnpm test`, `pnpm build`, formato/diff y PR fusionado en sus destinos.
- Implementación local (2026-09-29 UTC): commit `8528370` en `feature/012-21/render-warmup-alert`. `waitForServiceHealth` emite el evento del primer fallo y de recuperación sin alterar el GET `/health`, los 15 reintentos ni el timeout de 15 segundos; el estado compartido agrupa AuthUser/Game, cierra por servicio a los 30 segundos o manualmente, y el componente global `ServiceWarmupAlert` mantiene la superficie accesible durante la transición de salida. Se añadieron textos EN/ES, estilos para claro/oscuro y móvil, foco/teclado, `aria-live="polite"`, `prefers-reduced-motion` y cobertura unitaria.
- Validación local (2026-09-29 UTC): `pnpm typecheck`, `pnpm lint`, Prettier dirigido a los archivos modificados, `git diff --check`, `pnpm test` (28 archivos/86 pruebas) y `pnpm build` pasan. La comprobación visual local mostró el aviso en español con tema oscuro y ventana estrecha, confirmó cierre manual y el texto accesible; la recuperación y el timeout se validaron con pruebas de estado. El Frontend quedó limpio en la rama de trabajo.
- PR de implementación: [Frontend #59](https://github.com/CarlosSV923/GameBook.Frontend/pull/59), `feature/012-21/render-warmup-alert` → `develop`, fusionado el 2026-09-29 a las 02:08:14 UTC con merge `6eb0ff56fbf338258fae10e7f5f41a24c93d0dd3`; ambos checks `repository-baseline` terminaron `SUCCESS`.
- PR de promoción: [Frontend #60](https://github.com/CarlosSV923/GameBook.Frontend/pull/60), `develop` → `main`, fusionado el 2026-09-29 a las 02:09:48 UTC con merge `cbfedb9d7ac94c79df7d9be190d8ae6ee5004ce3`; ambos checks `repository-baseline` terminaron `SUCCESS`. El contenido actual de `main` (`38e9a82cd9f73725108f066925f9d83af6581c61`) contiene `healthcheck.ts`, `service-warmup-alert.ts`, `service-warmup-alert.tsx` y sus pruebas. GB-012.21 queda `[ RESOLVED ]`.

## 8. Registro de PR

Todo PR de una subtarea de aplicación, publicación o release-please se anota aquí al crearse. El estado de GitHub se actualiza tras consulta; la verificación de merge y el cambio a `[ RESOLVED ]` ocurren solo después del aviso del autor. Si una subtarea necesita varios PR, **todos** deberán constar y estar fusionados en sus destinos antes de su cierre. Las tareas sin PR conservarán su evidencia en el registro de coordinación.

| Subtarea | Repositorio y PR | Origen → destino | Estado verificado en GitHub | Aviso del autor / verificación de cierre |
| --- | --- | --- | --- | --- |
| GB-001.04 | AuthUser [#1](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/1) | `feature/001-04/create-bilingual-readmes` → `develop` | `MERGED` (2026-09-23 04:14:53 UTC) | Autor avisó; agente verificó merge y ambos README en `develop` el 2026-09-23 UTC. |
| GB-001.04 | Game [#1](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/1) | `feature/001-04/create-bilingual-readmes` → `develop` | `MERGED` (2026-09-23 04:14:39 UTC) | Autor avisó; agente verificó merge y ambos README en `develop` el 2026-09-23 UTC. |
| GB-001.04 | Frontend [#1](https://github.com/CarlosSV923/GameBook.Frontend/pull/1) | `feature/001-04/create-bilingual-readmes` → `develop` | `MERGED` (2026-09-23 04:14:27 UTC) | Autor avisó; agente verificó merge y ambos README en `develop` el 2026-09-23 UTC. |
| GB-004.01 | AuthUser [#2](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/2) | `feature/004-01/initialize-nestjs-pnpm` → `develop` | `MERGED` (2026-09-24 01:35:10 UTC); commit `0e53b303de28b34829cb47441651fb9b6c13fc23`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit y entregable en `develop` el 2026-09-24 UTC. El fallo de Vercel Preview fue histórico y **ya no bloquea** `GB-004.01` ni se exige para `GB-004.06`. |
| GB-004.02 | AuthUser [#3](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/3) | `feature/004-02/model-user-interfaces` → `develop` | `MERGED` (2026-09-24 02:33:12 UTC); merge commit `0fe82c79a342841383af93247a9060ce3922e993`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregable en `origin/develop` el 2026-09-24 UTC. |
| GB-004.03 | AuthUser [#4](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/4) | `feature/004-03/create-auth-schema-migration` → `develop` | `MERGED` (2026-09-24 02:52:23 UTC); merge commit `f0b13b769385a4a2e06789f45defe70ca0c3046a`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit y CI en GitHub. |
| GB-004.03 | AuthUser [#5](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/5) | `feature/004-03/create-auth-schema-migration` → `develop` | `MERGED` (2026-09-24 03:29:56 UTC); merge commit `65024ea9cd7b0aadee1608ad81502ead467cb478`; CI `repository-baseline` `SUCCESS` | PR de ajustes del mismo entregable; autor avisó y agente verificó merge, rama de destino, commit, CI y rutas finales en `origin/develop`. |
| GB-004.04 | AuthUser [#6](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/6) | `feature/004-04/implement-base-adapters` → `develop` | `MERGED` (2026-09-24 03:54:21 UTC); merge commit `3fce0efba61b3a39c14eb26e5b16718edd15e68b`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-004.05 | AuthUser [#7](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/7) | `feature/004-05/prepare-api-logging` → `develop` | `MERGED` (2026-09-24 04:19:10 UTC); merge commit `ec902a8436d44dd499c31ffd90e34395e0b7a224`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-004.06 | AuthUser [#8](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/8) | `feature/004-06/verify-local-base` → `develop` | `MERGED` (2026-09-24 05:14:09 UTC); merge commit `013fc786b9b935b11993dbdff2be8ffab3c1990c`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-004.07 | AuthUser [#9](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/9) | `feature/004-07/auth-migrations-action` → `develop` | `MERGED` (2026-09-24 05:24:44 UTC); merge commit `6d51226997b796d84c6a76d672b2fe55f1fadea6`; CI `repository-baseline` `SUCCESS`; Action `SUCCESS` (run `35959824069`) | Autor avisó; agente verificó merge, rama de destino, entregable y migración idempotente en `develop`. |
| GB-005.01 | Game [#2](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/2) | `feature/005-01/initialize-nestjs-pnpm` → `develop` | `MERGED` (2026-09-24 02:46:06 UTC); merge commit `4cc67509bebeb5a0ad0fbae66aa5ca515f041613`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop` el 2026-09-24 UTC. |
| GB-005.02 | Game [#3](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/3) | `feature/005-02/model-favorite-interfaces` → `develop` | `MERGED` (2026-09-24 03:49:20 UTC); merge commit `7d2039dd5e9c50698d01755df09c5be4eff64641`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-005.03 | Game [#4](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/4) | `feature/005-03/create-game-schema-migration` → `develop` | `MERGED` (2026-09-24 04:20:04 UTC); merge commit `c6dadf1a600cfccce4651a0ff9a91fa18ae752ef`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-005.04 | Game [#5](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/5) | `feature/005-04/implement-favorite-repository` → `develop` | `MERGED` (2026-09-24 04:46:47 UTC); merge commit `1225b94f5a6fb86dcdd36c584e7022df08a24acb`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-005.05 | Game [#6](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/6) | `feature/005-05/jwt-auth-guard` → `develop` | `MERGED` (2026-09-24 05:01:08 UTC); merge commit `37476e56bffad68abdfd4f42748d9709a2f51c5c`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-005.06 | Game [#7](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/7) | `feature/005-06/prepare-game-api-logging` → `develop` | `MERGED` (2026-09-24 05:25:38 UTC); merge commit `fb6efdbbf09e6802ba41cceb2da604c105a94c69`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, commit, CI y entregables en `origin/develop`. |
| GB-005.07 | Game [#8](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/8) | `feature/005-07/game-migration-action` → `develop` | `MERGED` (2026-09-24 05:40:19 UTC); merge commit `651521d54cf374774fe0f400d980052b1ac079fa`; CI `repository-baseline` `SUCCESS`; Action `FAILURE` (run `35960978415`) | Autor avisó; agente verificó merge y rama de destino. Se corrigió el permiso de metadatos, pero persiste `permission denied for database neondb`; tarea permanece activa. |
| GB-005.07 | Game [#9](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/9) | `fix/005-07/recover-game-migration` → `develop` | `MERGED` (2026-09-24 05:56:09 UTC); merge commit `ff9c8482d46cb7230a329cb3308084fe1e3ac5c5`; Action `SUCCESS` (run `35962148757`) | Autor avisó; agente verificó merge, rama de destino, entregable y aplicación/idempotencia de migraciones en `develop`. |
| GB-006.01 | Frontend [#2](https://github.com/CarlosSV923/GameBook.Frontend/pull/2) | `feature/006-01/interface-direction` → `develop` | `MERGED` (2026-09-24 05:58:38 UTC); merge commit `8248ce820a6c6a40c84ee44ca730d4b153ab3ec1`; CI `repository-baseline` `SUCCESS` | Autor avisó y aprobó la dirección el 2026-09-24; agente verificó merge, rama de destino, `.interface-design/direction.md` y CI. |
| GB-006.02 | Frontend [#3](https://github.com/CarlosSV923/GameBook.Frontend/pull/3) | `feature/006-02/initialize-nextjs` → `develop` | `MERGED` (2026-09-24 06:17:29 UTC); merge commit `320e9cc25492a5d4d497c8c18aa2e15197870b77`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, base Next.js/pnpm, README sin RAWG y CI en `develop`. |
| GB-006.03 | Frontend [#4](https://github.com/CarlosSV923/GameBook.Frontend/pull/4) | `feature/006-03/visual-system-preferences` → `develop` | `MERGED` (2026-09-25 00:37:09 UTC); merge commit `6ddb0e3a82ee6af5113041e60aedab00d894919c`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, preferencias visuales EN/ES persistentes y CI en `develop`. |
| GB-006.04 | Frontend [#5](https://github.com/CarlosSV923/GameBook.Frontend/pull/5) | `feature/006-04/api-adapters-mocks` → `develop` | `MERGED` (2026-09-25 01:01:37 UTC); merge commit `1451b1fd319ebf86566e2f5712c70bf0520bc3d6`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, adaptadores/mocks tipados y CI en `develop`. |
| GB-006.05 | Frontend [#6](https://github.com/CarlosSV923/GameBook.Frontend/pull/6) | `feature/006-05/navigation-components` → `develop` | `MERGED` (2026-09-25 01:35:52 UTC); merge commit `28e0feac91180be648986ae6c614ec285c90fd6b`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, navegación/componentes base y CI en `develop`. |
| GB-006.06 | Frontend [#7](https://github.com/CarlosSV923/GameBook.Frontend/pull/7) | `feature/006-06/verify-visual-ci` → `develop` | `MERGED` (2026-09-25 01:50:15 UTC); merge commit `8f78341d88d5d5644b86c423c38c7aafe015c5fe`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, corrección responsive/focus-visible, pruebas visuales declaradas y CI en `develop`. |
| GB-007.01 | AuthUser [#10](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/10) | `feature/007-01/register-user` → `develop` | `MERGED` (2026-09-24 06:18:47 UTC); merge commit `d957367ee49e410a8e207a73313d3cb5b6a7e573`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, endpoint de registro y CI en `develop`. |
| GB-007.02 | AuthUser [#11](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/11) | `feature/007-02/login-jwt` → `develop` | `MERGED` (2026-09-25 00:34:55 UTC); merge commit `421aa85827646912311ea31b031cfb00eb20395d`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, login/JWT contractual y CI en `develop`. |
| GB-007.03 | AuthUser [#12](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/12) | `feature/007-03/validate-session` → `develop` | `MERGED` (2026-09-25 00:51:05 UTC); merge commit `19dd5d5110c988d586d03b38f2f8c29a54ec5cab`; CI `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, sesión/JWT contractual y CI en `develop`. |
| GB-007.04 | AuthUser [#13](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/13) | `feature/007-04/change-password` → `develop` | `MERGED` (2026-09-25 01:03:10 UTC); merge commit `529290b89aa115cc802721f10d90b71bf9cdb2c5`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, cambio de contraseña, revocación atómica y CI en `develop`. |
| GB-007.05 | AuthUser [#14](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/14) | `feature/007-05/document-test-api` → `develop` | `MERGED` (2026-09-25 01:55:14 UTC); merge commit `50fa3296037785762d6b44e42bd2cf248a366b8f`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, Swagger/OpenAPI, pruebas de flujos y CI en `develop`. |
| GB-007.06 | AuthUser [#15](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/15) | `feature/007-06/verify-local-contract` → `develop` | `MERGED` (2026-09-25 02:16:10 UTC); merge commit `8fe2f45ef81ba22e97e014bee42aa051ab08ff43`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, entregable integrado y CI en `develop`. |
| GB-008.01 | Game [#10](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/10) | `feature/008-01/save-delete-favorite` → `develop` | `MERGED` (2026-09-25 01:25:47 UTC); merge commit `8331c40a9da110f96ac07eff23f855efdd632d51`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, guardado/eliminación de favoritos propios y CI en `develop`. |
| GB-008.02 | Game [#11](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/11) | `feature/008-02/list-filter-paginate` → `develop` | `MERGED` (2026-09-25 01:55:10 UTC); merge commit `aadc7441957033847118a627909c283296daeceb`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, filtros/paginación de favoritos propios y CI en `develop`. |
| GB-008.03 | Game [#12](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/12) | `feature/008-03/suggest-favorites` → `develop` | `MERGED` (2026-09-25 02:15:55 UTC); merge commit `7c9dbfd74382a4e02593b0e7cab71d4da786acfd`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, endpoint de sugerencias, aislamiento y CI en `develop`. |
| GB-008.04 | Game [#13](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/13) | `feature/008-04/sync-snapshot` → `develop` | `MERGED` (2026-09-25 02:34:52 UTC); merge commit `717980249348347b62cec2e3c075210a80c8c77b`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, sincronización parcial/atómica de instantánea, pruebas y CI en `develop`. |
| GB-008.05 | Game [#14](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/14) | `feature/008-05/integrate-jwt-revocation` → `develop` | `MERGED` (2026-09-25 02:49:16 UTC); merge commit `4a03ca6d6600c17318659ec64f6f0cbbbcb4c40f`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, frontera JWT/AuthUser, revocación, fallo cerrado y CI en `develop`. |
| GB-008.06 | Game [#15](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/15) | `feature/008-06/swagger-complete-tests` → `develop` | `MERGED` (2026-09-25 03:13:14 UTC); merge commit `9b95692133de749cb6dfe40cd2c589be151ae974`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, Swagger/OpenAPI, pruebas autenticadas/aislamiento y CI en `develop`. |
| GB-008.07 | Game [#16](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/16) | `feature/008-07/verify-local-contract` → `develop` | `MERGED` (2026-09-25 03:33:42 UTC); merge commit `e253ccea9c6927dbf9ba446bd655d868f456f701`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, Swagger/OpenAPI, URLs locales, contrato y CI en `develop`. |
| GB-009.01 | Frontend [#8](https://github.com/CarlosSV923/GameBook.Frontend/pull/8) | `feature/009-01/igdb-server-integration` → `develop` | `MERGED` (2026-09-25 02:18:35 UTC); merge commit `70cf0f19d39d65d958f1cd374b212a3a34d8f348`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, proxy server-only, mocks, smoke test real local y CI en `develop`. |
| GB-009.02 | Frontend [#9](https://github.com/CarlosSV923/GameBook.Frontend/pull/9) | `feature/009-02/catalog-cards` → `develop` | `MERGED` (2026-09-25 03:14:40 UTC); merge commit `5c74ff7d73f6c2119d3860f1edccd4b11d2fa02b`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, listado/tarjetas, estados seguros, pruebas y CI en `develop`. |
| GB-009.03 | Frontend [#10](https://github.com/CarlosSV923/GameBook.Frontend/pull/10) | `feature/009-03/catalog-filters` → `develop` | `MERGED` (2026-09-25 03:33:38 UTC); merge commit `0fc23cf5fcd14eb7fcfc98a931be9f071b7b37b8`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, filtros/sugerencias, cancelación, validaciones y CI en `develop`. |
| GB-009.04 | Frontend [#11](https://github.com/CarlosSV923/GameBook.Frontend/pull/11) | `feature/009-04/infinite-scroll` → `develop` | `MERGED` (2026-09-25 03:43:00 UTC); merge commit `c9c6b87f0eb5c57499b013bd734e1d6156abefdb`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, paginación incremental, deduplicación, estados y CI en `develop`. |
| GB-009.05 | Frontend [#12](https://github.com/CarlosSV923/GameBook.Frontend/pull/12) | `feature/009-05/game-detail-modal` → `develop` | `MERGED` (2026-09-25 03:57:29 UTC); merge commit `8bc7c05b413a9e1b3ed846ba5e9746acd8bb3657`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, detalle IGDB, accesibilidad del modal, pruebas y CI en `develop`. |
| GB-009.06 | Frontend [#13](https://github.com/CarlosSV923/GameBook.Frontend/pull/13) | `feature/009-06/external-states` → `develop` | `MERGED` (2026-09-25 04:12:02 UTC); merge commit `1ac594db874802544dcc259fd22da2333b98f9ad`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, atribución IGDB, estados externos, fallbacks y CI en `develop`. |
| GB-009.07 | Frontend [#14](https://github.com/CarlosSV923/GameBook.Frontend/pull/14) | `feature/009-07/catalog-review` → `develop` | `MERGED` (2026-09-25 04:20:40 UTC); merge commit `01a4a115874bd35ac71bfc505c316c75475e79e8`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, revisión final, pruebas, revisión visual y CI en `develop`. |
| GB-010.01 | Frontend [#15](https://github.com/CarlosSV923/GameBook.Frontend/pull/15) | `feature/010-01/auth-forms` → `develop` | `MERGED` (2026-09-25 04:41:40 UTC); merge commit `38df05ae6583538a05fd89cdd36cd42dc272dbb4`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, formularios/validaciones, redirección sin sesión, pruebas y CI en `develop`. |
| GB-010.02 | Frontend [#16](https://github.com/CarlosSV923/GameBook.Frontend/pull/16) | `feature/010-02/auth-session` → `develop` | `MERGED` (2026-09-25 04:56:21 UTC); merge commit `5ba67031cef8e9aac4f7b12e1b65b1623114850d`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, sessionStorage/JWT, validación de sesión, clientes Bearer y CI en `develop`. |
| GB-010.03 | Frontend [#17](https://github.com/CarlosSV923/GameBook.Frontend/pull/17) | `feature/010-03/navigation-profile` → `develop` | `MERGED` (2026-09-25 05:08:53 UTC); merge commit `fe92d371ed82db8ab36220836ccb90b93e791f5e`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, navegación/perfil, cambio de contraseña, pruebas y CI en `develop`. |
| GB-010.04 | Frontend [#18](https://github.com/CarlosSV923/GameBook.Frontend/pull/18) | `feature/010-04/session-lifecycle` → `develop` | `MERGED` (2026-09-25 05:26:35 UTC); merge commit `b7b49bfcdc3ef1ee06d8fcd04b1247e50cff14a9`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, ciclo de sesión, revocación/logout, manejo `401`/`5xx` y CI en `develop`. |
| GB-010.05 | Frontend [#19](https://github.com/CarlosSV923/GameBook.Frontend/pull/19) | `feature/010-05/authuser-local-integration` → `develop` | `MERGED` (2026-09-25 05:37:54 UTC); merge commit `40b18e55f871997eeb6030e680ce1500a929c37e`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, CORS/puertos locales, flujo real AuthUser, revocación, pruebas y CI en `develop`. |
| GB-010.06 | Frontend [#20](https://github.com/CarlosSV923/GameBook.Frontend/pull/20) | `feature/010-06/account-review` → `develop` | `MERGED` (2026-09-25 05:44:31 UTC); merge commit `ec049a60d67147cfb47bee8a63946318a740cd95`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, revisión de cuenta, preferencias, mensajes EN/ES, responsive, pruebas y CI en `develop`. |
| GB-011.01 | Frontend [#21](https://github.com/CarlosSV923/GameBook.Frontend/pull/21) | `feature/011-01/favorite-actions` → `develop` | `MERGED` (2026-09-25 06:02:17 UTC); merge commit `5f2c6c3c0e2c32b9c5f88e4e3fcaf1ed2e3ed2b6`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, acciones visitante/autenticado, estados de guardado/eliminación, pruebas y CI en `develop`. |
| GB-011.02 | Frontend [#22](https://github.com/CarlosSV923/GameBook.Frontend/pull/22) | `feature/011-02/personal-favorites` → `develop` | `MERGED` (2026-09-25 06:54:08 UTC); merge commit `902782c1bf37d82be00effd68516e2dc56ee36ae`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, vista personal, filtros/sugerencias, paginación y CI en `develop`. |
| GB-011.03 | Frontend [#23](https://github.com/CarlosSV923/GameBook.Frontend/pull/23) | `feature/011-03/favorite-management` → `develop` | `MERGED` (2026-09-25 07:16:46 UTC); merge commit `7a62cbd29cf4fefbb13b0d25d6ba51d99493d6ba`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, confirmaciones, guardado/eliminación, estados y CI en `develop`. |
| GB-011.04 | Frontend [#24](https://github.com/CarlosSV923/GameBook.Frontend/pull/24) | `feature/011-04/detail-sync` → `develop` | `MERGED` (2026-09-25 17:23:58 UTC); merge commit `517e550073bebf1cc329165f16728e9b97fa3724`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, sincronización de snapshot, fallback ante fallo IGDB, pruebas y CI en `develop`. |
| GB-011.05 | Frontend [#25](https://github.com/CarlosSV923/GameBook.Frontend/pull/25) | `feature/011-05/game-local-integration` → `develop` | `MERGED` (2026-09-25 17:45:31 UTC); merge commit `a55a12d4991d818e9d864f9b71efbf0af9a366b8`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, documentación de URL/CORS, flujo local con Game, pruebas y CI en `develop`. |
| GB-011.05 | Game [#17](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/17) | `feature/011-05/game-local-integration` → `develop` | `MERGED` (2026-09-25 17:45:34 UTC); merge commit `fac28b4076065cb76727d2e6f5274f79f7dbc427`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, CORS exacto, flujo JWT/AuthUser, errores contractuales, pruebas y CI en `develop`. |
| GB-011.06 | Frontend [#26](https://github.com/CarlosSV923/GameBook.Frontend/pull/26) | `feature/011-06/favorites-review` → `develop` | `MERGED` (2026-09-25 18:11:35 UTC); merge commit `5368e59fba9c06771fd0423a0ce16eadc47217c5`; CI `repository-baseline` `SUCCESS` (dos ejecuciones) | Autor avisó; agente verificó merge, rama de destino, pruebas de estados visitante/autenticado, aislamiento, revisión responsive y CI en `develop`. |
| GB-012.10 | Frontend [#27](https://github.com/CarlosSV923/GameBook.Frontend/pull/27) | `feature/012-10/compose-local-integration` → `develop` | `MERGED` (2026-09-25 19:01:17 UTC); merge commit `1623eaecb1f0317ca5de7a823fb529d69621f023`; CI `repository-baseline` `SUCCESS` (runs `36174122135`, `36174161006`) | Autor avisó; agente verificó merge, rama de destino, entregable integrado en `origin/develop` y CI. |
| GB-012.10 | AuthUser [#16](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/16) | `feature/012-10/compose-local-integration` → `develop` | `MERGED` (2026-09-25 19:01:24 UTC); merge commit `2e3b6d5b1d8121cf7a86bae46841f0949130f408`; CI `repository-baseline` `SUCCESS` (runs `36174117128`, `36174167824`) | Autor avisó; agente verificó merge, rama de destino, entregable integrado en `origin/develop` y CI. |
| GB-012.10 | Game [#18](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/18) | `feature/012-10/compose-local-integration` → `develop` | `MERGED` (2026-09-25 19:01:21 UTC); merge commit `fa7e700fa5c7b3ef97eb6fd1a33ab6716e7d6413`; CI `repository-baseline` `SUCCESS` (runs `36174118531`, `36174165660`) | Autor avisó; agente verificó merge, rama de destino, entregable integrado en `origin/develop` y CI. |
| GB-012.03 | Frontend [#28](https://github.com/CarlosSV923/GameBook.Frontend/pull/28) | `feature/012-03/bilingual-readmes` → `develop` | `MERGED` (2026-09-25 20:18:04 UTC); merge commit `fae380bf95becc56f2d4b11aef7dd9f87945d27c`; CI `repository-baseline` `SUCCESS` (runs `36184570495`, `36184573990`) | Autor avisó; agente verificó merge, rama destino, README bilingüe, `.env.example` y ausencia de Compose/Docker versionado en `develop`. |
| GB-012.03 | AuthUser [#17](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/17) | `feature/012-03/bilingual-readmes` → `develop` | `MERGED` (2026-09-25 20:18:13 UTC); merge commit `a633aacf8996002c6c522814d2cfe8da1aa31ead`; CI `repository-baseline` `SUCCESS` (runs `36184574689`, `36184578671`) | Autor avisó; agente verificó merge, rama destino, README bilingüe, `.env.example` y ausencia de Compose/Docker versionado en `develop`. |
| GB-012.03 | Game [#19](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/19) | `feature/012-03/bilingual-readmes` → `develop` | `MERGED` (2026-09-25 20:18:08 UTC); merge commit `ecee47b40d337ed29da81e3a92e828feb6dbf629`; CI `repository-baseline` `SUCCESS` (runs `36184578066`, `36184582539`) | Autor avisó; agente verificó merge, rama destino, README bilingüe, `.env.example` y ausencia de Compose/Docker versionado en `develop`. |
| GB-012.04 | Frontend [#29](https://github.com/CarlosSV923/GameBook.Frontend/pull/29) | `feature/012-04/release-please-vercel-policy` → `develop` | `MERGED` (2026-09-25 20:41:08 UTC); merge commit `16631ecdedae09e5c614a0de74be8aab5d744aaa`; CI `repository-baseline` `SUCCESS` (runs `36186331836`, `36186368578`) | Autor avisó; agente verificó merge, rama de destino, commit, CI y configuración integrada en `develop` el 2026-09-25 UTC. |
| GB-012.04 | AuthUser [#18](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/18) | `feature/012-04/release-please-vercel-policy` → `develop` | `MERGED` (2026-09-25 20:40:20 UTC); merge commit `49726539f105a93b38f4d2a27666a59117d80099`; CI `repository-baseline` `SUCCESS` (runs `36186336551`, `36186373561`) | Autor avisó; agente verificó merge, rama de destino, commit, CI y configuración integrada en `develop` el 2026-09-25 UTC. |
| GB-012.04 | Game [#20](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/20) | `feature/012-04/release-please-vercel-policy` → `develop` | `MERGED` (2026-09-25 20:40:56 UTC); merge commit `043f178503cd9dc9698d6f6e4623cce12a22a7dd`; CI `repository-baseline` `SUCCESS` (runs `36186340736`, `36186376634`) | Autor avisó; agente verificó merge, rama de destino, commit, CI y configuración integrada en `develop` el 2026-09-25 UTC. |
| GB-012.06 | Game [#21](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/21) | `fix/012-06/render-deployment` → `develop` | `MERGED` (2026-09-26 04:02:35 UTC); merge commit `0cb2960e4ed8b5b2701e6bf68219038c75c5ffcc`; CI `repository-baseline` `SUCCESS` (run `36216574891`) | Autor avisó; agente verificó merge hacia `develop`, ausencia de `vercel.json` y CI. |
| GB-012.06 | Game [#22](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/22) | `develop` → `main` | `MERGED` (2026-09-26 04:04:09 UTC); merge commit `fd510e359cdd2f776e16999d12a0dc54eae0577f`; CI `repository-baseline` `SUCCESS` (run `36216722613`) | Autor avisó; agente verificó merge hacia `main`, commit y CI. |
| GB-012.06 | Game [#23](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/23) | `release-please--branches--main--components--gamebook-microservice-game` → `main` | `MERGED` (2026-09-26 04:07:31 UTC); merge commit `9c061090f9b886bce9aa1f90858404cf74fe7f1d`; CI `repository-baseline` `SUCCESS` (run `36216828712`) | Release-please `0.1.0` fusionado y verificado; la migración posterior falló en la recuperación previa por ausencia de `public._prisma_migrations`. |
| GB-012.06 | Game [#24](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/24) | `fix/012-06/recover-empty-migration-history` → `develop` | `MERGED` (2026-09-26 04:19:14 UTC); merge commit `768ff6e314b5b62ecb28142a1e76da51f8a3eebd`; CI `repository-baseline` `SUCCESS` (run `36217443459`) | Autor avisó; agente verificó merge hacia `develop`, CI y migración `develop` exitosa (run `36217473171`). |
| GB-012.06 | Game [#26](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/26) | `develop` → `main` | `MERGED` (2026-09-26 04:20:52 UTC); merge commit `a5b5e50833ec7b5426463788392d300ff2d1855f`; CI `repository-baseline` `SUCCESS` (run `36217531342`); migración `develop` `SUCCESS` (run `36217473171`) | Autor avisó; agente verificó merge hacia `main`. |
| GB-012.06 | Game [#27](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/27) | `release-please--branches--main--components--gamebook-microservice-game` → `main` | `MERGED` (2026-09-26 UTC); release `0.1.1`, commit `469f7ee`; CI `repository-baseline` `SUCCESS` | Release-please fusionado y verificado. |
| GB-012.06 | Game [#28](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/28) | `fix/012-06/prisma-generate-build` → `develop` | `MERGED` (2026-09-26 04:27:56 UTC); merge commit `f54572d78cfdcdce24458fc2a99241814463eb45`; CI `repository-baseline` `SUCCESS` (run `36217858458`) | Autor avisó; agente verificó merge hacia `develop`, build local y configuración Prisma. |
| GB-012.06 | Game [#29](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/29) | `develop` → `main` | `MERGED` (2026-09-26 04:31:33 UTC); merge commit `30d0c65ca9e2de97d6da385fc40625fa42b3eed7`; CI `repository-baseline` y Release Please `SUCCESS` (runs `36218091679`, `36218091731`); migración `develop` `SUCCESS` (run `36217906971`) | Autor avisó; agente verificó merge hacia `main`. |
| GB-012.06 | Game [#30](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/30) | `release-please--branches--main--components--gamebook-microservice-game` → `main` | `MERGED` (2026-09-26 04:32:42 UTC); release `0.1.2`, merge commit `57a07f7a39b0a91a5e0ed62129428edf8145bb33`; CI y Release Please `SUCCESS` (runs `36218148948`, `36218148965`) | Release-please fusionado y verificado. |
| GB-012.07 | Frontend [#30](https://github.com/CarlosSV923/GameBook.Frontend/pull/30) | `develop` → `main` | `MERGED` (2026-09-26 04:56:16 UTC); merge commit `80a9e365082f61b0d91b877235c86f0786849014`; CI `repository-baseline` `SUCCESS` (run `36219234840`) | Autor avisó; agente verificó merge hacia `main`, commit y CI. |
| GB-012.07 | Frontend [#31](https://github.com/CarlosSV923/GameBook.Frontend/pull/31) | `release-please--branches--main--components--gamebook-frontend` → `main` | `MERGED` (2026-09-26 05:01:38 UTC); release `0.2.0`, merge commit `b44a527d36230c937b08373177aa4d2cf516007a`; CI y Release Please `SUCCESS` (runs `36219569308`, `36219569327`) | Release-please fusionado y verificado; Vercel Production y CORS se validaron después. |
| GB-012.05 | AuthUser [#19](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/19) | `develop` → `main` | `MERGED` (2026-09-26 01:01:04 UTC); merge commit `fbc6b496ad2dd40d15908f9da3fafb35a15b647e`; CI `repository-baseline` `SUCCESS` (run `36206826206`) | Autor avisó; agente verificó merge hacia `main`, commit y CI. |
| GB-012.05 | AuthUser [#20](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/20) | `release-please--branches--main--components--gamebook-microservice-authuser` → `main` | `MERGED` (2026-09-26 01:02:28 UTC); merge commit `190adbe9cf306036c9f60a9b15f28468690ad68a`; CI `repository-baseline` `SUCCESS` (run `36206985185`) | Release-please fusionado y verificado; el primer intento del Action falló por `public` y el reintento con `schema=auth` falló por `CREATE SCHEMA`/permisos sobre la base `neondb` en el run `36207150045`. |
| GB-012.05 | AuthUser [#21](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/21) | `fix/012-05/remove-auth-schema-create` → `develop` | `MERGED` (2026-09-26 01:25:20 UTC); merge commit `c5a8b15a13e501fe1f5c942a867e9e11e3509eb6`; CI `repository-baseline` `SUCCESS` (run `36208314268`) | Autor avisó; agente verificó merge hacia `develop`, commit y CI. |
| GB-012.05 | AuthUser [#22](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/22) | `develop` → `main` | `MERGED` (2026-09-26 01:27:06 UTC); merge commit `2b950255201847bebbf6de8a6afc5dc3bc6e75d0`; CI `repository-baseline` `SUCCESS` (run `36208417351`); migración `develop` `SUCCESS` (run `36208364226`) | Autor avisó; agente verificó merge hacia `main`, commit y checks. |
| GB-012.05 | AuthUser [#23](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/23) | `release-please--branches--main--components--gamebook-microservice-authuser` → `main` | `MERGED` (2026-09-26 01:27:53 UTC); merge commit `f0a16b00d91cc69b57300d629f955ce159220f2c`; CI `repository-baseline` `SUCCESS` | Release-please `0.1.1` fusionado y verificado; el Action de migración posterior falló con `P3009` en el run `36208582573` por la fila fallida pendiente. |
| GB-012.05 | AuthUser [#24](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/24) | `fix/012-05/generate-prisma-before-build` → `develop` | `MERGED` (2026-09-26 01:49:54 UTC); merge commit `fb1bed7ccc5856f2e3652cb88c381f86d89a4e6c`; CI `repository-baseline` `SUCCESS` (run `36209660959`) | Autor avisó; agente verificó merge hacia `develop`, commit y CI. |
| GB-012.05 | AuthUser [#25](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/25) | `develop` → `main` | `MERGED` (2026-09-26 01:52:51 UTC); merge commit `b1555661bae02640fce1522868424df51d7faaab`; CI `repository-baseline` `SUCCESS` (run `36209811241`) | Autor avisó; agente verificó merge hacia `main`, commit y CI. Vercel creó Production/Preview `READY`, pero los logs de Production registran HTTP 500; cierre pendiente. |
| GB-012.05 | AuthUser [#27](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/27) | `fix/012-05/render-deployment` → `develop` | `MERGED` (2026-09-26 03:35:55 UTC); merge commit `482c6718ca4093683652545980f0876bce051035`; CI `repository-baseline` `SUCCESS` (run `36215188266`) | Autor avisó; agente verificó merge hacia `develop`, ausencia de `vercel.json` y CI. |
| GB-012.05 | AuthUser [#28](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/28) | `develop` → `main` | `MERGED` (2026-09-26 03:37:45 UTC); merge commit `d7f750cfa78f997c6cc23b8503f0f8278b54c0a5`; CI `repository-baseline` `SUCCESS` (runs `36215363698`, `36215300526`) | Autor avisó; agente verificó merge hacia `main`. Release-please run `36215394385` terminó `SUCCESS` sin crear PR por ausencia de commits user-facing desde `v0.1.2`; Render quedó validado y `GB-012.05` se cerró `[ RESOLVED ]`. |
| GB-012.09 | Frontend [#32](https://github.com/CarlosSV923/GameBook.Frontend/pull/32) | `feature/012-09/release-readme-evidence` → `develop` | `MERGED` (2026-09-26 20:14:23 UTC); merge commit `fd369da158707efebd7be7326b28d94bad5cf5d0`; `repository-baseline` `SUCCESS` (runs `36264867639`, `36264886716`, `36265521399`, `36265524002`, `36266096197`, `36266098828`, `36266438947`, `36266442613`, `36266929034`, `36266930818`, `36267252871`, `36267256603`, `36268440968`, `36268443051`) | Documentación de URLs productivas y ajustes de catálogo/preferencias/contraseña/composición/estado de búsqueda/completitud de juegos/estados de backend y reintentos seguros; merge verificado por GitHub. |
| GB-012.09 | Frontend [#33](https://github.com/CarlosSV923/GameBook.Frontend/pull/33) | `develop` → `main` | `MERGED` (2026-09-26 20:19:23 UTC); merge commit `3fa13bcfeed21551e252330f65309b44d3335c32`; `repository-baseline` `SUCCESS` (run `36268998712`) | Promoción de los cambios aprobados de GB-012.09 a `main`; merge verificado por GitHub. |
| GB-012.09 | AuthUser [#29](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/29) | `feature/012-09/release-readme-evidence` → `develop` | `MERGED` (2026-09-26 20:14:42 UTC); merge commit `e233ed2277c5f3eb438d3c9c978f46d306fd9109`; `repository-baseline` `SUCCESS` (runs `36264867732`, `36264888462`) | Documentación de URLs productivas en `README.md` y `README.es.md`; merge verificado por GitHub. |
| GB-012.09 | AuthUser [#30](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/30) | `develop` → `main` | `MERGED` (2026-09-26 20:19:37 UTC); merge commit `a2c62060778af962fb96a74f11ecdf27fd7a9f4d`; `repository-baseline` `SUCCESS` (run `36268999728`) | Promoción de los cambios aprobados de GB-012.09 a `main`; merge verificado por GitHub. |
| GB-012.09 | Game [#31](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/31) | `feature/012-09/release-readme-evidence` → `develop` | `MERGED` (2026-09-26 20:14:34 UTC); merge commit `4625ef051a6e576409a58760a6817afec673ce83`; `repository-baseline` `SUCCESS` (runs `36264867769`, `36264888516`) | URL productiva de Game y dependencias productivas documentadas en `README.md` y `README.es.md`; merge verificado por GitHub. |
| GB-012.09 | Game [#32](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/32) | `develop` → `main` | `MERGED` (2026-09-26 20:19:30 UTC); merge commit `8374c901fd8480e4537e8d63b80da6f4eae9fc93`; `repository-baseline` `SUCCESS` (run `36269002235`) | Promoción de los cambios aprobados de GB-012.09 a `main`; merge verificado por GitHub. |
| GB-012.12 | AuthUser [#31](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/31) | `feature/012-12/account-deactivation` → `develop` | `MERGED` (2026-09-26 20:51:21 UTC); merge `00a4072`; `repository-baseline` `SUCCESS`; migración recuperada y `SUCCESS` (run `36270952454`) | Autor fusionó; se corrigió la propiedad SQL autorizada y GitHub confirmó aplicación e idempotencia de la migración. |
| GB-012.12 | Game [#33](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/33) | `feature/012-12/account-deactivation` → `develop` | `MERGED` (2026-09-26 20:51:14 UTC); merge `d408e62`; `repository-baseline` `SUCCESS` (runs `36270793611`, `36270811947`) | Autor fusionó; merge, destino y checks verificados. |
| GB-012.12 | Frontend [#35](https://github.com/CarlosSV923/GameBook.Frontend/pull/35) | `feature/012-12/account-deactivation` → `develop` | `MERGED` (2026-09-26 20:51:29 UTC); merge `ca937b7`; `repository-baseline` `SUCCESS` (runs `36270793369`, `36270815326`) | Autor fusionó; merge, destino y checks verificados. |
| GB-012.12 | AuthUser [#32](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/32) | `develop` → `main` | `OPEN`; `repository-baseline` `IN_PROGRESS` (run `36271647189`); migración `SUCCESS` (run `36270952454`) | PR de promoción creado; pendiente de merge del autor. |
| GB-012.12 | Game [#34](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/34) | `develop` → `main` | `OPEN`; `repository-baseline` `IN_PROGRESS` (run `36271646081`) | PR de promoción creado; pendiente de merge del autor. |
| GB-012.12 | Frontend [#36](https://github.com/CarlosSV923/GameBook.Frontend/pull/36) | `develop` → `main` | `OPEN`; `repository-baseline` `IN_PROGRESS` (run `36271648078`) | PR de promoción creado; pendiente de merge del autor. |
| GB-012.13 | Frontend [#38](https://github.com/CarlosSV923/GameBook.Frontend/pull/38) | `feature/012-13/fix-filter-suggestion-selection` → `develop` | `MERGED` (2026-09-27 18:31:05 UTC); merge commit `fd3083876cfd719886fa08cf516bd7e92a21f754`; `repository-baseline` `SUCCESS` (runs `36272716089`, `36272718674`) | Autor avisó; agente verificó merge, rama de destino, commits `3b57bcb`/`2ed3ba9`, entregable y CI. |
| GB-012.13 | Frontend [#39](https://github.com/CarlosSV923/GameBook.Frontend/pull/39) | `develop` → `main` | `MERGED` (2026-09-27 18:36:15 UTC); merge commit `2cd4b0badb89265d3ebfdf1ca039acccb9ba6731`; `repository-baseline` `SUCCESS` (runs `36340943392`, `36341011899`) | Autor avisó; agente verificó merge, rama de destino y CI. |
| GB-012.13 | Frontend [#40](https://github.com/CarlosSV923/GameBook.Frontend/pull/40) | `release-please--branches--main--components--gamebook-frontend` → `main` | `MERGED` (2026-09-27 18:39:40 UTC); merge commit `6de4f55aa439440c3fc565dce6c901fe1d426a61`; `repository-baseline` `SUCCESS` (run `36341277452`) | Release `gamebook-frontend-v0.4.1`; agente verificó merge, rama de destino, tag y CI. |
| GB-012.14 | Frontend [#41](https://github.com/CarlosSV923/GameBook.Frontend/pull/41) | `feature/012-14/fix-password-change-production` → `develop` | `MERGED` (2026-09-27 19:11:47 UTC); merge commit `9eb72a032320da81c04c6af93e537d34999ca532`; `repository-baseline` `SUCCESS` (runs `36343200701`, `36343203117`) | Autor fusionó; agente verificó merge, destino, CI y presencia del modal en `origin/develop`. |
| GB-012.14 | Frontend [#42](https://github.com/CarlosSV923/GameBook.Frontend/pull/42) | `develop` → `main` | `MERGED` (2026-09-27 19:23:12 UTC); merge commit `ebc94398c960ba579c13a1c2dd603fedbbb82f9a`; `repository-baseline` `SUCCESS` (runs `36343462285`, `36343502210`) | Autor avisó; agente verificó merge, rama de destino, checks, `origin/main` `d77c5d9` y el entregable final. |
| GB-012.15 | AuthUser [#34](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/34) | `feature/012-15/render-healthcheck` → `develop` | `MERGED` (2026-09-27 19:48:03 UTC); merge `33a7c6438478ae542c9f8fc36afbf2cad3ff0b49`; `repository-baseline` `SUCCESS` (runs `36345331558`, `36345360366`) | Agente verificó merge, rama de destino y checks. |
| GB-012.15 | Game [#36](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/36) | `feature/012-15/render-healthcheck` → `develop` | `MERGED` (2026-09-27 19:47:59 UTC); merge `c0ce96a7be59c5cec37607754eae3552f77c8ce7`; `repository-baseline` `SUCCESS` (runs `36345333580`, `36345359490`) | Agente verificó merge, rama de destino y checks. |
| GB-012.15 | Frontend [#44](https://github.com/CarlosSV923/GameBook.Frontend/pull/44) | `feature/012-15/render-healthcheck` → `develop` | `MERGED` (2026-09-27 19:47:55 UTC); merge `c37976345612e4516c76c481c7ebec43762d081f`; `repository-baseline` `SUCCESS` (runs `36345334769`, `36345359090`) | Agente verificó merge, rama de destino y checks. |
| GB-012.15 | AuthUser [#35](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/35) | `develop` → `main` | `MERGED` (2026-09-27 19:52:10 UTC); merge `637c50574b754f7d7a514ef144c686ab3c7a1836`; `repository-baseline` `SUCCESS` (runs `36345678678`, `36345768269`) | Agente verificó merge, rama de destino, checks y contenido de `origin/main`. |
| GB-012.15 | Game [#37](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/37) | `develop` → `main` | `MERGED` (2026-09-27 19:52:02 UTC); merge `095cacd14a824a8de2a5c464ed290d9409c9c947`; `repository-baseline` `SUCCESS` (runs `36345674820`, `36345738862`) | Agente verificó merge, rama de destino, checks y contenido de `origin/main`. |
| GB-012.15 | Frontend [#45](https://github.com/CarlosSV923/GameBook.Frontend/pull/45) | `develop` → `main` | `MERGED` (2026-09-27 19:51:56 UTC); merge `ca7ae7efbb81732aed308acdaff36a0101689c12`; `repository-baseline` `SUCCESS` (runs `36345670547`, `36345770677`) | Agente verificó merge, rama de destino, checks y contenido de `origin/main`. |
| GB-012.16 | Frontend [#47](https://github.com/CarlosSV923/GameBook.Frontend/pull/47) | `feature/012-16/axios-rxjs-http-transport` → `develop` | `MERGED` (2026-09-27 20:23:35 UTC); merge `b6091179b47d2ece99dfa26a34e5bfec053e8712`; `repository-baseline` `SUCCESS` (runs `36347740347`, `36347757705`) | Agente verificó merge, rama de destino y checks. |
| GB-012.16 | Frontend [#48](https://github.com/CarlosSV923/GameBook.Frontend/pull/48) | `develop` → `main` | `MERGED` (2026-09-27 20:27:19 UTC); merge `f9735f1d4b4b6fe0dd983e63538af5da9b2b6eb4`; `repository-baseline` `SUCCESS` (runs `36347859235`, `36347955065`) | Autor avisó; agente verificó merge, rama de destino, checks y contenido de `origin/main`. |
| GB-012.17 | Frontend [#50](https://github.com/CarlosSV923/GameBook.Frontend/pull/50) | `feature/012-17/catalog-favorite-state` → `develop` | `MERGED` (2026-09-27 UTC); merge `ed28a7aa34aa9e6a14c9489b6cfd10ab0f953289`; `repository-baseline` `SUCCESS` (runs `36351191690`, `36351222659`) | Agente verificó merge, rama de destino y checks. |
| GB-012.17 | Frontend [#51](https://github.com/CarlosSV923/GameBook.Frontend/pull/51) | `develop` → `main` | `MERGED` (2026-09-27 21:22:20 UTC); merge `7a633bcb052fbb8e54da47e176a892748952b360`; `repository-baseline` `SUCCESS` (runs `36351428872`, `36351366363`) | Autor avisó; agente verificó merge, rama de destino, checks y contenido de `main` (`b5c66f646141a9b360a958f7bd3faa6a6dfb7bbe`). |
| GB-012.18 | Frontend [#53](https://github.com/CarlosSV923/GameBook.Frontend/pull/53) | `feature/012-18/rxjs-favorite-pagination` → `develop` | `MERGED` (2026-09-28 01:45:11 UTC); merge `fb206863365246b4eca071615de844b8da1c7fdc`; `repository-baseline` `SUCCESS` (runs `36366144107`, `36366127805`) | Agente verificó merge, rama de destino y checks. |
| GB-012.18 | Frontend [#54](https://github.com/CarlosSV923/GameBook.Frontend/pull/54) | `develop` → `main` | `MERGED` (2026-09-28 01:47:33 UTC); merge `f25c1ed74672d0732f654ee995d82b1f7ffa22ec`; `repository-baseline` `SUCCESS` (runs `36367183709`, `36367237803`) | Autor avisó; agente verificó merge, rama de destino, checks y contenido de `main` (`f25c1ed74672d0732f654ee995d82b1f7ffa22ec`). |
| GB-012.19 | AuthUser [#39](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/39) | `feature/012-19/authuser-ts-import-extensions` → `develop` | `MERGED` (2026-09-29 01:10:51 UTC); merge `6032519541afa8413b8c85cebeef0a919a9633d7`; ambos checks `repository-baseline` `SUCCESS` (runs `36506417500`, `36506447731`) | Agente verificó merge, rama de destino y checks. |
| GB-012.19 | AuthUser [#40](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/40) | `develop` → `main` | `MERGED` (2026-09-29 01:13:40 UTC); merge `fb522e0b7172d9afa60409a8fa638b22c3a56983`; ambos checks `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, checks y contenido de `main`. |
| GB-012.20 | Game [#41](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/41) | `feature/012-20/game-ts-import-extensions` → `develop` | `MERGED` (2026-09-29 01:21:50 UTC); merge `c5a666e91d90b134766002be7d7e7a01543d0e07`; ambos checks `repository-baseline` `SUCCESS` (runs `36507370178`, `36507387929`) | Agente verificó merge, rama de destino y checks. |
| GB-012.20 | Game [#42](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/42) | `develop` → `main` | `MERGED` (2026-09-29 01:25:07 UTC); merge `b825daa6629ae83b9a2d1c801e7c0e086e59dfea`; ambos checks `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, checks y contenido de `main`. |
| GB-012.21 | Frontend [#59](https://github.com/CarlosSV923/GameBook.Frontend/pull/59) | `feature/012-21/render-warmup-alert` → `develop` | `MERGED` (2026-09-29 02:08:14 UTC); merge `6eb0ff56fbf338258fae10e7f5f41a24c93d0dd3`; ambos checks `repository-baseline` `SUCCESS` | Agente verificó merge, rama de destino y checks. |
| GB-012.21 | Frontend [#60](https://github.com/CarlosSV923/GameBook.Frontend/pull/60) | `develop` → `main` | `MERGED` (2026-09-29 02:09:48 UTC); merge `cbfedb9d7ac94c79df7d9be190d8ae6ee5004ce3`; ambos checks `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, checks y contenido final de `main` (`38e9a82cd9f73725108f066925f9d83af6581c61`). |
| GB-013.07 | Frontend [#55](https://github.com/CarlosSV923/GameBook.Frontend/pull/55) | `feature/013-07/frontend-architecture` → `develop` | `MERGED` (2026-09-28 02:32:02 UTC); merge `6634ddbdcf5de325a5535f4f59a8cc89cb14b492`; ambos checks `repository-baseline` `SUCCESS` | Agente verificó merge, rama de destino y checks. |
| GB-013.07 | Frontend [#57](https://github.com/CarlosSV923/GameBook.Frontend/pull/57) | `feature/013-07/frontend-architecture` → `develop` | `MERGED` (2026-09-28 02:43:43 UTC); merge `bd7bf1e0f03bcc16cfddea6a98a3c67d46418397`; ambos checks `repository-baseline` `SUCCESS` | Agente verificó merge, rama de destino y checks del ajuste de README. |
| GB-013.07 | Frontend [#56](https://github.com/CarlosSV923/GameBook.Frontend/pull/56) | `develop` → `main` | `MERGED` (2026-09-28 02:34:46 UTC); merge `f3d0d5b344a5b1693f88d0bcc30df41e83676826`; ambos checks `repository-baseline` `SUCCESS` | Agente verificó merge, rama de destino y checks de la promoción inicial. |
| GB-013.07 | Frontend [#58](https://github.com/CarlosSV923/GameBook.Frontend/pull/58) | `develop` → `main` | `MERGED` (2026-09-28 02:45:25 UTC); merge `7e9f4eb2862b5832c466f3b65048a814a75977ed`; ambos checks `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, checks, commit de `main` y HTTP 200 de las variantes EN/ES en Pages. |
| GB-013.08 | AuthUser [#37](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/37) | `feature/013-08/authuser-architecture` → `develop` | `MERGED` (2026-09-28 03:01:01 UTC); merge `f14014c6e34cf1a3075d065cfb445097e61749a5`; ambos checks `repository-baseline` `SUCCESS` | Agente verificó merge, rama de destino y checks. |
| GB-013.08 | AuthUser [#38](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/38) | `develop` → `main` | `MERGED` (2026-09-28 03:03:36 UTC); merge `12618e85e5404d05cc7a515621d58797f58bedfd`; ambos checks `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, checks, commit de `main` y HTTP 200 de las variantes EN/ES en Pages. |
| GB-013.09 | Game [#39](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/39) | `feature/013-09/game-architecture` → `develop` | `MERGED` (2026-09-28 03:16:06 UTC); merge `e1158b094111fa0816162b7dc6adfadb0c7a1c21`; ambos checks `repository-baseline` `SUCCESS` (runs `36373001193`, `36373019506`) | Agente verificó merge, rama de destino y checks. |
| GB-013.09 | Game [#40](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/40) | `develop` → `main` | `MERGED` (2026-09-28 03:19:42 UTC); merge `0ea4aed9960f75492b1e70811bc187a61bb3c116`; ambos checks `repository-baseline` `SUCCESS` | Autor avisó; agente verificó merge, rama de destino, checks, commit de `main` y HTTP 200 de las variantes EN/ES en Pages. |
