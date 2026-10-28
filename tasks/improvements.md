# GameBook — Tareas de la fase de mejoras

## 1. Reglas

Rigen sin cambios los estados (`[ NEW ]`, `[ ACTIVE ]`, `[ RESOLVED ]`), la **regla de bloqueo absoluta**, el registro obligatorio de cada PR y la verificación posterior al merge del autor definidos en [`mvp.md`](mvp.md) §1. La autoridad funcional es [`../specs/improvements.md`](../specs/improvements.md) y la estrategia, [`../plan/improvements.md`](../plan/improvements.md). El ejecutor previsto de cada subtarea es Codex (`gpt-6-luna`) bajo coordinación de Claude, según el plan §4.

En «Depende de», `GB-014` es el grupo cerrado del MVP en `mvp.md`.

## 2. Preparación

### [ RESOLVED ] GB-015 — Preparar la fase de mejoras

Ámbito: workspace de coordinación.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-015.01** — Aprobar especificación, plan y tareas | GB-014 | El autor aprueba `specs/improvements.md`, `plan/improvements.md` y este archivo, incluidos metadatos propuestos (spec §7.2). Registrar fecha y cambios pedidos. |
| [ RESOLVED ] **GB-015.02** — Actualizar Archify del workspace a 3.2 | GB-015.01 | `.agents/skills/archify` en la versión 3.2 de `tt-a1i/archify`, hash actualizado en `skills-lock.json`, `node bin/archify.mjs --help` y un `validate` de ejemplo funcionan. Sin cambios en repos de aplicación. |
| [ RESOLVED ] **GB-015.03** — Actualizar contexto global | GB-015.01 | `AGENTS.md` y `CLAUDE.md` enlazan la fase de mejoras como vigente; `tasks/mvp.md` muestra `GB-014` como `[ RESOLVED ]`. Diff revisado. |

#### GB-015.01 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-10-24). El autor aprobó en el chat de coordinación `specs/improvements.md`, `plan/improvements.md` y este archivo, sin cambios pedidos; los metadatos propuestos de spec §7.2 quedan aprobados como base.

#### GB-015.02 / GB-015.03 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-24). Dependencia `GB-015.01` verificada como `[ RESOLVED ]`. Ejecutor: Codex (`gpt-6-luna`) con coordinación de Claude, una ejecución por subtarea con archivos disjuntos: `.agents/skills/archify/` y `skills-lock.json` para GB-015.02; `AGENTS.md`, `CLAUDE.md` y `tasks/mvp.md` para GB-015.03. El workspace no es repositorio Git, por lo que no hay PR; la evidencia es el diff revisado por la coordinación.

#### GB-015.02 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-10-24).
- Primer intento con Codex (`gpt-6-luna`): sin cambios, porque el sandbox `workspace-write` de Codex denegó la escritura en `.agents/` ("Access denied"). Codex identificó el tag y el algoritmo de hash, pero su hash propuesto (`74bf0ffd…`) no se usó, porque no reproducía el valor antiguo del lock.
- Ejecución final por la coordinación (Claude) a partir del mismo origen: `tt-a1i/archify` tag `v3.2.0`, commit `208acba8d412335185b2056fedf54b3405df43fb`, carpeta `archify/` (203 archivos, finales LF). La copia 2.17.0-dev.1 anterior se movió a un respaldo fuera del workspace.
- `skills-lock.json`: `computedHash` de archify pasa a `e8cbde4db89de7b49cfab034923a9f6028afd0bc413ab26fe1a68c242e2005fe`, calculado con el algoritmo de `vercel-labs/skills` (`src/local-lock.ts`: SHA-256 de rutas relativas ordenadas más contenido). El mismo script reproduce exactamente el hash previo de la copia 2.17 (`6543c582…`), lo que valida el método. La entrada `interface-design` no cambió.
- Verificación: `SKILL.md` declara `version: "3.2"` y `skill-release.json` declara `3.2.0`; `node bin/archify.mjs --help` funciona; `validate architecture examples/brand-aware-delivery.architecture.json --quality showcase --json` termina `ok=true`, con 9 comprobaciones, 0 errores y 0 advertencias. No se instalaron dependencias.
- Impacto para `GB-020`: las fuentes 2.17 usan `schema_version: 1`, que 3.2 sigue admitiendo, pero la validación 3.2 es más estricta. `GameBook.Frontend-architecture-en.json` falla con `layout/boundary-encloses-non-member` (AuthUser dentro del marco «Server-only IGDB credentials»). Los diagramas deben regenerarse, no solo re-renderizarse, como ya prevé el plan §3.6.
- Nota operativa: las ejecuciones Codex no pueden escribir en `.agents/`; las tareas que solo leen el skill no se ven afectadas.

#### GB-015.03 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-10-24). Ejecutado por Codex (`gpt-6-luna`, `codex exec`); diff revisado por la coordinación contra copias previas.
- `AGENTS.md` y `CLAUDE.md` declaran el MVP completo y la fase de mejoras activa, resumen las decisiones globales de spec §2–§7, mantienen la regla de bloqueo apuntando a `tasks/mvp.md` y `tasks/improvements.md`, y actualizan las frases que describían `interface-design`, `archify` y `GameBook.System` como futuros. Codex verificó 18 enlaces relativos existentes en cada archivo; codificación UTF-8 y finales LF conservados.
- `tasks/mvp.md` cambió en una sola línea: la cabecera de `GB-014` pasa a `[ RESOLVED ]` tras comprobar sus 12 subtareas resueltas.
- Observación menor no bloqueante: el párrafo sobre `GameBook.System` menciona `docs/development*.md`, que pertenecen a los repositorios de aplicación; se precisará en `GB-020.10`.

## 3. Frontend

### [ RESOLVED ] GB-016 — Arquitectura del Frontend

Ámbito: `GameBook.Frontend`. Subtareas secuenciales; cada una con PR a `develop` y CI verde (`test`, `lint`, `typecheck`, `build` y, desde `.01`, `e2e`). Cierre del grupo: spec §2, §4 y §6 cumplidos sin pérdida de comportamiento.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-016.01** — Añadir E2E Playwright como red de seguridad | GB-015.01 | Playwright con proyectos desktop/mobile, red simulada (`page.route` + fixtures ficticias) y selectores por rol. Cubre los flujos de spec §6. Job `e2e` en CI. Sin cambios de código de producción salvo atributos accesibles imprescindibles. |
| [ RESOLVED ] **GB-016.02** — Dividir el adaptador IGDB y validar con esquema | GB-016.01 | `igdb-query-builder`, `igdb-mapper` (Zod) e `igdb-client` según plan §3.2. Pruebas unitarias reubicadas sin perder casos; rutas `/api/igdb/*` con respuestas idénticas. Secretos server-only. |
| [ RESOLVED ] **GB-016.03** — Aplicar umbral de valoraciones al ranking | GB-016.02 | `MIN_TOTAL_RATING_COUNT` aplicado solo a consultas ordenadas por `total_rating`; nombre y sugerencias sin umbral (spec §2). Valor elegido con evidencia del top real registrada en el PR. `contracts/igdb-mapping.md` actualizado; puntuación mostrada en 0–100; pruebas del builder y E2E. |
| [ RESOLVED ] **GB-016.04** — Reorganizar módulos por contexto | GB-016.03 | `src/features/<contexto>/{api,model,ui}` y `shared/` reducido (plan §3.2.4). Solo movimientos e imports, en commit propio; todas las pruebas sin cambios de aserciones. |
| [ RESOLVED ] **GB-016.05** — Migrar estado de servidor a TanStack Query | GB-016.04 | `QueryClientProvider`, colección infinita compartida para catálogo y favoritos, consultas de detalle/sugerencias y mutaciones con `retry: 0`. Se eliminan duplicación y `useState` de servidor. Healthchecks, Axios + RxJS, cancelación y reinicio de paginación preservados; unitarias y E2E en verde. |
| [ RESOLVED ] **GB-016.06** — Precargar el catálogo con Server Components | GB-016.05 | `app/page.tsx` como Server Component con primera página deshidratada en TanStack Query y fallback cliente ante error. El HTML inicial contiene resultados; `IGDB_CLIENT_SECRET` ausente del bundle cliente (comprobado en build); E2E en verde. |

### [ RESOLVED ] GB-017 — Rediseño visual dirección A

Ámbito: `GameBook.Frontend`; **skill obligatorio `interface-design`**. Cada PR adjunta capturas antes/después en claro/oscuro, EN/ES, 1440×900 y 375×812. Cierre del grupo: spec §3 y criterios 2–4.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-017.01** — Consolidar la dirección A en el sistema visual | GB-016 | `.interface-design/direction.md` y `system.md` describen la dirección A (tokens, tipografía, tira de índice, composición, móvil y estados) con referencia al canvas; revisión del autor. |
| [ RESOLVED ] **GB-017.02** — Base Tailwind 4, fuentes y primitivas | GB-017.01 | Tokens en `@theme` claro/oscuro, `next/font` (Newsreader, IBM Plex Sans/Mono) y primitivas `Button`, `IconButton`, `TextField`, `SegmentedControl` e `IndexStrip` con pruebas; vistas existentes sin regresiones. |
| [ RESOLVED ] **GB-017.03** — Cabecera, hero y barra de filtros | GB-017.02 | Cabecera de una fila, hero compacto (≤ ~1/3 del viewport a 1440×900), filtros en una fila fija en escritorio, y menú y panel de filtros en móvil; sugerencias por clic y teclado intactas. |
| [ RESOLVED ] **GB-017.04** — Ficha con tira de índice | GB-017.03 | Ficha con rango, puntuación 0–100, año y plataformas en catálogo y favoritos; grid en escritorio y ficha horizontal en móvil; acciones de guardar/eliminar conservadas; la corrección de §4.4/§4.5 de `mvp.md` corresponde a GB-017.08. |
| [ RESOLVED ] **GB-017.05** — Modal de detalle y confirmaciones | GB-017.04 | Scrim oscuro en ambos temas, ficha técnica, foco atrapado, cierre por teclado; confirmaciones y estado de detalle no disponible. |
| [ RESOLVED ] **GB-017.06** — Cuenta, perfil, estados y avisos | GB-017.05 | Login, registro, perfil (contraseña y desactivación), carga/vacío/error, avisos y alerta Render con la dirección A; acción primaria ≥ 4.5:1. |
| [ RESOLVED ] **GB-017.08** — Restaurar elemento contextual sin sesión y confirmación de guardado | GB-017.05 | Cumplir `specs/mvp.md` §4.4 y §4.5 en tarjeta y modal, con la dirección A. Sin sesión, la acción de favoritos no navega: muestra junto a ella un elemento contextual accesible con «Para guardar en favoritos debes:» y los enlaces «Iniciar sesión» y «Crear cuenta» (textos EN equivalentes, mismo mensaje en tarjeta y modal), cerrable con Escape o clic fuera y sin guardar nada. Con sesión, pulsar guardar abre una confirmación contextual y solo tras confirmar se llama a Game; éxito, duplicado y error se muestran como hoy. Eliminar mantiene su confirmación. Pruebas unitarias, y E2E actualizadas de «navega a login / guarda directo» al comportamiento especificado en desktop y mobile; EN/ES y claro/oscuro. |
| [ RESOLVED ] **GB-017.07** — Retirar CSS heredado, revisión final y capturas | GB-017.06, GB-017.08 | `globals.css` reducido a la base mínima, sin clases huérfanas; revisión de contraste y estados; estado «fin de resultados» sin la tira «Unavailable» superpuesta (regresión de GB-017.06); texto de desactivación del perfil sin la mención «for the MVP»/«para el MVP» (EN/ES, decisión del autor del 2026-10-25); `pnpm screenshots` genera `docs/screenshots/{en,es}` y un GIF desde fixtures E2E; promoción `develop → main` y release-please. |

