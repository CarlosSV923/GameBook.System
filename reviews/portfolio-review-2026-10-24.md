# GameBook — Revisión de portfolio (2026-10-24)

Revisión de solo lectura de los cuatro repositorios (`origin/develop` de las aplicaciones y `main` de `GameBook.System`) y del despliegue productivo. No se modificó código. Este documento es la entrada para `specs/improvements.md`, `plan/improvements.md` y `tasks/improvements.md`.

Propuestas visuales: canvas «GameBook Redesign Proposals» (privado, no publicado).

## 1. Estética del Frontend

### Impresión general

La base es sólida (tokens claro/oscuro, tipografía serif editorial, foco visible, EN/ES), pero la implementación no cumple la dirección aprobada en `.interface-design/direction.md`: la «tira de índice», firma visual del sistema, no aparece en las tarjetas, y el catálogo es exactamente el patrón rechazado en ese documento («grid homogéneo de cards con imagen arriba»).

### Usabilidad

| Hallazgo | Severidad | Recomendación |
| --- | --- | --- |
| El ranking «más populares» muestra juegos casi desconocidos con 10.0 (p. ej. *Wortgewandt*, *Better Mart Simulator*): `sort total_rating desc` sin umbral de votos premia juegos con una o dos valoraciones. Es lo primero que ve un reclutador. | 🔴 Crítica | Exigir un mínimo de `total_rating_count` (requiere decisión de producto en la especificación). |
| Los botones primarios («Create account», «Search») usan gris pizarra con texto oscuro y se leen como deshabilitados. | 🔴 Crítica | Acción primaria con el acento de producto (cobre) y contraste ≥ 4.5:1. |
| El hero ocupa ~45 % del primer viewport a 1440×900; el catálogo empieza bajo el pliegue. | 🟡 Moderada | Hero compacto en una fila y filtros pegajosos. |
| Panel de filtros con etiqueta «Filters» duplicada y botón «Search» separado de los campos. | 🟡 Moderada | Barra de filtros de una fila, con búsqueda con icono y acción «Apply filters». |
| El fondo del modal en tema oscuro es gris claro lavado; resta contraste y foco al detalle. | 🟡 Moderada | Scrim oscuro semitransparente en ambos temas. |
| En móvil la navegación se apila en tres filas y «Catalog» queda debajo de las acciones de cuenta. | 🟡 Moderada | Cabecera de una fila con menú; filtros en hoja inferior. |
| Puntuación IGDB (0–100) se muestra como 10.0 sobre 10 sin indicar la escala. | 🟢 Menor | Mostrar 0–100 o explicar la escala una vez. |

### Jerarquía y consistencia

- Lo primero que capta la vista es el titular del hero; debería ser el catálogo/ranking.
- `globals.css` tiene 2 222 líneas en un solo archivo con clases BEM, mientras `tailwindcss` está importado pero no se usa: dos sistemas de estilos a medias.
- La tipografía serif de sistema (`Georgia`) se ve distinta según el sistema operativo; conviene cargar fuentes con `next/font`.

### Lo que funciona

- Tokens semánticos y temas claro/oscuro completos; preferencia persistida sin destello.
- Controles con foco visible, `aria-pressed`, áreas táctiles de 44 px y textos traducidos.

### Tres direcciones propuestas (ver canvas)

- **A — Archive Refined:** evoluciona la dirección actual y por fin implementa la tira de índice (rango, puntuación, año y plataformas en el lomo de cada ficha); hero compacto, filtros pegajosos, Newsreader + IBM Plex.
- **B — Spotlight Shelf:** oscuro y cinematográfico, con el juego n.º 1 destacado, «siguientes en el ranking», carteles 3:4 con puntuación y franja de llamada a crear cuenta; Space Grotesk + Inter, acento ámbar.
- **C — Data Index:** claro y técnico, panel de facetas, vista lista/tabla con barra de puntuación y atajo de búsqueda; Inter Tight + JetBrains Mono. Contradice el rechazo a la «sidebar permanente» de la dirección original, así que exige actualizarla.

## 2. Documentación y metadatos

