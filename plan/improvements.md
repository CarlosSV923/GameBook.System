# GameBook — Plan de la fase de mejoras

## 1. Alcance y autoridad

Este plan organiza cómo implementar [`../specs/improvements.md`](../specs/improvements.md). Las subtareas, sus estados y dependencias están en [`../tasks/improvements.md`](../tasks/improvements.md), con las mismas reglas de estados, bloqueo y verificación de PR que [`../tasks/mvp.md`](../tasks/mvp.md) §1. La especificación manda sobre el plan. Si una decisión técnica exige cambiar el comportamiento, se detiene la subtarea y se pregunta al autor **una cuestión a la vez**.

## 2. Orden de trabajo

La regla rectora es **primero red de seguridad, luego refactor, luego rediseño, al final documentación**: los diagramas y README describen el código ya refactorizado, y las capturas muestran la interfaz ya rediseñada.

| Grupo | Repositorio | Depende de | Propósito |
| --- | --- | --- | --- |
| **GB-015** — Preparación | Workspace | `GB-014` | Aprobar la fase, actualizar Archify 3.2 y el contexto global. |
| **GB-016** — Arquitectura Frontend | Frontend | `GB-015.01` | E2E Playwright, división del adaptador IGDB, umbral del ranking, módulos por contexto, TanStack Query y Server Components. |
| **GB-017** — Rediseño A | Frontend | `GB-016` | Tailwind 4, `next/font`, tira de índice y vistas rediseñadas. |
| **GB-018** — DDD AuthUser | AuthUser | `GB-015.01` | Agregado con mutaciones de dominio, guard de sesión y composición. |
| **GB-019** — DDD Game | Game | `GB-015.01` | Casos de uso sin NestJS, mutaciones vía agregado y puerto de lectura. |
| **GB-020** — Documentación y publicación | Los cuatro | `GB-017`, `GB-018`, `GB-019` | Metadatos, licencia, diagramas Archify 3.2, README, `docs/` y sincronización SDD. |

**Paralelismo:** `GB-016`, `GB-018` y `GB-019` pueden avanzar a la vez, porque están en repositorios distintos. Dentro de un repositorio las subtareas son **secuenciales**, ya que comparten archivos. `GB-020.01` (metadatos y licencia) solo depende de `GB-015.01` y puede adelantarse. Los diagramas de `GB-020.02`–`.04` exigen además `GB-015.02` (Archify 3.2).

## 3. Estrategia por grupo

### 3.1 GB-015 — Preparación

- El autor aprueba `specs/plan/tasks/improvements.md`.
- Se actualiza `.agents/skills/archify` a la versión 3.2 de `tt-a1i/archify` y se registra su hash en `skills-lock.json`. Se comprueba que `node bin/archify.mjs --help` funcione.
- `AGENTS.md` y `CLAUDE.md` del workspace pasan a enlazar los documentos `improvements.md` como fase activa. `tasks/mvp.md` corrige la cabecera de `GB-014` a `[ RESOLVED ]`, ya que todas sus subtareas lo están.

### 3.2 GB-016 — Arquitectura Frontend