#### GB-016.01 / GB-018.01 / GB-019.01 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-24). Dependencia `GB-015.01` verificada `[ RESOLVED ]`. Ejecución en paralelo con Codex (`gpt-6-luna`), una por repositorio. Ramas creadas por la coordinación desde `origin/develop`: Frontend `feature/016-01/playwright-e2e-safety-net` (`05ca4ef`), AuthUser `feature/018-01/user-aggregate-mutations` (`ce4b205`) y Game `feature/019-01/framework-free-use-cases` (`79763e5`). Codex implementa y verifica; la coordinación revisa el diff, hace commit, push y abre el PR.

- GB-016.01 (2026-10-24): el primer intento de Codex no pudo abrir shell (sandbox Windows con raíz extra `ms-playwright`). El segundo escribió la configuración, las fixtures, las pruebas y el job de CI, pero no pudo instalar dependencias. La coordinación instaló Chromium y ejecutó: `pnpm test:e2e` 3 pasan y 3 se omiten por diseño (desktop/mobile), `pnpm test` 86/86, lint, typecheck y build correctos. Ajustó una aserción: tras guardar, la tarjeta pasa a «Remove…», según GB-012.17. Commits `848dfbd` y `49331b4`; PR #67 abierto.
- **Hallazgo GB-016.01 (decidido el 2026-10-24: se corrige en GB-017.08):** el comportamiento productivo difiere de `specs/mvp.md`. Guardar sin sesión navega a `/login` en vez de mostrar el elemento contextual «Para guardar en favoritos debes: Iniciar sesión / Crear cuenta» (§4.4), y guardar con sesión no muestra la confirmación previa (§4.5). La suite E2E documenta el comportamiento actual.
- GB-018.01 (2026-10-24): Codex implementó el cambio, pero su sandbox no pudo arrancar Vitest (`spawn EPERM`). La coordinación ejecutó las suites localmente: `pnpm test` 59/59, `pnpm test:e2e` 32/32 (incluye el contrato OpenAPI), lint sin errores, `tsc` y `build` correctos. Además corrigió una prueba unitaria que comparaba contra el agregado ya mutado. Commits `680f867` y `6690ca1`; PR #46 abierto, pendiente de CI y merge del autor.
- GB-019.01 (2026-10-24): Codex implementó y verificó con workaround de Vitest. La coordinación repitió `pnpm test` 67/67, `pnpm test:e2e` 22/22, lint, typecheck y build, y aplicó Prettier a un import. El módulo Nest de casos de uso pasa a `src/application.module.ts` para que `application/` no importe NestJS (protegido por una prueba de límites). Commits `8e647bd` y `8210d59`; PR #48 abierto, pendiente de CI y merge del autor.