| Hallazgo | Recomendación |
| --- | --- |
| Los README (117–125 líneas por app, 221 en System) repiten variables, políticas de release-please, Vercel y detalles de proceso SDD que no aportan a un reclutador. | Seguir la estructura de `VideoExplorer`: estado + badge CI, captura/GIF, «What it does», demo en vivo, capturas, desarrollo rápido, arquitectura con diagramas, tabla de stack y licencia. Mover los detalles operativos a `docs/` o a `GameBook.System`. |
| Los cuatro repos no tienen descripción, topics ni licencia; AuthUser apunta su homepage a un Vercel histórico (`gamebook-ms-authuser.vercel.app`). | Descripción en inglés, topics, homepage correcta (Vercel/Render/Pages) y licencia. |
| `architecture/` publica en cada repo recibos `visual-check` y capturas de cuatro tamaños (~3 MB por repo). | Conservar solo fuente JSON, HTML y la imagen usada por el README; exportar SVG como en VideoExplorer. |
| El skill Archify del workspace (`.agents/skills/archify`) está en 2.17; upstream y el skill de Claude están en 3.2. | Actualizar a 3.2 antes de regenerar diagramas. |
| Solo hay diagramas de arquitectura. | Añadir un diagrama de secuencia del flujo JWT/revocación (Frontend → Game → AuthUser) y uno de ciclo de vida de sesión; índice de GitHub Pages por repo. |
| El README de Frontend no muestra la app (sin capturas ni GIF). | Capturas EN generadas de forma reproducible (Playwright) y GIF del flujo principal. |

## 3. Arquitectura y DDD

### AuthUser

- ✅ Dominio puro (`User`, `EmailAddress`, `PasswordPolicy`), casos de uso sin NestJS y cableado por factorías.
- ⚠️ **Dominio anémico en mutaciones:** `User.changePasswordHash()` y `User.disable()` existen, pero los casos de uso llaman a `repository.replacePassword()`/`disable()`, que reimplementan la regla (incrementar `sessionVersion`) en Prisma. La invariante vive duplicada en dominio e infraestructura. Propuesta: cargar el agregado, invocar el método de dominio y `save(user)` con concurrencia optimista por versión.
- ⚠️ Autenticación dentro de los casos de uso: `ChangePasswordUseCase` y `DisableAccountUseCase` reciben el token y llaman a `ValidateSessionUseCase`; cada controlador parsea `Bearer` con su propia regex. Propuesta: guard + decorador `@CurrentSession()` en `api/` y casos de uso que reciban el `userId` autenticado.
- ⚠️ `ApplicationModule` importa `InfrastructureModule` y lee `ConfigService`; la composición debería vivir en un módulo raíz/composición, no en la capa de aplicación.
- Menor: `try/catch` que relanza sin cambios en `RegisterUserUseCase`; la sesión validada expone `passwordHash` a los casos de uso.

### Game

- ⚠️ **Estilos de DI inconsistentes entre servicios:** en Game los casos de uso son `@Injectable()` con `@Inject` (la capa de aplicación depende de NestJS), y el controlador inyecta clases concretas aunque existen interfaces de puerto sin usar.
- ⚠️ Mismo patrón anémico: `updateSnapshot` y `delete` resueltos en el repositorio en lugar de `favorite.updateSnapshot()` + `save`.
- ⚠️ Consultas (`list`, `suggest`) en el repositorio de dominio; un puerto de lectura en `application/` (CQRS ligero) separa mejor el modelo de escritura.
- `favorites.controller.ts` tiene 440 líneas, en su mayoría decoradores Swagger; extraer decoradores compuestos o mover la documentación a los DTO.
- Duplicación aceptable entre repos independientes (`api/http/*`), pero conviene documentar la decisión.

### Frontend

- ⚠️ Todo es Client Component: `app/page.tsx` es `"use client"` y el primer listado se pide desde el navegador; no se aprovechan Server Components/streaming ni SEO para la vista pública.
- ⚠️ `CatalogList` y `FavoritesList` (~400 líneas cada uno, ~15 `useState`) duplican paginación, scroll infinito, cancelación y reintentos. Extraer un hook de colección infinita o adoptar TanStack Query para estado de servidor.
- ⚠️ `src/server/igdb/igdb-client.ts` (609 líneas) mezcla construcción de consultas APICalypse, transporte y normalización; dividir en `query-builder`, `mapper` y `client`, con validación por esquema (p. ej. Zod) en vez de lectores manuales.
- Límites confusos: contratos/tipos en `shared/api`, clientes en `features/api`. Agrupar por contexto (`catalog`, `favorites`, `auth`) con `api/`, `model/`, `ui/`.
- Estilos: decidir entre Tailwind y CSS Modules por componente y eliminar el otro.
- Sin pruebas E2E de navegador (VideoExplorer usa Playwright, que además genera las capturas del README).

## 4. Proceso SDD

- Existen dos copias divergentes de `specs/`, `plan/` y `tasks/` (raíz del workspace y `GameBook.System`). Decisión del autor: la fase de mejoras usará archivos nuevos `*/improvements.md` en la raíz, sincronizados después a `GameBook.System` por PR.
- `tasks/mvp.md` mantiene `GB-014` como `[ ACTIVE ]` en la cabecera aunque todas sus subtareas figuran `[ RESOLVED ]`.
- Codex está configurado con `gpt-6-luna` (`~/.codex/config.toml`); las tareas se lanzarán con `codex exec` una vez aprobadas.