1. **E2E primero:** `@playwright/test` con proyectos `desktop` (1440×900) y `mobile` (375×812). La red se intercepta con `page.route` y fixtures ficticias para `/api/igdb/*`, AuthUser y Game. Los selectores son por rol y nombre accesible, para que sobrevivan al rediseño. CI añade un job `e2e`. Se cubren los flujos de spec §6.
2. **Adaptador IGDB:** `src/server/igdb/igdb-client.ts` se divide en `igdb-query-builder.ts` (APICalypse a partir de filtros permitidos), `igdb-mapper.ts` (validación con Zod y normalización a `IgdbGameCard`/`IgdbGameDetail`) e `igdb-client.ts` (transporte, token y limitador). Las pruebas unitarias actuales se reubican sin perder casos.
3. **Umbral del ranking:** constante `MIN_TOTAL_RATING_COUNT` en el query builder, aplicada solo cuando la consulta ordena por `total_rating` (sin término de nombre). El valor se elige comparando el top 50 real de IGDB con desarrollo local; se parte de 100–300 y se registran en el PR el valor y la evidencia. Se actualizan `contracts/igdb-mapping.md` y las pruebas del builder. La UI muestra la escala 0–100.
4. **Módulos por contexto:** `src/features/<contexto>/{api,model,ui}`, con `catalog`, `favorites`, `auth`, `profile`, `preferences` y `layout`. Los tipos de contrato pasan al `model` del contexto dueño; `shared/` conserva HTTP, healthcheck, i18n y primitivas UI. Es un movimiento sin cambios de lógica, con un commit propio.
5. **TanStack Query:** `QueryClientProvider` en el layout. Un `useInfiniteCollection` sobre `useInfiniteQuery` sirve a catálogo y favoritos; `useQuery` cubre detalle y sugerencias, y `useMutation` con `retry: 0` las mutaciones. Las query keys incluyen filtros y `userId`. Tras una mutación se invalida la consulta afectada, para cumplir el reinicio de paginación de `mvp.md`. Las IDs favoritas del catálogo se resuelven con una consulta propia. Se eliminan el estado duplicado y los `useState` de `CatalogList` y `FavoritesList`.
6. **Server Components:** `app/page.tsx` pasa a Server Component que precarga la primera página con el mismo query builder y la deshidrata (`HydrationBoundary`). Las partes interactivas quedan como Client Components hijos. Si la precarga falla, se cae al flujo cliente actual con su estado de error. Se verifica que `IGDB_CLIENT_SECRET` no aparezca en el bundle del cliente.

Cada paso es una subtarea con su PR hacia `develop` y CI verde: `test`, `lint`, `typecheck`, `build` y `e2e`.

### 3.3 GB-017 — Rediseño A

Se aplica `interface-design` en todas las subtareas. Las primeras subtareas no eliminan los estilos antiguos para no romper vistas aún no migradas; el retiro final es `GB-017.07`.

1. Actualizar `.interface-design/direction.md` y `system.md` con la dirección A: tokens, tipografías, tira de índice, composición, móvil y estados. Se toma como referencia visual el canvas.
2. Base: tokens en `@theme` de Tailwind 4 con variables claro/oscuro, `next/font` (Newsreader, IBM Plex Sans, IBM Plex Mono) y primitivas `Button`, `IconButton`, `TextField`, `SegmentedControl` e `IndexStrip`.
3. Cabecera, hero compacto y barra de filtros fija; en móvil, menú y panel de filtros.
4. Ficha con tira de índice (rango, puntuación 0–100, año, plataformas), en grid de escritorio y fichas horizontales en móvil; misma ficha en favoritos con la acción de eliminar.
5. Modal de detalle con scrim oscuro y ficha técnica; confirmaciones y tooltips.
6. Inicio de sesión, registro, perfil, estados de carga/vacío/error, avisos de éxito o error y alerta de arranque Render.
8. *(GB-017.08, añadida el 2026-10-24)* Restaurar `mvp.md` §4.4 y §4.5 en `FavoriteAction` (tarjeta y modal): popover accesible sin sesión con «Iniciar sesión»/«Crear cuenta» (foco gestionado, cierre con Escape y clic fuera, `aria-expanded`/`aria-controls`) y confirmación contextual previa al `useMutation` de guardado. Reutiliza el patrón de confirmación de eliminación existente. Se actualizan las aserciones E2E de GB-016.01, que documentaban el comportamiento desviado. Se ejecuta después de `.05` y antes de la revisión final `.07`.
7. Eliminar `globals.css` heredado salvo la base mínima, revisar contraste (≥ 4.5:1) y estados en claro/oscuro, EN/ES, 1440×900 y 375×812, y añadir `pnpm screenshots`: un script Playwright que genera `docs/screenshots/{en,es}/*.png` y un GIF del flujo principal a partir de los fixtures E2E.

Cada PR que cambie la UI adjunta capturas antes/después, como exige `mvp.md`.

### 3.3.1 GB-021 — Fichas póster (enmienda del 2026-10-27)