#### GB-016.01 / GB-018.01 / GB-019.01 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-10-24). El autor avisó del merge y la coordinación verificó en GitHub el estado `MERGED`, el destino `develop`, CI verde y el entregable en `origin/develop`.
- GB-016.01: [Frontend #67](https://github.com/CarlosSV923/GameBook.Frontend/pull/67) fusionado a las 20:04:03 UTC con merge `48d069b96fba8a6e224008f8c1cb37ac693b721e`; checks `repository-baseline` y `e2e` en `SUCCESS`. `playwright.config.ts` y `tests/e2e/` presentes en `develop`.
- GB-018.01: [AuthUser #46](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/46) fusionado a las 20:04:18 UTC con merge `adf5208472ac4dd74bfea6d122167fdb45dff6f5`; `repository-baseline` en `SUCCESS`. `replacePassword` ya no existe en `src/` de `develop`.
- GB-019.01: [Game #48](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/48) fusionado a las 20:04:42 UTC con merge `40efd54744bd05f14dbdc5d1076563002f8a9c33`; `repository-baseline` en `SUCCESS`. `src/application.module.ts` y la prueba de límites están presentes; `src/application` y `src/domain` no importan `@nestjs/*` en `develop`.
- Los grupos GB-016, GB-018 y GB-019 vuelven a `[ NEW ]` (estado derivado: progreso parcial sin subtareas activas). Quedan habilitadas GB-016.02, GB-018.02 y GB-019.02.

#### GB-016.02 / GB-018.02 / GB-019.02 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-24). Dependencias `GB-016.01`, `GB-018.01` y `GB-019.01` verificadas `[ RESOLVED ]`. Ejecución en paralelo con Codex (`gpt-6-luna`), una por repositorio, desde `origin/develop`: Frontend `feature/016-02/split-igdb-adapter` (`48d069b`), AuthUser `feature/018-02/bearer-session-guard` (`adf5208`) y Game `feature/019-02/favorite-aggregate-mutations` (`40efd54`).
- GB-016.02 (2026-10-24): Codex dividió el adaptador en `igdb-query-builder` (147 líneas), `igdb-mapper` con Zod 4.6.5 (214) e `igdb-client` (187), y registró `zod` con `--lockfile-only`. La coordinación instaló dependencias, confirmó que todos los casos previos de `igdb-client.test.ts` siguen presentes (6 casos nuevos), comparó el mapper con los lectores anteriores y ejecutó: `pnpm test` 92/92, `pnpm test:e2e` 3 pasan y 3 se omiten por diseño, lint, typecheck y build correctos. Commits `139e445` y `736153c`; PR #68 abierto.
- GB-018.02 (2026-10-24): Codex implementó `BearerSessionGuard` y `@CurrentSession()` y reportó que el hash del OpenAPI cambió sin explicarlo. La coordinación comparó un volcado ordenado antes y después. La única diferencia: `GET /v1/auth/session` y `DELETE /v1/users/me` dejan de listar un parámetro de cabecera `authorization`, generado antes por `@Headers()`; `PATCH /password` nunca lo tuvo. La seguridad `BearerAuth`, rutas, operationIds, cuerpos, respuestas y esquemas no cambian, y `contracts/authuser.openapi.yaml` no declara ese parámetro. La coordinación lo considera alineación con el contrato congelado y lo explica en el PR para revisión del autor. Suites: `pnpm test` 66/66, `pnpm test:e2e` 32/32, lint sin errores, `tsc` y build correctos. Commits `7d64b49` y `bb9c02b`; PR #47 abierto.
- GB-019.02 (2026-10-24): Codex implementó y verificó. La coordinación repitió las suites (`pnpm test` 66/66, `pnpm test:e2e` 22/22, lint, typecheck y build) y añadió una prueba de caso de uso para instantánea vacía sin escritura, porque se eliminaron las pruebas de repositorio de `updateSnapshot`. Commits `18edc30` y `5250cb6`; PR #49 abierto.

#### GB-016.02 / GB-018.02 / GB-019.02 — Evidencia de cierre

- Estado: `[ RESOLVED ]` (2026-10-24). Tras el aviso del autor, la coordinación verificó en GitHub `MERGED` hacia `develop`, CI verde y entregables en `origin/develop`.
- GB-016.02: [Frontend #68](https://github.com/CarlosSV923/GameBook.Frontend/pull/68) fusionado a las 20:19:32 UTC con merge `c64cf3c14ff74940b92b31225e40f63d3d8f1af5`; `repository-baseline` y `e2e` en `SUCCESS`. `igdb-query-builder.ts`, `igdb-mapper.ts` e `igdb-client.ts` presentes en `src/server/igdb/`.
- GB-018.02: [AuthUser #47](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/47) fusionado a las 20:19:46 UTC con merge `45b30c72d2da440d5a38d7e4ec5799a4fdadea8e`; `repository-baseline` en `SUCCESS`. `BEARER_PATTERN` solo existe en `bearer-session.guard.ts`. El autor fusionó el PR con la nota de OpenAPI (cabecera `authorization` explícita retirada, alineada con `contracts/authuser.openapi.yaml`).
- GB-019.02: [Game #49](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/49) fusionado a las 20:20:05 UTC con merge `0e4468227b29aa942178037b9cfe073be26fa97a`; `repository-baseline` en `SUCCESS`. `updateSnapshot` ya no existe en el puerto del repositorio ni en la infraestructura.
- Los grupos vuelven a `[ NEW ]` (estado derivado). Quedan habilitadas GB-016.03, GB-018.03 y GB-019.03.

#### GB-016.03 / GB-018.03 / GB-019.03 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-24). Dependencias `GB-016.02`, `GB-018.02` y `GB-019.02` verificadas `[ RESOLVED ]`. Ramas desde `origin/develop`: Frontend `feature/016-03/rating-count-threshold` (`c64cf3c`), AuthUser `feature/018-03/composition-root` (`45b30c7`) y Game `feature/019-03/favorite-read-model` (`0e44682`). Ejecución con Codex (`gpt-6-luna`), una por repositorio. Para GB-016.03, la coordinación mide el umbral contra IGDB con credenciales locales de desarrollo sin exponerlas, y Codex no lee secretos.
- Medición GB-016.03 (2026-10-24, coordinación): consultas `/games` y `/games/count` con credenciales de desarrollo de `.env` (valores no impresos) y el filtro de completitud del catálogo. Top 20 sin umbral: notas 100 con 0–10 valoraciones (p. ej. *Wortgewandt*). Con ≥ 50 aparecen casos con pocas valoraciones (*American Chopper*, 50). Con ≥ 100 el top es reconocible (*The Witcher 3 GOTY*, *Super Metroid*, *A Link to the Past*, *Elden Ring*…). Juegos restantes: 3.477 (≥ 50), 1.961 (≥ 100), 1.017 (≥ 200), 645 (≥ 300). Con filtros de nicho, ≥ 100 y ≥ 200 dejan respectivamente Sega Saturn 13 y 8, Game Boy Advance 32 y 14, año 1987 11 y 5, y año 2025 24 y 7. Valor elegido: **100**. `contracts/igdb-mapping.md` se enmendó en consecuencia.
- GB-016.03 (2026-10-24): Codex aplicó `MIN_TOTAL_RATING_COUNT = 100` solo a consultas sin nombre, cambió la puntuación a escala 0–100 con etiquetas EN/ES y añadió la nota traducida sobre el ranking, sin acceso a IGDB ni a `.env`. La coordinación ejecutó: `pnpm test` 93/93, `pnpm test:e2e` 3 pasan y 3 se omiten por diseño, lint, typecheck y build correctos. Corrigió una aserción E2E que buscaba la nota en español en la página de favoritos y la movió al catálogo público. Commits `87ecf14` y `6274425`; PR #69 abierto. Versión esperada: *minor* (`feat`).
- GB-018.03 (2026-10-24): Codex movió la composición a `src/application.module.ts` (mismo patrón que Game), introdujo `AuthTokenSettings`, limitó la sesión validada a campos públicos, eliminó el `try/catch` redundante y añadió la prueba de límites. La coordinación ejecutó: `pnpm test` 68/68, `pnpm test:e2e` 32/32, lint sin errores, `tsc` y build correctos. Commits `34bc513` y `f8953e4`; PR #48 abierto.
- GB-019.03 (2026-10-24): Codex creó `FavoriteReadModel` y `PrismaFavoriteReadModel`; el repositorio de dominio queda con `findByIdentity`/`create`/`save`/`delete`. La coordinación comprobó que `list`/`suggest` se movieron sin cambios y que siguen todos los casos de prueba anteriores, y ejecutó `pnpm test` 66/66, `pnpm test:e2e` 22/22, lint, typecheck y build. Commits `cb02651` y `8908c56`; PR #50 abierto.

#### GB-016.03 / GB-018.03 / GB-019.03 — Verificación de merges

- 2026-10-25: tras el aviso del autor, la coordinación verificó `MERGED` hacia `develop`, CI verde y los entregables en `origin/develop`.
- GB-016.03 → `[ RESOLVED ]`: [Frontend #69](https://github.com/CarlosSV923/GameBook.Frontend/pull/69), merge `6ce9b05835a597f45455c092c2cd46a63d5ed440` (03:05:19 UTC); `repository-baseline` y `e2e` en `SUCCESS`; `MIN_TOTAL_RATING_COUNT = 100` presente en `src/server/igdb/igdb-query-builder.ts`.
- GB-019.03 → `[ RESOLVED ]`: [Game #50](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/50), merge `07c6513727b6f1df2f189fb656455ced534679b5` (03:06:26 UTC); `repository-baseline` en `SUCCESS`; `favorite-read-model.ts` y `prisma-favorite-read-model.ts` presentes, sin `list`/`suggest` en el repositorio de dominio.
- GB-018.03 sigue `[ ACTIVE ]`: [AuthUser #48](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/48), merge `4144f923c0ba3e1c1ab8331cf23f27f7ad7b1558` (03:06:02 UTC), CI `SUCCESS`, `src/application.module.ts` y prueba de límites presentes. Falta la parte de promoción de su entregable: se abrió [AuthUser #49](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/49) `develop → main` (sin conflictos según `git merge-tree`). Tras su merge se espera el release PR de release-please.
- Observación: `origin/main` de AuthUser contiene commits de release (0.1.0–0.4.0) que nunca se sincronizaron de vuelta a `develop`, contra lo que pide `plan/mvp.md` §5.6. No bloquea la promoción; se propondrá sincronizar `main → develop` tras el release.

#### GB-016.04 / GB-019.04 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencias `GB-016.03` y `GB-019.03` verificadas `[ RESOLVED ]`. Ramas desde `origin/develop`: Frontend `feature/016-04/feature-modules-by-context` (`6ce9b05`) y Game `feature/019-04/composed-swagger-decorators` (`07c6513`). Ejecución en paralelo con Codex (`gpt-6-luna`). El autor fusiona en paralelo la promoción AuthUser #49 de GB-018.03.
- GB-016.04 (2026-10-25): Codex movió los archivos, pero no pudo escribir el índice de Git ni ejecutar E2E/build. La coordinación preparó los movimientos y comprobó que Git los detecta todos como renombres, con cambios solo en imports y cuatro `.gitkeep` redundantes eliminados. Sin referencias a rutas antiguas. Suites: `pnpm test` 93/93, `pnpm test:e2e` 3 pasan y 3 se omiten por diseño, lint, typecheck y build correctos. Commit `cf42b30`; PR #70 abierto.
- GB-019.04 (2026-10-25): Codex creó `favorites.swagger.ts` con `ApiFavoriteErrors()` y un decorador compuesto por operación; el controlador pasa de 455 a 315 líneas. La coordinación repitió la comparación del OpenAPI ordenado antes y después: idéntico byte a byte, 22.180 bytes. Suites: `pnpm test` 66/66, `pnpm test:e2e` 22/22, lint, typecheck y build correctos. Commit `a3aaf16`; PR #51 abierto. Tras el merge queda la promoción `develop → main` del grupo.

#### GB-018.03 — Evidencia de cierre y del grupo GB-018

- Estado: `[ RESOLVED ]` (2026-10-25). Promoción [AuthUser #49](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/49) `develop → main` fusionada por el autor a las 03:09:27 UTC con merge `e34012a31eb80d3d0434d18f474e9c6f06e52b9a`; workflows `CI` y `Release Please` en `main` terminaron `success`.
- release-please registró `No user facing commits found since 5b79827… - skipping`. Es lo esperado: GB-018 solo contiene commits `refactor`/`test`, que con `release-type: node` no generan versión, así que no hay release PR y AuthUser sigue en `v0.4.0`. La expectativa *patch/minor* de `plan/improvements.md` §5 no aplica a refactors puros.
- Producción (Render): `GET /health` → 200 y `GET /docs/openapi.json` → 200. El documento desplegado ya refleja el código nuevo, porque `/v1/auth/session` no lista el parámetro de cabecera `authorization`. No hubo migraciones ni variables nuevas.
- Grupo GB-018 `[ RESOLVED ]`: GB-018.01–.03 resueltas (spec §5.1 cumplida).
- Pendiente propuesto, fuera de esta subtarea: sincronizar `main → develop` en AuthUser, donde los commits de release 0.1.0–0.4.0 y los merges de promoción no están en `develop`.

#### GB-016.04 / GB-019.04 — Verificación de merges

- 2026-10-25: tras el aviso del autor, la coordinación verificó `MERGED` hacia `develop` y CI verde.
- GB-016.04 → `[ RESOLVED ]`: [Frontend #70](https://github.com/CarlosSV923/GameBook.Frontend/pull/70), merge `362056a5046c2d0912c4d31f7122dc5cc7d4f17b` (03:21:38 UTC); `repository-baseline` y `e2e` en `SUCCESS`; `origin/develop` contiene `src/features/{auth,catalog,favorites,layout,preferences,profile}`. Queda habilitada GB-016.05.
- GB-019.04 sigue `[ ACTIVE ]`: [Game #51](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/51), merge `fb30d97282b0ccf54053d341d1801be759d80c88` (03:22:38 UTC), CI `SUCCESS`, `favorites.swagger.ts` presente. Falta la promoción: se abrió [Game #52](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/52) `develop → main`, sin conflictos según `git merge-tree`.

#### GB-016.05 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-016.04` verificada `[ RESOLVED ]`. Rama `feature/016-05/tanstack-query-server-state` desde `origin/develop` (`362056a`). Ejecución con Codex (`gpt-6-luna`).
- 2026-10-25: Codex migró catálogo, favoritos, detalle, sugerencias e IDs favoritos a TanStack Query 5.104.5. `CatalogList` pasa de 364 a 192 líneas y `FavoritesList` de 358 a 231. No pudo ejecutar E2E ni build. La coordinación sí los ejecutó, y la red E2E de GB-016.01 detectó una regresión: sin sesión, la consulta deshabilitada de IDs favoritos queda `pending` en v5 y la acción de guardar se quedaba en «loading». La coordinación lo corrigió (solo carga con sesión) y volvió exacto un selector de la nueva prueba de scroll. Resultado: `pnpm test` 96/96, `pnpm test:e2e` 5 pasan y 3 se omiten por diseño, lint, typecheck y build correctos. Commits `9191d36` y `65e236f`; PR #71 abierto.
- Cierre (2026-10-25): [Frontend #71](https://github.com/CarlosSV923/GameBook.Frontend/pull/71) fusionado por el autor a las 03:40:19 UTC con merge `10b5736e004311bad03be644ddf5f02e715193c8`; `repository-baseline` y `e2e` en `SUCCESS`; `src/shared/query/` y `@tanstack/react-query` presentes en `origin/develop`. GB-016.05 → `[ RESOLVED ]`. Queda habilitada GB-016.06.

#### GB-019.04 — Evidencia de cierre y del grupo GB-019

- Estado: `[ RESOLVED ]` (2026-10-25). Promoción [Game #52](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/52) `develop → main` fusionada por el autor a las 03:24:16 UTC con merge `f45299fa5e7423bee067fc8466e7853991c68134`; `CI` y `Release Please` en `main` terminaron `success`.
- release-please: `No user facing commits found since 13d56f1… - skipping`. GB-019 solo contiene `refactor`/`test`, así que no hay release PR y Game sigue en `v0.4.0`.
- Producción (Render): `GET /health` → 200 (primer intento de ~111 s, por el arranque en frío tras el despliegue) y `GET /docs/openapi.json` → 200. Sin migraciones ni variables nuevas.
- Grupo GB-019 `[ RESOLVED ]`: GB-019.01–.04 resueltas (spec §5.2 cumplida).
- Pendiente propuesto, fuera de la subtarea: sincronizar `main → develop` en Game, igual que en AuthUser.

#### GB-016.06 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-016.05` verificada `[ RESOLVED ]`. Rama `feature/016-06/server-rendered-catalog` desde `origin/develop` (`10b5736`). Ejecución con Codex (`gpt-6-luna`). Decisión técnica para las E2E: un servidor IGDB/Twitch falso solo de pruebas, con *overrides* `IGDB_API_BASE_URL`/`TWITCH_TOKEN_URL` aceptados únicamente si apuntan a `127.0.0.1`/`localhost` y configurados solo en `playwright.config.ts`.
- 2026-10-25: Codex convirtió `/` en un Server Component dinámico con precarga deshidratada, añadió los *overrides* de loopback, el servidor IGDB falso y pruebas nuevas, pero no pudo ejecutar E2E ni build. La coordinación ejecutó las suites y corrigió cuatro puntos: (1) **defecto previo**: `getIgdbServerClient()` creaba un cliente por petición y la caché del token de Twitch nunca se aprovechaba, lo que con SSR habría pedido un token por visita; ahora es un *singleton* de proceso; (2) `staleTime` limitado al catálogo en lugar de global; (3) la prueba de scroll parte de una búsqueda filtrada paginada en el navegador; (4) `/_next/image` se sirve localmente en E2E para que el optimizador no llame a `images.igdb.com`. Resultado: `pnpm test` 104/104, `pnpm test:e2e` 7 pasan y 3 se omiten por diseño, lint, typecheck y build correctos (`/` dinámico). `.next/static` no contiene nombres ni valores de las credenciales IGDB ni la URL de Twitch. Commits `c891b15` y `22bbce5`; PR #72 abierto. La promoción `develop → main` del Frontend queda para GB-017.07.
- Cierre (2026-10-25): [Frontend #72](https://github.com/CarlosSV923/GameBook.Frontend/pull/72) fusionado por el autor a las 03:52:30 UTC con merge `254e1327ed1857293ee566eeae30c342200190e9`; `repository-baseline` y `e2e` en `SUCCESS`; `origin/develop` contiene `HydrationBoundary` en `src/app/page.tsx`, `home-intro.tsx` y `tests/e2e/fake-igdb-server.mjs`. GB-016.06 → `[ RESOLVED ]` y **grupo GB-016 `[ RESOLVED ]`** (spec §2, §4 y §6 cumplidos en `develop`). Queda habilitado GB-017 (GB-017.01).

#### GB-017.01 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia de grupo `GB-016` verificada `[ RESOLVED ]`. Rama `feature/017-01/archive-refined-system` desde `origin/develop` (`254e132`). Ejecución con Codex (`gpt-6-luna`) y el skill `interface-design`; entradas: spec §3, plan §3.3, revisión §1 y las fuentes de los artboards de la dirección A del canvas. Requiere revisión del autor antes de cerrarse.
- 2026-10-25: Codex reescribió `.interface-design/direction.md` (58 líneas) y `system.md` (164) con la dirección A, tokens claro/oscuro, tipografía, composición, componentes, popover anónimo y confirmación de guardado (§4.4/§4.5) y el mapa de componentes a subtareas. La coordinación recalculó los contrastes WCAG (12.50/13.18, 4.74/7.05, 6.81/5.90, coinciden) y corrigió el color de foco claro `#C6794E`, que daba 2.93:1 (no textual < 3:1), por `copper` `#A85F3B`, que da ≥ 3.70:1. Commit `740f2f8`; PR #73 abierto, pendiente de la revisión del autor.
- 2026-10-25: el autor **aprobó el sistema visual** de GB-017.01 en el chat de coordinación. GB-017.01 se cerrará cuando se verifique el merge de PR #73 en `develop`; GB-017.02 no se inicia antes.
- Cierre (2026-10-25): [Frontend #73](https://github.com/CarlosSV923/GameBook.Frontend/pull/73) fusionado por el autor a las 04:03:31 UTC con merge `a7c234dad99882db3a5ab8a16370bad08487af5a`; `repository-baseline` y `e2e` en `SUCCESS`; `origin/develop` contiene el `system.md` aprobado (foco `#A85F3B`). GB-017.01 → `[ RESOLVED ]`.

#### GB-017.02 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-017.01` verificada `[ RESOLVED ]`. Rama `feature/017-02/tailwind-tokens-primitives` desde `origin/develop` (`a7c234d`). Ejecución con Codex (`gpt-6-luna`) y `interface-design`.
- 2026-10-25: Codex implementó los tokens `@theme`, `next/font` y las primitivas. La coordinación ejecutó: `pnpm test` 112/112, `pnpm test:e2e` 7 pasan y 3 se omiten por diseño, lint, typecheck y build correctos. Tomó 24 capturas antes/después (catálogo, detalle, login y registro; 1440×900 y 375×812; claro EN, oscuro EN y claro ES) con un script Playwright reutilizable y servidor IGDB falso. Así detectó una regresión: en oscuro, el `slate` nuevo (`#2C333D`, tono de superficie) dejaba ilegibles «Create account» y «Search» del CSS heredado. Se mantiene el alias heredado `--slate` con su valor oscuro anterior hasta GB-017.03–.06. Commits `b78cb45` y `aedb95a`; PR #74 abierto.
- Cierre (2026-10-25): [Frontend #74](https://github.com/CarlosSV923/GameBook.Frontend/pull/74) fusionado por el autor a las 04:13:20 UTC con merge `5d7c1582040a59ab8483ffcb1aee20725ac35655`; `repository-baseline` y `e2e` en `SUCCESS`; `button.tsx`, `icon-button.tsx` y `text-field.tsx` presentes en `origin/develop`. GB-017.02 → `[ RESOLVED ]`. Queda habilitada GB-017.03.

#### GB-017.03 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-017.02` verificada `[ RESOLVED ]`. Rama `feature/017-03/header-hero-filters` desde `origin/develop` (`5d7c158`). Ejecución con Codex (`gpt-6-luna`) y `interface-design`; referencia: `system.md` y artboards A.
- 2026-10-25: Codex implementó cabecera, hero, título del ranking y barra de filtros con panel móvil. La coordinación ejecutó las suites y la comparación visual (24 capturas) y corrigió: (1) icono de tema sin `stroke` (punto negro); (2) CSS de elementos fuera de capa que anulaba utilidades (texto oscuro en «Create account», hero en 3 líneas por `h1 { max-width: 12ch }`), ahora en `@layer base`; (3) cabecera fija por encima del modal, con `z-index` de modales a 50 y alerta a 40; (4) dos E2E móviles que asumían controles visibles fuera del menú/panel. El hero pasa de ~380 a ~200 px a 1440×900. Resultado: `pnpm test` 113/113, `pnpm test:e2e` 8 pasan y 4 se omiten por diseño, lint, typecheck y build correctos. Commits `18c171f` y `41edd49`; PR #75 abierto.
- Cierre (2026-10-25): [Frontend #75](https://github.com/CarlosSV923/GameBook.Frontend/pull/75) fusionado por el autor a las 04:35:04 UTC con merge `69a24fc0d7649afeeb58e555129ffcd70ef18673`; `repository-baseline` y `e2e` en `SUCCESS`; `origin/develop` contiene los estilos de elementos en `@layer base`. GB-017.03 → `[ RESOLVED ]`. Queda habilitada GB-017.04.

#### GB-017.04 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-017.03` verificada `[ RESOLVED ]`. Rama `feature/017-04/index-strip-cards` desde `origin/develop` (`69a24fc`). Ejecución con Codex (`gpt-6-luna`) y `interface-design`. Interpretación técnica de spec §3 («posición en el listado»): la tira muestra la posición 1-based del elemento en la lista renderizada, rotulada «Posición»/«Position», sin presentarla como ranking cuando la lista no se ordena por puntuación.
- 2026-10-25: la primera versión de Codex pasó las pruebas automáticas, pero la revisión visual mostró que no cumplía `system.md`: etiquetas visibles truncadas en la tira, puntuación sin redondear partida («92. / 4/1 / 00»), tira de media ficha en móvil sin portada de 72 px, portada diminuta en escritorio y corazón relleno en negro. Se relanzó Codex con las capturas anotadas, y la coordinación añadió padding a la fila del título en escritorio. Codex informó que no pudo leer `../.agents/skills/interface-design/SKILL.md` desde su sandbox y siguió `system.md`. Resultado: `pnpm test` 115/115, `pnpm test:e2e` 8 pasan y 4 se omiten por diseño (aserción de puntuación ajustada a la etiqueta de la tira), lint, typecheck y build correctos. Commits `e40ea0c` y `e62fa5e`; PR #76 abierto.
- Cierre (2026-10-25): [Frontend #76](https://github.com/CarlosSV923/GameBook.Frontend/pull/76) fusionado por el autor a las 04:51:15 UTC con merge `c378f7e956dc03788c7768191ecb7709a8b2654a`; `repository-baseline` y `e2e` en `SUCCESS`; la variante `card-compact` de `IndexStrip` está presente en `origin/develop`. GB-017.04 → `[ RESOLVED ]`. Queda habilitada GB-017.05.

#### GB-017.05 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-017.04` verificada `[ RESOLVED ]`. Rama `feature/017-05/detail-modal` desde `origin/develop` (`c378f7e`). Ejecución con Codex (`gpt-6-luna`) y `interface-design`.
- 2026-10-25: Codex reestilizó el modal: fondo oscuro en ambos temas, panel con tira de detalle y ficha técnica, estados «no disponible» y error separados y confirmación de eliminar con los botones compartidos. Separó el contenido en `game-detail-content.tsx` y añadió la devolución del foco al control de apertura. La coordinación ejecutó las suites y la comparación visual, sin regresiones. Resultado: `pnpm test` 117/117, `pnpm test:e2e` 8 pasan y 4 se omiten por diseño (incluye la nueva aserción de foco), lint, typecheck y build correctos. Commits `46c58f3` y `5e1c974`; PR #77 abierto.
- Cierre (2026-10-25): [Frontend #77](https://github.com/CarlosSV923/GameBook.Frontend/pull/77) fusionado por el autor a las 05:02:04 UTC con merge `5208ad1290a7dc66193c65f294352862e7f7e1b1`; `repository-baseline` y `e2e` en `SUCCESS`. GB-017.05 → `[ RESOLVED ]`. Quedan habilitadas GB-017.06 y GB-017.08.

#### GB-017.06 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-017.05` verificada `[ RESOLVED ]`. Rama `feature/017-06/account-states-notices` desde `origin/develop` (`5208ad1`). Ejecución con Codex (`gpt-6-luna`). Decisión del autor: GB-017.08 se ejecuta después del merge de esta subtarea, para evitar conflictos en `globals.css` y `messages.ts`.
- 2026-10-25: Codex reestilizó login, registro, perfil, el modal de contraseña cambiada, la página de favoritos, los estados del catálogo y la alerta de Render, y eliminó el alias temporal de `--slate` oscuro. `globals.css` queda en 430 líneas. La coordinación amplió el script de capturas a las vistas con sesión (favoritos y perfil, 36 capturas) sin encontrar regresiones. Resultado: `pnpm test` 119/119, `pnpm test:e2e` 8 pasan y 4 se omiten por diseño, lint, typecheck y build correctos. Commits `d347541` y `ce7b778`; PR #78 abierto. Texto «…for the MVP»: el autor decidió retirarlo en GB-017.07.
- Cierre (2026-10-25): [Frontend #78](https://github.com/CarlosSV923/GameBook.Frontend/pull/78) fusionado por el autor a las 21:05:16 UTC con merge `3d6f7f62d52aa638e2ea9d7ebdc681b36f0ee287`. Checks del PR: `e2e` en `SUCCESS` (×2) y `repository-baseline` en `SUCCESS` en el run `pull_request` 40233728694, pero `FAILURE` en el run `push` 40233696431 sobre el mismo commit `ce7b778`. El fallo es intermitente y externo: `next/font/google` no pudo descargar Newsreader durante `next build` («next/font/google queries have exactly one entry»). `origin/develop` ya no contiene el alias `#91a7af`, y el CI `push` de `develop` sobre el merge `3d6f7f6` terminó `success`. GB-017.06 → `[ RESOLVED ]`.
- Riesgo registrado: el build depende de descargar fuentes de Google (CI y Vercel). Se propone resolverlo en GB-017.07 sirviendo las fuentes con `next/font/local` (mismas familias y variables).

#### GB-017.08 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencia `GB-017.05` verificada `[ RESOLVED ]`; se ejecuta después de GB-017.06 por decisión del autor. Rama `feature/017-08/favorite-save-popover-confirmation` desde `origin/develop` (`3d6f7f6`). Ejecución con Codex (`gpt-6-luna`).
- 2026-10-25: Codex implementó el panel contextual sin sesión (§4.4) y la confirmación de guardado con sesión (§4.5) en ficha y modal. Nombres nuevos: grupo «Confirm saving {name}» con «Save»/«Cancel» y en ES «Confirmar que se guarda {name}» con «Guardar»/«Cancelar». La coordinación ejecutó las suites, acotó un selector E2E ambiguo («Create account» aparece en la cabecera y en el formulario) y revisó capturas del panel en escritorio y móvil, claro/oscuro y EN/ES. Resultado: `pnpm test` 119/119, `pnpm test:e2e` 8 pasan y 4 se omiten por diseño, lint, typecheck y build correctos. Commits `12873e9` y `4ceccb7`; PR #79 abierto.
- Cierre (2026-10-25): [Frontend #79](https://github.com/CarlosSV923/GameBook.Frontend/pull/79) fusionado por el autor a las 21:22:04 UTC con merge `e33066e1b4353711dde744df6a254b2972729ea4`; `repository-baseline` y `e2e` en `SUCCESS`; `favorite-save-state.ts` y `favorite-delete-action.tsx` presentes en `origin/develop`. GB-017.08 → `[ RESOLVED ]` (criterio 2 de la spec de mejoras cumplido en `develop`). Queda habilitada GB-017.07.

#### GB-017.07 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-25). Dependencias `GB-017.06` y `GB-017.08` verificadas `[ RESOLVED ]`. Rama `feature/017-07/final-review-screenshots` desde `origin/develop` (`e33066e`). Ejecución con Codex (`gpt-6-luna`) de la parte de código: retirar el CSS heredado, corregir la regresión del fin de resultados, el texto del perfil, `next/font/local` (decisión del autor del 2026-10-25), revisión de contraste y `pnpm screenshots`. La coordinación hace después la promoción `develop → main` y release-please.
- 2026-10-25: Codex retiró el CSS heredado (`globals.css` queda en 178 líneas), corrigió el estado de fin de resultados con un marcador decorativo, quitó la mención al MVP del texto de desactivación, migró a `next/font/local` y creó `pnpm screenshots` con portadas ficticias locales y GIF (`gifenc` + `pngjs`). No pudo ejecutar build ni Playwright. La coordinación corrigió: `import.meta` en las fixtures, que fallaba con el cargador CommonJS de Playwright; el `cwd` de los servidores de capturas; un selector solo en español; y las cabeceras OFL que Codex atribuyó a «Google Inc.», sustituidas por los textos oficiales de IBM Plex y Newsreader. Resultado: `pnpm test` 120/120, `pnpm test:e2e` 8 pasan y 4 se omiten por diseño, `pnpm screenshots` genera 14 recursos (`main-flow.gif` ≈ 136 KB), lint, typecheck y build correctos, sin referencias a Google Fonts en `.next`. Contraste mínimo de texto 4.74/5.31; foco 4.19/7.38. Commits `9cb599f`, `aee6ce7` y `d533ece`; PR #80 abierto. Pendiente para cerrar la subtarea: promoción `develop → main` y release-please.
- 2026-10-26: [Frontend #80](https://github.com/CarlosSV923/GameBook.Frontend/pull/80) fusionado por el autor (2026-10-25 21:38:36 UTC) con merge `5a7cb0797a8a023dfac77018ca1087f18676e6a7`; `repository-baseline` y `e2e` en `SUCCESS`; CI de `develop` en `5a7cb07` `success`; `src/app/fonts/` (9 archivos) y `docs/screenshots/` (14 archivos) presentes en `origin/develop`. Se abrió la promoción [Frontend #81](https://github.com/CarlosSV923/GameBook.Frontend/pull/81) `develop → main`, con 27 commits de GB-016 y GB-017 y sin conflictos según `git merge-tree`. GB-017.07 sigue `[ ACTIVE ]` hasta su merge y el release de release-please.
- 2026-10-27: el autor fusionó [Frontend #81](https://github.com/CarlosSV923/GameBook.Frontend/pull/81) a las 00:30:25 UTC con merge `fa072652da64732b1545c6ce5538a6b0deafb550`; `CI` y `Release Please` en `main` terminaron `success`. Despliegue Vercel `Production` de `fa07265` en estado `success`. Producción: `/`, `/favorites`, `/login` y `/api/igdb/platforms` responden 200. El HTML inicial ya contiene «Top rated of all time» con *The Witcher 3 GOTY*, *Bloodborne GOTY* y *Super Metroid* (sin *Wortgewandt*), sin referencias a Google Fonts, y la revisión visual en navegador confirma la dirección A. release-please abrió [Frontend #82](https://github.com/CarlosSV923/GameBook.Frontend/pull/82) `gamebook-frontend` **0.11.0** (minor). Observación: el changelog lista cada `feat` dos veces (el commit original y el merge commit del PR); es cosmético y se propone como mejora aparte. GB-017.07 se cierra tras el merge de #82.
- Cierre (2026-10-27): el autor fusionó [Frontend #82](https://github.com/CarlosSV923/GameBook.Frontend/pull/82) a las 00:47:27 UTC con merge `ed6c94b04dd45e45eca8e105c2b765e5c5797f8a`; `Release Please` terminó `success` y publicó la release **gamebook-frontend v0.11.0** (Latest) con tag `gamebook-frontend-v0.11.0` → `ed6c94b`. GB-017.07 → `[ RESOLVED ]` y **grupo GB-017 `[ RESOLVED ]`**: spec §3 y criterios 2–4 cumplidos y verificados en producción.
- **Regresión detectada (de GB-017.06, ya en `develop`):** el estado «fin del catálogo» muestra un `IndexStrip` con «Unavailable» superpuesto a su mensaje. La coordinación no lo vio al revisar las capturas de GB-017.06. Se corrige en GB-017.07.

### [ RESOLVED ] GB-021 — Fichas póster de la opción B

Ámbito: `GameBook.Frontend`; **skill obligatorio `interface-design`**. Enmienda de spec §3 del 2026-10-27, tras la revisión del autor en producción: la ficha A con tira lateral genera grandes espacios en blanco y recorta los pósters. Cierre del grupo: spec §3 (enmienda) y criterio 3, verificados en producción.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-021.01** — Fichas póster 3:4 en catálogo y favoritos | GB-017 | `system.md`/`direction.md` actualizados; `GameCard` póster 3:4 sin recorte, con etiqueta de puntuación, título, «año · plataformas» truncado y acción de favoritos con nombres y flujo de GB-017.08; cuadrícula de filas uniformes (escritorio y móvil); esqueletos, pruebas unitarias, E2E y `pnpm screenshots` actualizados; capturas antes/después con datos reales y ficticios sin huecos verticales. |
| [ RESOLVED ] **GB-021.02** — Publicar las fichas póster | GB-021.01 | Promoción `develop → main`, release-please y verificación en producción (Vercel) de catálogo y favoritos. |

#### GB-021 — Inicio y cierre

- Estado: `[ ACTIVE ]` (2026-10-27). Dependencia de grupo `GB-017` verificada `[ RESOLVED ]`. Rama `feature/021-01/poster-cards` desde `origin/develop` (`5a7cb07`), en un worktree aparte porque la copia principal del Frontend está en GB-020.02. Ejecución inicial con Codex (`gpt-6-luna`) y `interface-design`.
- Decisión del autor (2026-10-27, spec §3, enmienda de ejecución): GB-021 lo implementa la coordinación directamente, sin Codex ni subagentes, con `/design` como único skill de diseño. La salida de la ejecución Codex en curso se descarta sin integrarse y la rama se rehace desde `origin/develop` actualizado.
- Implementación (coordinación, 2026-10-27): referencia `/design` en los artboards «A + B cards» (escritorio y teléfono) del canvas. Rama rehecha desde `origin/develop` (`daac946`). PR #85 → `develop`. Evidencia: unitarias 37 archivos/121 pruebas; E2E 8 en verde (escritorio y móvil), con una comprobación nueva de altura uniforme por fila y portada 3:4; `lint`, `typecheck`, `build` y Prettier en verde. Capturas con fixtures (claro/oscuro, EN/ES, escritorio/móvil) y con datos reales de IGDB de desarrollo: alturas por fila uniformes (escritorio 349 px; móvil 289/270 px), portadas sin recorte, sin huecos verticales. `pnpm screenshots` se regeneró con cuatro pósters ficticios. Ajustes relacionados: popovers de la ficha anclados a toda la ficha, separación bajo la barra de filtros y el aviso de fin de resultados ya no hereda la altura mínima del panel.
- Cierre GB-021.01: `[ RESOLVED ]` (2026-10-27). PR #85 fusionado por el autor en `develop` (`8e42f64`), con checks `repository-baseline` y `e2e` en verde; `GameCard` póster presente en `develop`.
- GB-021.02: `[ ACTIVE ]` (2026-10-27), dependencia GB-021.01 `[ RESOLVED ]`. Promoción `develop → main` por PR #86 (incluye #83, #84 y #85); la fusiona el autor.
- Promoción verificada (2026-10-28): #86 `MERGED` en `main` (`b344a01`), checks `repository-baseline` y `e2e` en verde. Despliegue de producción de Vercel `success` sobre `b344a01`. El HTML de `https://gamebook-frontend.vercel.app/` trae 20 fichas `data-game-card` con `aspect-[3/4]`, ninguna tira `data-variant="card"` y ninguna petición a Google Fonts. Capturas de producción (claro/oscuro, EN/ES, escritorio/móvil): altura por fila uniforme (349 px en escritorio; 289/270 px en móvil) y pósters sin recorte. GitHub detecta ya la licencia MIT del Frontend. release-please abrió #87 (0.12.0); su changelog repite la entrada `feat` (commit de merge y commit original), el problema de duplicados ya conocido. 
- Cierre GB-021.02 y GB-021: `[ RESOLVED ]` (2026-10-28). #87 `MERGED` en `main` (`2adc429`); etiqueta `gamebook-frontend-v0.12.0` → `2adc429`, release publicada (no borrador), `package.json` 0.12.0 en `main` y despliegue de producción de Vercel `success` sobre `2adc429`. La coordinación tomará las capturas antes/después.

## 4. Backends

### [ RESOLVED ] GB-018 — DDD de AuthUser

Ámbito: `GameBook.Microservice.AuthUser`. Sin cambios de OpenAPI, esquema ni migraciones; `test`, `test:e2e`, `lint`, `typecheck` y `build` en verde. Cierre del grupo: spec §5.1.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-018.01** — Mutaciones por el agregado `User` con concurrencia optimista | GB-015.01 | `changePassword`/`disable` en dominio como única fuente de la regla de `sessionVersion`; `save(user, expectedVersion)` sustituye `replacePassword`/`disable`; pruebas de conflicto y revocación. |
| [ RESOLVED ] **GB-018.02** — Guard Bearer y `@CurrentSession()` | GB-018.01 | Controladores sin parseo manual de `Authorization`; casos de uso protegidos reciben la identidad autenticada; `GET /v1/auth/session` idéntico (OpenAPI comparado y pruebas de Game contra AuthUser local). |
| [ RESOLVED ] **GB-018.03** — Composición fuera de la capa de aplicación | GB-018.02 | Módulo de composición; `ApplicationModule` sin `InfrastructureModule` ni `ConfigService`; limpieza de `try/catch` redundante y de exposición de `passwordHash`; promoción `develop → main` y release-please. |

### [ RESOLVED ] GB-019 — DDD de Game

Ámbito: `GameBook.Microservice.Game`. Mismas restricciones que GB-018. Cierre del grupo: spec §5.2.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-019.01** — Casos de uso sin NestJS y puertos inyectados | GB-015.01 | Casos de uso sin decoradores Nest, factorías y tokens de puerto; controladores dependen de puertos; pruebas en verde. |
| [ RESOLVED ] **GB-019.02** — Instantánea y eliminación vía agregado | GB-019.01 | `Favorite.updateSnapshot()` y repositorio de escritura reducido a `findByIdentity`/`save`/`delete`; aislamiento por `userId` probado. |
| [ RESOLVED ] **GB-019.03** — Puerto de lectura para listado y sugerencias | GB-019.02 | `FavoriteReadModel` en `application/ports/` con implementación Prisma; filtros, paginación y sugerencias idénticos. |
| [ RESOLVED ] **GB-019.04** — Decoradores Swagger compuestos | GB-019.03 | `favorites.controller.ts` reducido; `/docs/openapi.json` idéntico antes y después (diff adjunto); promoción `develop → main` y release-please. |

## 5. Documentación y publicación

### [ RESOLVED ] GB-020 — Documentación, diagramas y metadatos

Ámbito: los cuatro repositorios. **Skill obligatorio `archify` 3.2** en `.02`–`.05`. Cierre del grupo: spec §7 y criterios 9–12.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-020.01** — Metadatos de GitHub y licencia MIT | GB-015.01 | Descripción, topics y homepage de spec §7.2 aplicados con `gh repo edit` y comprobados; `LICENSE` MIT por PR en los cuatro repos. |
| [ RESOLVED ] **GB-020.02** — Diagramas Archify 3.2 de Frontend | GB-015.02, GB-017 | Arquitectura y secuencia (catálogo IGDB/Twitch) EN/ES: `validate --quality showcase`, `deliver`, `visual-check` y revisión perceptual registrados; SVG para README; índice Pages; recibos fuera del repo. |
| [ RESOLVED ] **GB-020.03** — Diagramas Archify 3.2 de AuthUser | GB-015.02, GB-018 | Arquitectura y secuencia (login y revocación) EN/ES con las mismas comprobaciones y entregables. |
| [ RESOLVED ] **GB-020.04** — Diagramas Archify 3.2 de Game | GB-015.02, GB-019 | Arquitectura y secuencia (favorito protegido con validación de sesión) EN/ES con las mismas comprobaciones y entregables. |
| [ RESOLVED ] **GB-020.05** — Diagrama general de GameBook.System | GB-020.02, GB-020.03, GB-020.04 | Diagrama general EN/ES regenerado con 3.2, SVG, Pages y recibos fuera del repo. |
| [ RESOLVED ] **GB-020.06** — README y docs de Frontend | GB-020.02, GB-021 | README EN/ES con estructura spec §7.1, capturas y GIF; `docs/development.md` y `.es.md`; enlaces válidos; sin referencias históricas. |
| [ RESOLVED ] **GB-020.07** — README y docs de AuthUser | GB-020.03 | Mismos criterios, sin capturas de UI; Swagger como demo. |
| [ RESOLVED ] **GB-020.08** — README y docs de Game | GB-020.04 | Mismos criterios, sin capturas de UI; Swagger como demo. |
| [ RESOLVED ] **GB-020.09** — README de GameBook.System | GB-020.05, GB-020.06 | README EN/ES reducido según spec §7.1, con capturas del Frontend, diagrama general y Compose resumido. |
| [ RESOLVED ] **GB-020.10** — Sincronizar SDD y verificar entrega | GB-020.07, GB-020.08, GB-020.09 | `specs/plan/tasks/improvements.md` y `reviews/` publicados en `GameBook.System` por PR; cuatro repos verificados (README, Pages HTTP 200, licencia, metadatos, enlaces bidireccionales, cero secretos). |
| [ RESOLVED ] **GB-020.11** — Regenerar el GIF del flujo principal | GB-020.06, GB-020.09 | `pnpm screenshots` genera un GIF del flujo completo (spec §7.1, enmienda 2026-10-28): cada paso dura ≥ 2 s, se captura tras las animaciones y tiene paleta propia por fotograma. Se copia a `GameBook.System/docs/screenshots/en/` y se revisan los fotogramas. |

#### GB-020.01–.04 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-27). Dependencias verificadas: `GB-015.01`/`GB-015.02` y los grupos `GB-017`, `GB-018` y `GB-019` en `[ RESOLVED ]`.
- GB-020.01 (coordinación): `gh repo edit` aplicó la descripción, los topics y la homepage de spec §7.2 a los cuatro repositorios (verificado con `gh repo view`). La homepage de `GameBook.System` apunta de momento al diagrama EN de Pages, porque la raíz de Pages da 404 hasta que GB-020.05 cree el índice. PR con `LICENSE` MIT (titular Carlos Sesme Vera): Frontend #83, AuthUser #50 y Game #53 hacia `develop`, y System #9 hacia `main`. En las aplicaciones, GitHub detectará la licencia tras la próxima promoción a `main`.
- GB-020.02–.04: Codex (`gpt-6-luna`) escribe las fuentes JSON de arquitectura y secuencia EN/ES y el índice de Pages. La coordinación ejecuta `validate`/`deliver`/`visual-check`/exportación, porque el sandbox de Codex no puede lanzar el renderizador de Archify (`spawn EPERM`, visto en GB-015.02). Ramas: Frontend `feature/020-02/archify-diagrams` (`5a7cb07`), AuthUser `feature/020-03/archify-diagrams` (`4144f92`) y Game `feature/020-04/archify-diagrams` (`fb30d97`).

#### GB-020.01–.04 — Cierre

- Estado: `[ RESOLVED ]` (2026-10-27). PR fusionados por el autor y verificados en GitHub (`MERGED`, destino correcto, CI en verde): licencias Frontend #83, AuthUser #50 y Game #53 → `develop`, y System #9 → `main`; diagramas Frontend #84, AuthUser #51 y Game #54 → `develop`. `LICENSE` MIT presente en `develop` de las tres aplicaciones; la API de GitHub detecta MIT en `GameBook.System`.
- Diagramas: la coordinación corrigió en las fuentes de Codex la geometría (arquitectura de Game ≤ 1100 px), etiquetas largas, notas solapadas y las bandas de fase de la secuencia de AuthUser. Rehízo además la secuencia del Frontend con el flujo real del código (render en servidor → hidratación → paginación por `/api/igdb/games`) y quitó el participante «limiter», que pasa a ser una nota. Las variantes ES declaran `locale: es` (`<html lang="es">` y visor en español).
- Evidencia (no versionada, en `architecture/.evidence/` ignorado): `validate --quality showcase` 9/9 en las 12 fuentes; `deliver` + `visual-check --require-provenance` con containment, readability, viewerChrome, themeStates y captures en `pass` para los 12 HTML; revisión perceptual de capturas oscuras/claras y de los SVG exportados (menú Export del visor, «SVG · Auto» de doble tema) en ambos esquemas de color; topología EN/ES idéntica, comprobada por script. En `develop`, cada repo tiene 4 SVG y ningún recibo `visual-check`/`delivery`.
- Pendiente en GB-020.06–.08: los README solo cambiaron la imagen PNG por el SVG; su reescritura y el enlace a la secuencia van en esas subtareas.

#### GB-020.05–.08 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-28). Dependencias verificadas `[ RESOLVED ]`: GB-020.02, GB-020.03 y GB-020.04 (para .05–.08) y el grupo GB-021 (para .06). Ejecución con Codex (`gpt-6-luna`), una por repositorio y en paralelo, a petición del autor.
- Ramas: System `feature/020-05/system-diagram` desde `main` (`c14b3e0`); Frontend `feature/020-06/readme-docs` (`8e42f64`), AuthUser `feature/020-07/readme-docs` (`09cd4cb`) y Game `feature/020-08/readme-docs` (`178e0c1`) desde `develop`.
- GB-020.05: Codex escribe las fuentes JSON EN/ES y el índice de Pages; la coordinación ejecuta `validate`/`deliver`/`visual-check`/exportación SVG y retira los recibos versionados. GB-020.06–.08: README EN/ES con la estructura de spec §7.1, con VideoExplorer como referencia de estilo, y `docs/development.md`/`.es.md` por repositorio.
- Revisión de la coordinación (2026-10-28): en los README, las SVG se incrustaron en vez de enlazarse (spec §7.4), se quitó la imagen de arquitectura duplicada en los backends, se alinearon la línea de estado (última versión) y el enlace de idioma, se añadieron Node.js 24, `db:generate` y el par RS256 local, y los enlaces a la raíz de Render, que no tiene página, pasaron a apuntar a los repositorios. Comprobado: Prettier, enlaces relativos e imágenes existentes, encabezados EN/ES iguales y sin secretos. En Frontend, los enlaces externos responden HTTP 200. En AuthUser y Game, las páginas de Pages de la secuencia y el índice existirán tras la promoción a `main` (se verifican en GB-020.10). En el diagrama de System se añadieron las referencias de evidencia que exige el validador, «GameBook.System» pasó a nodo `cloud` (Codex lo había tipado como `messagebus`), la arista a AuthUser dice «session» en vez de «profile» y se separó una etiqueta del rótulo de la frontera Render. `validate` 9/9, `deliver` y `visual-check` en `pass` para EN/ES, con la misma topología; se retiraron los recibos `visual-check` versionados.
- PR: System #10 → `main`; Frontend #88, AuthUser #52 y Game #55 → `develop`.
- Cierre GB-020.05–.08: `[ RESOLVED ]` (2026-10-28). Los cuatro PR están fusionados por el autor y verificados en GitHub: System #10 en `main` (`5cc8b47`), y Frontend #88 (`385ddf4`), AuthUser #52 (`868d991`) y Game #55 (`7ed7425`) en `develop`, con checks en verde (System no tiene CI). `docs/development.md` y `.es.md` están en `develop` de las tres aplicaciones. El workflow de Pages de System terminó en `success`: el índice, los dos HTML y la SVG responden HTTP 200. La homepage de `GameBook.System` pasó al índice de Pages (`https://carlossv923.github.io/GameBook.System/`), como fija spec §7.2; queda así cerrada la nota pendiente de GB-020.01.

#### GB-020.09 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-28). Dependencias GB-020.05 y GB-020.06 verificadas `[ RESOLVED ]`. Rama `feature/020-09/readme` desde `main` (`5cc8b47`). Ejecución con Codex (`gpt-6-luna`): README EN/ES reducido con la estructura de spec §7.1, capturas del Frontend copiadas a `docs/screenshots/`, el diagrama general y `docs/development.md`/`.es.md` con la guía completa de Compose y el resumen de despliegue.
- Revisión de la coordinación: datos de Compose contrastados con `compose.yaml` (contextos de build, servicios, health checks y nombres de variables), tablas formateadas con Prettier, enlaces relativos e imágenes existentes, encabezados EN/ES iguales y capturas idénticas a las del Frontend. Los enlaces a `docs/development.md` de cada aplicación en `main` dan 404 hasta promover `develop → main` en las tres; se verifican en GB-020.10. PR System #11 → `main`.
- Cierre GB-020.09: `[ RESOLVED ]` (2026-10-28). #11 fusionado por el autor en `main` (`d5efcbd`); README de 76 líneas, `docs/development*.md` y `docs/screenshots/` presentes en `main`.
- Promociones `develop → main` previas a GB-020.10 (2026-10-28; plan §5): Frontend #89, AuthUser #53 y Game #56. Llevan a `main` y a Pages los README, las guías y, en AuthUser y Game, los diagramas y la licencia. Son commits `docs`, así que no se espera versión nueva de release-please.
- Promociones verificadas (2026-10-28): #89 (`66f1b6b`), #53 (`6c5c6cb`) y #56 (`6564540`) están `MERGED` en `main`. El workflow de Pages se republicó en `success` en AuthUser y Game. release-please no abrió PR de versión. GitHub detecta la licencia MIT en las tres aplicaciones.

#### GB-020.11 — Inicio

- Estado: `[ ACTIVE ]` (2026-10-28), a petición del autor: el GIF era muy corto (3 fotogramas de 0,8 s), el modal se capturaba a medio abrir y la paleta única degradaba los fotogramas oscuros. Dependencias GB-020.06 y GB-020.09 verificadas `[ RESOLVED ]`. La coordinación lo implementa sin Codex, en la rama Frontend `feature/020-11/main-flow-gif` desde `develop` (`385ddf4`). La copia a System se hará en un PR aparte hacia `main`, cuando termine GB-020.10.
- Implementación (2026-10-28): una prueba propia de `pnpm screenshots` graba el GIF a 1280×800 con 8 pasos de 2,5–4 s (bucle de 23,5 s): catálogo, cuadrícula de pósters, detalle abierto, aviso sin sesión, formulario de inicio de sesión, confirmación de guardado, ficha guardada y favoritos. Cada paso se captura cuando terminan las fuentes, las imágenes visibles y las animaciones finitas. Cada fotograma lleva su propia paleta y difuminado Floyd–Steinberg. Pesa 560 KB (antes, 178 KB y 2,4 s). Se revisaron los 8 fotogramas muestreando el GIF en reproducción. `typecheck`, `lint` y Prettier en verde. PR Frontend #90 → `develop` y System #13 → `main` (copia idéntica byte a byte).
- Fusionados y verificados (2026-10-28): #90 en `develop` (`5b6e978`, CI en verde) y #13 en `main` de System (`702c443`); el GIF servido desde `main` de System es idéntico al del Frontend (571.701 bytes). El README del Frontend muestra el GIF de `main`, así que se abrió la promoción #91. GB-020.11 se cierra cuando se fusione.
- Cierre GB-020.11: `[ RESOLVED ]` (2026-10-28). La promoción #91 está fusionada en `main` del Frontend (`8823630`) y Vercel desplegó en producción sobre ese commit (`success`). release-please no abrió versión. El GIF servido desde `main` del Frontend es idéntico al de System (571.701 bytes).

#### GB-020 — Cierre del grupo y de la fase

- Estado: `[ RESOLVED ]` (2026-10-28). GB-020.01–.11 están `[ RESOLVED ]` y cumplen spec §7 y los criterios 9–12 de §9: README EN/ES con la estructura de §7.1 y `docs/development*.md` en los cuatro repositorios (9); descripción, topics, homepage y MIT verificados (10); diagramas Archify 3.2 publicados en Pages, con SVG EN/ES en los README y ningún recibo `visual-check`/`delivery` en `architecture/` de `main` en los cuatro repositorios (11); y `specs/`, `plan/` y `tasks/` de la fase en `GameBook.System` (12), con este estado final sincronizado por PR.
- Con GB-015–GB-021 en `[ RESOLVED ]`, la fase de mejoras queda completa. Versiones finales: Frontend v0.12.0, AuthUser v0.4.0 y Game v0.4.0.
- Sincronización final verificada (2026-10-28): System #14 está fusionado en `main` (`e909e43`) y el `tasks/improvements.md` publicado muestra GB-020 `[ RESOLVED ]` y ninguna subtarea `[ ACTIVE ]` ni `[ NEW ]`. Criterio 12 cumplido.

#### GB-020.10 — Inicio

- Resultado (2026-10-28): Codex verificó los cuatro repositorios en `main`. Los README EN/ES se enlazan entre sí y con System, GitHub detecta MIT, los metadatos coinciden con spec §7.2, Pages y los 41 enlaces externos distintos responden HTTP 200, los enlaces relativos resuelven y no hay secretos (las URL PostgreSQL con contraseña son solo valores locales en pruebas, `.env.example` y `compose.yaml`). Su sincronización pasó Prettier por las copias, lo que renumeró una lista del plan y rompía referencias como «plan §3.3 paso 8»; la coordinación las volvió a copiar sin cambios desde el workspace (sus enlaces relativos ya resuelven en System) y aplicó solo la enmienda GB-016.03 a `contracts/igdb-mapping.md`. PR System #12 → `main`.
- Cierre GB-020.10: `[ RESOLVED ]` (2026-10-28). #12 fusionado por el autor en `main` de System (`4e73758`); `specs/`, `plan/` y `tasks/improvements.md` y `reviews/` están publicados allí. La copia de `tasks/improvements.md` en System refleja el estado previo a este cierre; el estado final se sincronizará con el cierre del grupo GB-020.
- Estado: `[ ACTIVE ]` (2026-10-28). Dependencias GB-020.07, GB-020.08 y GB-020.09 verificadas `[ RESOLVED ]`. Rama `feature/020-10/sdd-sync` desde `main` de System. Ejecución con Codex (`gpt-6-luna`). Parte A: copia de `specs/plan/tasks/improvements.md` y `reviews/`, con los enlaces adaptados al repositorio público como en GB-014 y la enmienda GB-016.03 de `contracts/igdb-mapping.md`; no se sobrescriben los archivos MVP ni `AGENTS.md`/`CLAUDE.md`, que se adaptaron al publicarse. Parte B: verificación de la entrega en los cuatro repositorios (puerta I): README, Pages HTTP 200, licencia, metadatos, enlaces bidireccionales y cero secretos.

### [ RESOLVED ] GB-022 — Ajustes posteriores al cierre

Ámbito: Frontend, AuthUser, Game y `GameBook.System`. Spec §10 y plan §3.7. Cierre del grupo: spec §10.5.

| Estado y subtarea | Depende de | Entregable y comprobación |
| --- | --- | --- |
| [ RESOLVED ] **GB-022.01** — Aviso de fin en favoritos | GB-020 | Favoritos muestra «Personal shelf» y su texto de fin en EN/ES; el catálogo no cambia; pruebas unitaria y E2E; PR a `develop`. |
| [ RESOLVED ] **GB-022.02** — Sincronizar `main → develop` y automatizarlo | GB-020 | PR `main → develop` fusionado en las tres aplicaciones; `release-please.yml` abre ese PR tras cada release (sin duplicarlo). |
| [ RESOLVED ] **GB-022.03** — Changelog sin duplicados | GB-022.02 | `CHANGELOG.md` sin entradas repetidas y la regla de merge de spec §10.3 documentada en `docs/development*.md` de las tres aplicaciones; PR a `develop` fusionado con «Rebase and merge». |
| [ RESOLVED ] **GB-022.04** — Quitar el enlace al canvas | GB-020 | `specs/improvements.md` y la revisión lo mencionan sin enlace; sincronizado con `GameBook.System` por PR. |
| [ RESOLVED ] **GB-022.05** — Publicar y verificar | GB-022.01, GB-022.02, GB-022.03 | Promociones `develop → main`; la release de Frontend sale sin entradas repetidas y abre sola el PR `main → develop`. |

#### GB-022 — Inicio

- Aprobación: el autor aprobó spec §10 y el plan §3.7 el 2026-10-28, incluido el cambio de `merge_commit_message` en los tres repositorios.
- Estado: GB-022.01, .02 y .04 `[ ACTIVE ]` (2026-10-28); su dependencia de grupo GB-020 está `[ RESOLVED ]`. GB-022.03 espera a GB-022.02 y GB-022.05 a .01–.03.
- Ejecución: la coordinación hace GB-022.01 (ajuste del Frontend, sin Codex), las operaciones de Git, el cambio de configuración y GB-022.04. Codex (`gpt-6-luna`) escribe el paso de `release-please.yml` en cada aplicación (GB-022.02) y la limpieza de `CHANGELOG.md` (GB-022.03).
- GB-022.01: `CollectionPagination` recibe el contexto y favoritos usa `favorites.endOfResults` con el rótulo «Personal shelf» / «Estantería personal». Pruebas unitarias: 37 archivos y 122 pruebas, incluida una nueva para favoritos en EN/ES. E2E: 8 pasan; el recorrido con sesión comprueba el aviso y que no aparece «Public catalog». `lint`, `typecheck` y Prettier en verde. PR Frontend #93.
- GB-022.02: PR únicos `main → develop` (solo difieren manifiesto, `CHANGELOG.md` y la versión de `package.json`; se fusionan sin conflictos): Frontend #92, AuthUser #54 y Game #57. Codex añadió a `release-please.yml` el paso que abre el PR de sincronización cuando hay `release_created` y lo omite si ya hay uno abierto, y una frase en `docs/development*.md`. La coordinación unificó el paso en las tres aplicaciones y validó el YAML con js-yaml. PR Frontend #94, AuthUser #55 y Game #58.
- GB-022.04: el canvas se menciona sin enlace en `specs/improvements.md` y en la revisión. Ningún Markdown público de los cuatro repositorios lo enlaza salvo las copias de System, que se actualizan por PR.
- Cierre GB-022.01, .02 y .04: `[ RESOLVED ]` (2026-10-28). Los ocho PR están fusionados por el autor y verificados: Frontend #93 (`a1247e1`), #92 (`1c9e079`) y #94 (`37d7c90`); AuthUser #54 (`d99c0ba`) y #55 (`f961913`); Game #57 (`052623c`) y #58 (`f64585b`); System #15 (`c291557`). En las tres aplicaciones, `develop` contiene todo `main` (0 commits solo en `main`) y la misma versión en `package.json`. El paso automático de sincronización se comprobará en la próxima release (GB-022.05).
- GB-022.03: `[ ACTIVE ]` (2026-10-28); dependencia GB-022.02 `[ RESOLVED ]`. Al aplicar `merge_commit_message=BLANK`, GitHub respondió 422: solo admite `PR_TITLE`+`PR_BODY`, `PR_TITLE`+`BLANK` o `MERGE_MESSAGE`+`PR_TITLE`, y en las tres el merge commit repite el `feat`. No se cambió ninguna configuración. El autor eligió «Rebase and merge» para los PR de funcionalidad (spec §10.3, enmienda del 2026-10-28).
- Ramas `feature/022-03/changelog-dedupe` desde `develop` en las tres aplicaciones. Ejecución con Codex (`gpt-6-luna`), una por repositorio.
- Revisión (2026-10-28): se quitaron 25 entradas en Frontend, 10 en AuthUser y 14 en Game. Un script comprobó que cada entrada eliminada apuntaba a un merge commit (dos padres) y que su texto queda una vez en la misma release; no quedan duplicados ni se perdieron secciones. La coordinación corrigió una entrada de Frontend 0.2.0, donde Codex había conservado el merge commit en lugar del original, y restauró en los docs del Frontend la frase de la sincronización automática, que Codex había sustituido. PR Frontend #95, AuthUser #56 y Game #59, a fusionar con «Rebase and merge».
- Cierre GB-022.03: `[ RESOLVED ]` (2026-10-28). #95 (`9e60a1c`), #56 (`449d481`) y #59 (`0979d65`) están fusionados en `develop` y la limpieza de `CHANGELOG.md` está publicada allí. Se fusionaron con merge commit, no con rebase; como son commits `docs`, release-please no los lista y no generan duplicados. La regla se aplicará en los próximos PR `feat`/`fix`.
- GB-022.05: `[ ACTIVE ]` (2026-10-28); dependencias GB-022.01–.03 `[ RESOLVED ]`. Aviso: #93 (`fix(favorites)`) se fusionó con merge commit antes de la regla, así que la release 0.12.1 del Frontend tendrá ese `fix` duplicado. Es el último caso heredado y se tratará en el PR de release.
- Promociones fusionadas (2026-10-28): Frontend #96 (`d80cae9`), AuthUser #57 (`b8f7412`) y Game #60 (`596fd5c`); AuthUser y Game no generan release (solo `ci`/`docs`). En el Frontend, el autor fusionó la release #97 (`499c6e5`): `gamebook-frontend-v0.12.1` está publicada y su changelog lista dos veces `fix(favorites)` (`a1247e1` es el merge de #93 y `9eab334` el commit original), como se esperaba. Al publicarse la release, el workflow abrió solo #98 `chore: sync main into develop` (autor `github-actions`): **la automatización de GB-022.02 queda verificada**. Su CI muestra `action_required` porque GitHub pide aprobar los workflows de PR creados por Actions; no bloquea, ya que no hay checks obligatorios.
- #98 está fusionado con merge commit (`8de5de8`) y `develop` contiene todo `main`. La entrada heredada `a1247e1` (merge commit de #93) se quita de `CHANGELOG.md` y se conserva `9eab334`, en el PR Frontend #99, a fusionar con «Rebase and merge». Las notas de la release v0.12.1 en GitHub no se modifican.
- Cierre GB-022.05 y del grupo GB-022: `[ RESOLVED ]` (2026-10-28). #99 está fusionado en `develop` (`ac31153`); su `CHANGELOG.md` tiene 13 secciones y ninguna entrada repetida. `main` lo recibirá en la próxima promoción. Criterios de spec §10.5: (a) favoritos muestra su aviso de fin en EN/ES, con pruebas unitaria y E2E (v0.12.1 en producción); (b) tras la release 0.12.1, el workflow abrió solo #98 y `develop` contiene todo `main`; el duplicado de 0.12.1 venía de #93, fusionado antes de la regla, y ya está corregido; (c) ningún archivo público enlaza el canvas.
- Pendiente de hábito, no de código: #95, #56, #59 y #99 se fusionaron con merge commit en vez de «Rebase and merge». Al ser `docs` no generaron duplicados, pero el próximo PR `feat`/`fix` hacia `develop` debe fusionarse con «Rebase and merge»; esa será la primera release que compruebe la regla en la práctica.

## 6. Registro de PR

| Subtarea | Repositorio | PR | Origen → destino | Estado |
| --- | --- | --- | --- | --- |
| GB-020.10 | `GameBook.System` | [#12](https://github.com/CarlosSV923/GameBook.System/pull/12) | `feature/020-10/sdd-sync` → `main` | `MERGED` |
| GB-022.05 | `GameBook.Frontend` | [#99](https://github.com/CarlosSV923/GameBook.Frontend/pull/99) | `feature/022-05/changelog-0.12.1` → `develop` (rebase) | `MERGED` |
| GB-022.05 | `GameBook.Frontend` | [#98](https://github.com/CarlosSV923/GameBook.Frontend/pull/98) | `main` → `develop` (sincronización automática) | `MERGED` |
| GB-022.05 | `GameBook.Frontend` | [#97](https://github.com/CarlosSV923/GameBook.Frontend/pull/97) | release-please → `main` (release 0.12.1) | `MERGED` |
| GB-022.05 | `GameBook.Frontend` | [#96](https://github.com/CarlosSV923/GameBook.Frontend/pull/96) | `develop` → `main` (promoción) | `MERGED` |
| GB-022.05 | `GameBook.Microservice.AuthUser` | [#57](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/57) | `develop` → `main` (promoción) | `MERGED` |
| GB-022.05 | `GameBook.Microservice.Game` | [#60](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/60) | `develop` → `main` (promoción) | `MERGED` |
| GB-022.03 | `GameBook.Frontend` | [#95](https://github.com/CarlosSV923/GameBook.Frontend/pull/95) | `feature/022-03/changelog-dedupe` → `develop` (rebase) | `MERGED` |
| GB-022.03 | `GameBook.Microservice.AuthUser` | [#56](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/56) | `feature/022-03/changelog-dedupe` → `develop` (rebase) | `MERGED` |
| GB-022.03 | `GameBook.Microservice.Game` | [#59](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/59) | `feature/022-03/changelog-dedupe` → `develop` (rebase) | `MERGED` |
| GB-022.04 | `GameBook.System` | [#15](https://github.com/CarlosSV923/GameBook.System/pull/15) | `feature/022-04/canvas-mention` → `main` | `MERGED` |
| GB-022.01 | `GameBook.Frontend` | [#93](https://github.com/CarlosSV923/GameBook.Frontend/pull/93) | `feature/022-01/favorites-end-notice` → `develop` | `MERGED` |
| GB-022.02 | `GameBook.Frontend` | [#92](https://github.com/CarlosSV923/GameBook.Frontend/pull/92) | `main` → `develop` (sincronización) | `MERGED` |
| GB-022.02 | `GameBook.Microservice.AuthUser` | [#54](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/54) | `main` → `develop` (sincronización) | `MERGED` |
| GB-022.02 | `GameBook.Microservice.Game` | [#57](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/57) | `main` → `develop` (sincronización) | `MERGED` |
| GB-022.02 | `GameBook.Frontend` | [#94](https://github.com/CarlosSV923/GameBook.Frontend/pull/94) | `feature/022-02/release-sync-pr` → `develop` | `MERGED` |
| GB-022.02 | `GameBook.Microservice.AuthUser` | [#55](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/55) | `feature/022-02/release-sync-pr` → `develop` | `MERGED` |
| GB-022.02 | `GameBook.Microservice.Game` | [#58](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/58) | `feature/022-02/release-sync-pr` → `develop` | `MERGED` |
| GB-020 (cierre) | `GameBook.System` | [#14](https://github.com/CarlosSV923/GameBook.System/pull/14) | `feature/020-close/final-task-state` → `main` (estado final de tareas) | `MERGED` |
| GB-020.11 | `GameBook.Frontend` | [#91](https://github.com/CarlosSV923/GameBook.Frontend/pull/91) | `develop` → `main` (promoción) | `MERGED` |
| GB-020.11 | `GameBook.System` | [#13](https://github.com/CarlosSV923/GameBook.System/pull/13) | `feature/020-11/main-flow-gif` → `main` | `MERGED` |
| GB-020.11 | `GameBook.Frontend` | [#90](https://github.com/CarlosSV923/GameBook.Frontend/pull/90) | `feature/020-11/main-flow-gif` → `develop` | `MERGED` |
| GB-020.10 | `GameBook.Frontend` | [#89](https://github.com/CarlosSV923/GameBook.Frontend/pull/89) | `develop` → `main` (promoción) | `OPEN` |
| GB-020.10 | `GameBook.Microservice.AuthUser` | [#53](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/53) | `develop` → `main` (promoción) | `OPEN` |
| GB-020.10 | `GameBook.Microservice.Game` | [#56](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/56) | `develop` → `main` (promoción) | `OPEN` |
| GB-020.09 | `GameBook.System` | [#11](https://github.com/CarlosSV923/GameBook.System/pull/11) | `feature/020-09/readme` → `main` | `MERGED` |
| GB-020.05 | `GameBook.System` | [#10](https://github.com/CarlosSV923/GameBook.System/pull/10) | `feature/020-05/system-diagram` → `main` | `MERGED` |
| GB-020.06 | `GameBook.Frontend` | [#88](https://github.com/CarlosSV923/GameBook.Frontend/pull/88) | `feature/020-06/readme-docs` → `develop` | `MERGED` |
| GB-020.07 | `GameBook.Microservice.AuthUser` | [#52](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/52) | `feature/020-07/readme-docs` → `develop` | `MERGED` |
| GB-020.08 | `GameBook.Microservice.Game` | [#55](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/55) | `feature/020-08/readme-docs` → `develop` | `MERGED` |
| GB-021.02 | `GameBook.Frontend` | [#87](https://github.com/CarlosSV923/GameBook.Frontend/pull/87) | `release-please--branches--main--components--gamebook-frontend` → `main` (release 0.12.0) | `MERGED` |
| GB-021.02 | `GameBook.Frontend` | [#86](https://github.com/CarlosSV923/GameBook.Frontend/pull/86) | `develop` → `main` (promoción) | `MERGED` |
| GB-021.01 | `GameBook.Frontend` | [#85](https://github.com/CarlosSV923/GameBook.Frontend/pull/85) | `feature/021-01/poster-cards` → `develop` | `MERGED` |
| GB-020.02 | `GameBook.Frontend` | [#84](https://github.com/CarlosSV923/GameBook.Frontend/pull/84) | `feature/020-02/archify-diagrams` → `develop` | `MERGED` |
| GB-020.03 | `GameBook.Microservice.AuthUser` | [#51](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/51) | `feature/020-03/archify-diagrams` → `develop` | `MERGED` |
| GB-020.04 | `GameBook.Microservice.Game` | [#54](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/54) | `feature/020-04/archify-diagrams` → `develop` | `MERGED` |
| GB-020.01 | `GameBook.Frontend` | [#83](https://github.com/CarlosSV923/GameBook.Frontend/pull/83) | `feature/020-01/mit-license` → `develop` | `MERGED` |
| GB-020.01 | `GameBook.Microservice.AuthUser` | [#50](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/50) | `feature/020-01/mit-license` → `develop` | `MERGED` |
| GB-020.01 | `GameBook.Microservice.Game` | [#53](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/53) | `feature/020-01/mit-license` → `develop` | `MERGED` |
| GB-020.01 | `GameBook.System` | [#9](https://github.com/CarlosSV923/GameBook.System/pull/9) | `feature/020-01/mit-license` → `main` | `MERGED` |
| GB-017.07 | `GameBook.Frontend` | [#82](https://github.com/CarlosSV923/GameBook.Frontend/pull/82) | `release-please--branches--main--components--gamebook-frontend` → `main` (release 0.11.0) | `MERGED` |
| GB-017.07 | `GameBook.Frontend` | [#81](https://github.com/CarlosSV923/GameBook.Frontend/pull/81) | `develop` → `main` (promoción) | `MERGED` |
| GB-017.07 | `GameBook.Frontend` | [#80](https://github.com/CarlosSV923/GameBook.Frontend/pull/80) | `feature/017-07/final-review-screenshots` → `develop` | `MERGED` |
| GB-017.08 | `GameBook.Frontend` | [#79](https://github.com/CarlosSV923/GameBook.Frontend/pull/79) | `feature/017-08/favorite-save-popover-confirmation` → `develop` | `MERGED` |
| GB-017.06 | `GameBook.Frontend` | [#78](https://github.com/CarlosSV923/GameBook.Frontend/pull/78) | `feature/017-06/account-states-notices` → `develop` | `MERGED` |
| GB-017.05 | `GameBook.Frontend` | [#77](https://github.com/CarlosSV923/GameBook.Frontend/pull/77) | `feature/017-05/detail-modal` → `develop` | `MERGED` |
| GB-017.04 | `GameBook.Frontend` | [#76](https://github.com/CarlosSV923/GameBook.Frontend/pull/76) | `feature/017-04/index-strip-cards` → `develop` | `MERGED` |
| GB-017.03 | `GameBook.Frontend` | [#75](https://github.com/CarlosSV923/GameBook.Frontend/pull/75) | `feature/017-03/header-hero-filters` → `develop` | `MERGED` |
| GB-017.02 | `GameBook.Frontend` | [#74](https://github.com/CarlosSV923/GameBook.Frontend/pull/74) | `feature/017-02/tailwind-tokens-primitives` → `develop` | `MERGED` |
| GB-017.01 | `GameBook.Frontend` | [#73](https://github.com/CarlosSV923/GameBook.Frontend/pull/73) | `feature/017-01/archive-refined-system` → `develop` | `MERGED` |
| GB-016.06 | `GameBook.Frontend` | [#72](https://github.com/CarlosSV923/GameBook.Frontend/pull/72) | `feature/016-06/server-rendered-catalog` → `develop` | `MERGED` |
| GB-016.05 | `GameBook.Frontend` | [#71](https://github.com/CarlosSV923/GameBook.Frontend/pull/71) | `feature/016-05/tanstack-query-server-state` → `develop` | `MERGED` |
| GB-019.04 | `GameBook.Microservice.Game` | [#52](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/52) | `develop` → `main` (promoción) | `MERGED` |
| GB-016.04 | `GameBook.Frontend` | [#70](https://github.com/CarlosSV923/GameBook.Frontend/pull/70) | `feature/016-04/feature-modules-by-context` → `develop` | `MERGED` |
| GB-019.04 | `GameBook.Microservice.Game` | [#51](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/51) | `feature/019-04/composed-swagger-decorators` → `develop` | `MERGED` |
| GB-018.03 | `GameBook.Microservice.AuthUser` | [#49](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/49) | `develop` → `main` (promoción) | `MERGED` |
| GB-016.03 | `GameBook.Frontend` | [#69](https://github.com/CarlosSV923/GameBook.Frontend/pull/69) | `feature/016-03/rating-count-threshold` → `develop` | `MERGED` |
| GB-018.03 | `GameBook.Microservice.AuthUser` | [#48](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/48) | `feature/018-03/composition-root` → `develop` | `MERGED` |
| GB-019.03 | `GameBook.Microservice.Game` | [#50](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/50) | `feature/019-03/favorite-read-model` → `develop` | `MERGED` |
| GB-016.02 | `GameBook.Frontend` | [#68](https://github.com/CarlosSV923/GameBook.Frontend/pull/68) | `feature/016-02/split-igdb-adapter` → `develop` | `MERGED` |
| GB-016.01 | `GameBook.Frontend` | [#67](https://github.com/CarlosSV923/GameBook.Frontend/pull/67) | `feature/016-01/playwright-e2e-safety-net` → `develop` | `MERGED` |
| GB-019.02 | `GameBook.Microservice.Game` | [#49](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/49) | `feature/019-02/favorite-aggregate-mutations` → `develop` | `MERGED` |
| GB-018.02 | `GameBook.Microservice.AuthUser` | [#47](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/47) | `feature/018-02/bearer-session-guard` → `develop` | `MERGED` |
| GB-018.01 | `GameBook.Microservice.AuthUser` | [#46](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/pull/46) | `feature/018-01/user-aggregate-mutations` → `develop` | `MERGED` |
| GB-019.01 | `GameBook.Microservice.Game` | [#48](https://github.com/CarlosSV923/GameBook.Microservice.Game/pull/48) | `feature/019-01/framework-free-use-cases` → `develop` | `MERGED` |

## 7. Registro de coordinación

### Decisiones del autor (2026-10-24)

- 2026-10-27: tras revisar producción, el autor pide sustituir las fichas de la dirección A por las fichas póster de la opción B, conservando hero, filtros, colores, tipografía y el resto de A. Se abre GB-021 y GB-020.06 pasa a depender de GB-021.

- 2026-10-25: GB-017.07 sirve las fuentes con `next/font/local` (archivos en el repo) para que el build no dependa de Google Fonts.

- 2026-10-25: el texto de desactivación de cuenta del perfil dejará de mencionar el MVP en EN y ES; se incluye en GB-017.07.

- Corrección de favoritos (tras el hallazgo de GB-016.01): restaurar el comportamiento de `specs/mvp.md` §4.4 (elemento contextual sin sesión) y §4.5 (confirmación antes de guardar) como subtarea nueva del rediseño, `GB-017.08`; la especificación MVP no se modifica.

- SDD de la fase en archivos nuevos `*/improvements.md` en la raíz del workspace, sincronizados después a `GameBook.System`.
- Dirección visual A «Archive Refined» del canvas de propuestas.
- Ranking: mínimo de `total_rating_count`, aplicado solo a listados ordenados por `total_rating`; la búsqueda por nombre no lo aplica.
- Licencia MIT en los cuatro repositorios.
- Refactors incluidos: DDD de ambos backends, estado de servidor y adaptador IGDB del Frontend, Server Components y un único sistema de estilos, y E2E Playwright.
- Estilos con Tailwind CSS 4 y estado de servidor con TanStack Query.
- Diagramas: arquitectura y secuencia por repo de aplicación, con SVG EN/ES e índice de GitHub Pages.
- Detalles operativos de los README movidos a `docs/development.md`/`.es.md` en cada repositorio.