Ejecución: la coordinación implementa GB-021 sin Codex ni subagentes y usa `/design` como único skill de diseño (spec §3, enmienda de ejecución). No aplican aquí la §4 ni la puerta H de `interface-design`; la revisión visual se hace contra el canvas `/design` con capturas en ambos temas, idiomas y tamaños.

1. Actualizar `.interface-design/system.md` (GameCard, IndexStrip, composición) y `direction.md` con las fichas póster de la opción B. Reescribir `GameCard` para catálogo y favoritos: póster 3:4 con `next/image` y `object-cover` sobre un contenedor 3:4, de modo que la portada IGDB (3:4) no se recorte; etiqueta de puntuación; título; línea «año · plataformas» con `line-clamp`; acción de favoritos con los mismos nombres accesibles y el comportamiento de GB-017.08. Cuadrícula `auto-fill` (mínimo ~190 px en escritorio, 2 columnas en móvil) con filas de altura uniforme. Actualizar esqueletos, pruebas, E2E y `pnpm screenshots`.
2. Promoción `develop → main`, release-please y verificación en producción.

### 3.4 GB-018 — DDD AuthUser

1. `User.changePassword(newHash)` y `User.disable()` son la única fuente de la regla de `sessionVersion`. `UserRepository` reemplaza `replacePassword`/`disable` por `save(user, expectedVersion)`, que traduce un conflicto de versión a `SessionValidationError('SESSION_REVOKED')` (`updateMany` con `where` de versión). Se mantienen las pruebas de concurrencia y revocación.
2. `BearerSessionGuard` y `@CurrentSession()` en `src/api/auth/`. Los controladores dejan de parsear `Authorization`. `ChangePasswordUseCase` y `DisableAccountUseCase` reciben `{ userId, sessionVersion }` y cargan el usuario. `GET /v1/auth/session` conserva su contrato exacto, verificado con las pruebas de integración de Game.
3. `CompositionModule` (o `app.module.ts`) cablea application ↔ infrastructure. `ApplicationModule` deja de importar `InfrastructureModule` y de leer `ConfigService`; issuer y audience se inyectan como configuración tipada. Se limpian el `try/catch` redundante del registro y la exposición de `passwordHash`.

### 3.5 GB-019 — DDD Game

1. Casos de uso como clases sin decoradores Nest, cableadas por factorías y tokens de puerto (`CREATE_FAVORITE_USE_CASE`, …). Los controladores inyectan los puertos.
2. `Favorite.updateSnapshot()` y la eliminación vía agregado; `FavoriteRepository` queda reducido a `findByIdentity`, `save` y `delete`.
3. `FavoriteReadModel` en `application/ports/` con `list` y `suggest`, implementado en Prisma en `infrastructure/`.
4. Decoradores Swagger compuestos (`@ApiFavoriteErrors()`, …) para reducir `favorites.controller.ts`. OpenAPI generado idéntico, comparando `/docs/openapi.json` antes y después.

### 3.6 GB-020 — Documentación y publicación

1. **Metadatos y licencia:** `gh repo edit` con descripción, topics y homepage de spec §7.2, y `LICENSE` MIT por PR en cada repo.
2. **Diagramas** (uno por repo de aplicación y otro para `GameBook.System`): con Archify 3.2, arquitectura y secuencia EN/ES, `validate --quality showcase`, `deliver`, `visual-check` y revisión perceptual. Se exporta SVG para README y se publica un índice `architecture/index.html` en Pages. Se eliminan de `architecture/` los recibos y capturas `visual-check` (la evidencia se registra en la tarea) y el workflow de Pages se ajusta.
3. **README y docs** (uno por repo): estructura de spec §7.1, `docs/development.md` y `.es.md`, capturas desde `docs/screenshots/` (Frontend y System) y enlaces cruzados válidos.
4. **Sincronización SDD:** `specs/plan/tasks/improvements.md` y `reviews/` se copian a `GameBook.System` por PR hacia `main`, y se verifica la entrega final de los cuatro repos.

### 3.7 GB-022 — Ajustes posteriores al cierre (spec §10)

1. **Fin de favoritos (Frontend):** `CollectionPagination` recibe el contexto y lo pasa a `CatalogState`; mensajes nuevos EN/ES; pruebas unitaria y E2E.
2. **`main → develop` (las tres aplicaciones):** PR único de sincronización y un paso en `release-please.yml` que, cuando hay `release_created`, abre el PR `main → develop` con `gh` si no existe. Va antes del paso 3 para que `develop` tenga el `CHANGELOG.md` completo.
3. **Changelog (las tres aplicaciones):** `gh repo edit` con `merge_commit_message=BLANK` (cambio de configuración aprobado en spec §10) y limpieza de las entradas duplicadas de `CHANGELOG.md` por PR a `develop`.
4. **Canvas:** quitar el enlace en `specs/improvements.md` y en la revisión, y sincronizar con `GameBook.System`.
5. **Publicación:** promoción `develop → main` de las tres aplicaciones. La versión de Frontend (`fix`) sirve para comprobar el changelog sin duplicados y el PR automático `main → develop`.

## 4. Ejecución con Codex

La coordinación (Claude) asigna cada subtarea a Codex con el modelo `gpt-6-luna`, desde el repositorio dueño y con una sola subtarea por ejecución:

```bash
codex exec -m gpt-6-luna -C ../GameBook.Frontend --sandbox workspace-write \
  "Execute GameBook subtask GB-016.01 only. Read ../specs/improvements.md, ../plan/improvements.md §3.2 and ../tasks/improvements.md. ..."
```

- El prompt incluye: ID exacto, dependencias verificadas, archivos de propiedad, secciones de spec y plan aplicables, skill obligatorio (`interface-design` en GB-017, `archify` en GB-020.02–.05), comandos de calidad y definición de cierre.
- Codex crea la rama `feature/<nnn-nn>/<resumen>` desde `develop`, hace commits Conventional Commits en inglés y abre el PR con `gh`. **No fusiona.**
- Codex devuelve la nota de traspaso de `tasks/mvp.md` §6. La coordinación registra el PR, verifica CI y diff y, tras el aviso de merge del autor, marca `[ RESOLVED ]`.
- Los prompts y salidas de Codex no deben contener secretos. Las credenciales locales viven en `.env` ignorados.
- Grupos en paralelo: una ejecución Codex por repositorio a la vez, nunca dos sobre el mismo repo.

## 5. Puertas de calidad

- **Puerta F — Red de seguridad:** `GB-016.01` en verde antes de cualquier refactor del Frontend.
- **Puerta G — Comportamiento intacto:** tras cada refactor, las pruebas unitarias, de integración y E2E existentes pasan sin modificar aserciones de comportamiento. En los backends, el OpenAPI generado no cambia.
- **Puerta H — Visual:** revisión `interface-design` con capturas en ambos temas, idiomas y tamaños antes de fusionar cada subtarea de `GB-017`.
- **Puerta I — Documentación:** enlaces relativos y externos válidos, paridad EN/ES, Pages HTTP 200 y cero secretos antes de cerrar `GB-020`.
- Promoción `develop → main` y release-please por aplicación al cerrar cada grupo. Las versiones esperadas son *minor* para Frontend (cambio de ranking) y *patch/minor* para refactors según Conventional Commits.

## 6. Riesgos

- **RSC + TanStack Query + healthchecks:** la precarga en servidor solo toca IGDB, nunca AuthUser/Game, así que los healthchecks siguen siendo responsabilidad del cliente.
- **Umbral demasiado alto o bajo:** se valida con datos reales y es fácil de ajustar porque es una constante.
- **Rediseño grande en un repo:** se mitiga con subtareas pequeñas, E2E por rol y retiro de CSS al final.
- **Cambio de repositorio en AuthUser con concurrencia:** se conservan las pruebas de revocación concurrente y se añade una prueba de conflicto de versión.
- **Archify 3.2 con cambios de esquema:** las fuentes 2.17 se migran con las herramientas del propio skill (`migrations/`) o se regeneran; no se editan los HTML a mano.
