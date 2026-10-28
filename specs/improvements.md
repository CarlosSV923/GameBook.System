# GameBook — Especificación de la fase de mejoras

## 1. Propósito, autoridad y estado

Esta especificación define la fase posterior al MVP, que mejora la estética, la documentación y la arquitectura de los cuatro repositorios publicados de GameBook. Parte del diagnóstico de [`../reviews/portfolio-review-2026-10-24.md`](../reviews/portfolio-review-2026-10-24.md) y de las decisiones del autor del 2026-10-24.

[`mvp.md`](mvp.md) sigue vigente para todo comportamiento que este documento no modifica. Si ambos difieren, prevalece este documento solo en los puntos que modifica expresamente: el criterio del listado público (§2) y la dirección visual (§3). La estrategia está en [`../plan/improvements.md`](../plan/improvements.md) y el estado de las subtareas en [`../tasks/improvements.md`](../tasks/improvements.md). Este documento no autoriza a ejecutar subtareas cuyas dependencias no figuren como `[ RESOLVED ]`.

Fuera del §2, la fase **no añade funcionalidades ni cambia contratos HTTP**. Rutas, métodos, cuerpos, códigos de error, claims JWT, esquemas Prisma y migraciones de AuthUser y Game se conservan. Los refactors deben dejar intacto el comportamiento visible y pasar las pruebas existentes.

## 2. Cambio funcional: criterio de «mejor puntuados»

- Esta sección sustituye el criterio de §4.1 de `mvp.md`. El listado sigue ordenado por `total_rating` descendente (escala 0–100) y, además, excluye los juegos cuyo `total_rating_count` sea menor que un umbral mínimo de valoraciones.
- El umbral se aplica a todo listado público ordenado por `total_rating`: el listado inicial y los resultados filtrados solo por plataforma y/o año.
- La búsqueda por nombre, que sigue la relevancia de IGDB, **no** aplica el umbral, de modo que cualquier juego sigue pudiéndose encontrar. Las sugerencias de nombre y plataforma tampoco lo aplican.
- La vista de favoritos no se ve afectada, porque sus datos proceden de Game.
- El valor del umbral es una constante del servidor de Next.js. El plan lo fija tras contrastar resultados reales de IGDB y lo registra en `contracts/igdb-mapping.md`; no se expone como parámetro del navegador.
- La interfaz indicará de forma breve que el ranking usa la valoración combinada de IGDB. La puntuación se mostrará en escala 0–100.

## 3. Rediseño visual: dirección A «Archive Refined»

El autor eligió la dirección A del [canvas de propuestas](https://claude.ai/artifact/8pSb2nDSPc4WXptsj1qdWc). Evoluciona la identidad aprobada en `GB-006` («archivo de juegos / guía de campo») sin sustituirla. Se aplica con el skill `interface-design`.

- **Tira de índice obligatoria:** cada ficha del catálogo y de favoritos muestra una franja lateral con posición en el listado, puntuación, año y plataformas. La misma tira, a otra escala, aparece en el detalle y en los estados de carga, vacío y error.
- **Enmienda 2026-10-27 (decisión del autor tras revisar producción): fichas de la opción B.** Las fichas de catálogo y favoritos dejan de usar la tira de índice lateral y la portada 4:3, que generaban grandes espacios en blanco con listas largas de plataformas y recortaban los pósters. Pasan a ser pósters de proporción 3:4 sin recorte relevante, con la puntuación 0–100 en una etiqueta sobre la portada, el título (hasta dos líneas) y una línea «año · plataformas» truncada, más la acción de favoritos al lado del título, como en el artboard B «Spotlight Shelf» del canvas. Las filas de la cuadrícula tienen altura uniforme. Se mantienen sin cambios los colores, la tipografía, la cabecera, el hero, la barra de filtros, el modal de detalle (que conserva la tira en su variante de detalle) y el resto de la dirección A. La posición en el listado no se muestra en las fichas. Prevalece sobre el punto anterior en lo que respecta a las fichas.
- **Enmienda 2026-10-27 (ejecución de GB-021, decisión del autor):** la coordinación (Claude) implementa GB-021 directamente, sin Codex ni subagentes. El skill de diseño es `/design` (el canvas Design de las propuestas A/B); no se aplica `interface-design` ni otro skill de diseño. Prevalece sobre §8 y sobre el plan en lo que respecta a GB-021.
- **Composición del catálogo:** cabecera de una fila; hero compacto que no supera aproximadamente un tercio del primer viewport a 1440×900; barra de filtros de una fila que queda fija al desplazarse en escritorio; título de sección «Top rated of all time»; atribución IGDB visible.
- **Acciones:** la acción primaria usa el acento cobre con contraste ≥ 4.5:1 y no puede parecer deshabilitada. Las acciones secundarias son neutras.
- **Modal de detalle:** scrim oscuro semitransparente en ambos temas, portada, título y puntuación en primer plano, y metadatos como ficha técnica.
- **Móvil:** cabecera en una sola fila con menú, búsqueda visible, filtros en una hoja o panel desplegable y fichas horizontales con tira de índice.
- **Tipografía:** Newsreader para titulares, IBM Plex Sans para interfaz e IBM Plex Mono para cifras, cargadas con `next/font` y con fallback de sistema.
- **Alcance:** catálogo, detalle, favoritos, inicio de sesión, registro, perfil, confirmaciones, avisos y alerta de arranque Render, en claro/oscuro, EN/ES, escritorio y móvil.
- Se conservan los requisitos de §3.2, §3.3, §4 y §6 de `mvp.md`: tema y persistencia, idioma y persistencia, filtros con sugerencias, desplazamiento infinito, tooltips sin sesión, confirmaciones y textos IGDB sin traducir.
- **Corrección obligatoria:** la producción actual no cumple `mvp.md` §4.4 (guardar sin sesión navega a inicio de sesión en lugar de mostrar el elemento contextual con «Iniciar sesión» y «Crear cuenta») ni §4.5 (guardar con sesión no pide confirmación). El rediseño debe restaurar ambos comportamientos tal como los define `mvp.md`, en tarjeta y modal.
- `.interface-design/direction.md` y `system.md` se actualizan para reflejar la dirección A como sistema vigente.

## 4. Arquitectura del Frontend

- **Estilos:** Tailwind CSS 4 es el único sistema de estilos. Los tokens del tema se declaran en `@theme` y las variantes claro/oscuro se mantienen mediante variables. Se elimina el `globals.css` monolítico; solo quedan estilos base globales mínimos.
- **Estado de servidor:** TanStack Query gestiona catálogo, favoritos, detalle y sugerencias. Catálogo y favoritos comparten un mismo patrón de colección infinita (`useInfiniteQuery`) en lugar de duplicar paginación, cancelación y reintentos.
  - Las mutaciones (guardar, eliminar, sincronizar instantánea, cambiar contraseña, desactivar cuenta) nunca se reintentan automáticamente.
  - Se conservan los healthchecks de §5.4 de `mvp.md` y el transporte Axios + RxJS de §5.5; TanStack Query consume las Promises existentes.
- **Server Components:** la primera página del catálogo público se obtiene en el servidor de Next.js. El HTML inicial ya contiene los primeros resultados, que se hidratan en la caché de TanStack Query. La interactividad, la sesión y las preferencias siguen en el cliente. Las credenciales IGDB/Twitch siguen siendo server-only.
- **Integración IGDB:** el adaptador servidor se divide en un constructor de consultas APICalypse, un mapeador validado por esquema y un cliente de transporte. La validación usa un esquema declarativo en lugar de lectores manuales.
- **Módulos:** el código se organiza por contexto (`catalog`, `favorites`, `auth`, `profile`, `preferences`), con subcarpetas de API, modelo y UI. `shared/` queda solo para primitivas realmente transversales.
- Ningún archivo de producción debería superar aproximadamente 300 líneas sin una justificación registrada en su PR.

## 5. Arquitectura DDD de los backends

Sin cambios de contrato, esquema ni comportamiento.

### 5.1 AuthUser

- Las mutaciones del agregado `User` (cambio de contraseña y desactivación) se expresan en métodos de dominio. El caso de uso carga el agregado, invoca el método y persiste con `save` y concurrencia optimista por `sessionVersion`. La regla de incrementar la versión deja de duplicarse en infraestructura.
- La autenticación Bearer se resuelve en la capa `api/` con un guard y un decorador de sesión actual compartidos. Los casos de uso protegidos reciben la identidad autenticada, no el token crudo.
- `GET /v1/auth/session` conserva exactamente su contrato, porque Game depende de él.
- La composición de dependencias vive en un módulo de composición. La capa de aplicación no importa módulos de infraestructura ni lee configuración directamente.
- La sesión validada que reciben los casos de uso no expone el hash de contraseña salvo donde el caso de uso lo necesite explícitamente.

### 5.2 Game

- Los casos de uso son clases sin dependencias de NestJS, cableadas por factorías como en AuthUser. Los controladores dependen de los puertos de caso de uso, no de clases concretas.
- La actualización de instantánea y la eliminación pasan por el agregado `Favorite` y por `save`/`delete` del repositorio de escritura.
- Listado filtrado y sugerencias usan un puerto de lectura definido en `application/`. El repositorio de dominio queda para el modelo de escritura.
- La documentación Swagger del controlador se compone con decoradores reutilizables, de modo que el controlador refleje solo el flujo HTTP.

### 5.3 Común

- Ambos servicios mantienen la estructura `src/api`, `src/application`, `src/domain`, `src/infrastructure` y `tests/unit`, `tests/integration`.
- La duplicación intencional de utilidades HTTP entre repositorios independientes se documenta como decisión.

## 6. Pruebas E2E del Frontend

- El Frontend incorpora pruebas E2E con Playwright para el flujo visitante y el flujo autenticado: catálogo, filtros, detalle, intento de guardar sin sesión, registro, inicio de sesión, guardar, favoritos, eliminar, cambio de idioma y de tema.
- En CI, las pruebas interceptan las llamadas de red con datos ficticios y no consumen IGDB/Twitch ni servicios reales. Localmente pueden ejecutarse además contra el Compose de `GameBook.System`.
- Cubren escritorio y móvil, y deben existir **antes** de los refactors del Frontend para servir de red de seguridad.
- Un script reproducible basado en esas pruebas genera las capturas y el GIF que usan los README.

## 7. Documentación, metadatos y licencia

### 7.1 README

- Los README de las tres aplicaciones y de `GameBook.System` siguen la estructura del repositorio de referencia [`CarlosSV923/VideoExplorer`](https://github.com/CarlosSV923/VideoExplorer):
  1. Título, enlace al otro idioma, estado y badge de CI.
  2. Captura o GIF principal.
  3. Párrafo de propósito y «What it does».
  4. Demo en vivo o Swagger.
  5. Capturas (solo Frontend y `GameBook.System`).
  6. Desarrollo rápido.
  7. Arquitectura con diagramas.
  8. Tabla de stack y decisiones.
  9. Proyectos relacionados y licencia.
- Los detalles operativos pasan a `docs/development.md` (inglés) y `docs/development.es.md` (español) en cada repositorio de aplicación: variables completas, migraciones, release-please, despliegue Render/Vercel, Docker y comandos de calidad. El README los enlaza. Se eliminan las referencias a políticas históricas (Vercel Preview, proyectos eliminados).
- Se mantiene la paridad EN/ES: `README.md` en inglés y `README.es.md` en español, completos y equivalentes.
- **Enmienda 2026-10-28 (decisión del autor): GIF principal.** El GIF del README del Frontend y de `GameBook.System` recorre el flujo completo: catálogo, detalle, aviso de guardado sin sesión, inicio de sesión, confirmación de guardado y favoritos. Cada paso se mantiene en pantalla al menos 2 s, se captura cuando terminan las animaciones (el modal ya abierto) y cada fotograma usa su propia paleta de color. Lo implementa la coordinación sin Codex, como los demás ajustes del Frontend en esta fase.
- El README de `GameBook.System` también se reduce. Conserva visión, enlaces a los tres repositorios, arquitectura, Compose local resumido y trazabilidad SDD.

### 7.2 Metadatos de GitHub

Se configuran descripción en inglés, topics en inglés y homepage. Valores propuestos, ajustables por el autor en la revisión del PR o de la tarea:

| Repositorio | Descripción | Topics | Homepage |
| --- | --- | --- | --- |
| `GameBook.Frontend` | Next.js frontend for GameBook: browse top-rated games from IGDB, sign in and keep a personal favorites shelf. EN/ES, light/dark. | `nextjs`, `react`, `typescript`, `tailwindcss`, `tanstack-query`, `igdb`, `rxjs`, `playwright`, `vercel`, `portfolio` | `https://gamebook-frontend.vercel.app` |
| `GameBook.Microservice.AuthUser` | NestJS authentication microservice for GameBook: registration, RS256 JWT sessions with instant revocation, Prisma and PostgreSQL, DDD. | `nestjs`, `typescript`, `prisma`, `postgresql`, `jwt`, `authentication`, `domain-driven-design`, `microservice`, `swagger`, `portfolio` | Swagger en Render (`/docs`) |
| `GameBook.Microservice.Game` | NestJS favorites microservice for GameBook: per-user favorite games with filters and pagination, JWT-protected, Prisma and PostgreSQL, DDD. | `nestjs`, `typescript`, `prisma`, `postgresql`, `jwt`, `domain-driven-design`, `microservice`, `swagger`, `igdb`, `portfolio` | Swagger en Render (`/docs`) |
| `GameBook.System` | Documentation hub for GameBook: architecture diagrams, spec-driven development records and a local Docker Compose for the three services. | `documentation`, `architecture`, `spec-driven-development`, `docker-compose`, `archify`, `microservices`, `portfolio` | Índice de GitHub Pages |

La homepage histórica de AuthUser (`gamebook-ms-authuser.vercel.app`) se reemplaza.

### 7.3 Licencia

Los cuatro repositorios incluyen `LICENSE` MIT con el titular «Carlos Sesme Vera» y lo indican al final de ambos README.

### 7.4 Diagramas Archify

- Se usa Archify **3.2**. La copia del workspace (`.agents/skills/archify`) se actualiza antes de generar.
- Cada repositorio de aplicación tiene:
  - el diagrama de arquitectura regenerado desde el código resultante de esta fase;
  - un diagrama de secuencia de su flujo principal: catálogo público IGDB/Twitch en Frontend, inicio de sesión y revocación en AuthUser, operación protegida de favoritos con validación de sesión en Game.
- `GameBook.System` regenera el diagrama general del sistema.
- Cada diagrama tiene variantes EN/ES con la misma topología, HTML interactivo y exportación SVG. El README muestra el SVG de su idioma y enlaza su HTML publicado en GitHub Pages. Cada repo publica un índice de Pages con sus diagramas.
- `architecture/` conserva solo fuentes JSON, HTML y SVG publicados. Los recibos y capturas de `visual-check` se registran como evidencia de la tarea y no se versionan en los repositorios públicos.

## 8. Restricciones

- Sin secretos en código, documentación, capturas, fixtures ni prompts de Codex.
- Flujo Git de `mvp.md` §8: ramas `feature/[task-number]/[resumen]` desde `develop`, PR hacia `develop`, promoción `develop → main` y release-please en las aplicaciones. `GameBook.System` usa PR hacia `main`.
- El autor fusiona personalmente los PR; la coordinación verifica el merge y el entregable antes de marcar `[ RESOLVED ]`.
- Las tareas se ejecutan con Codex (modelo `gpt-6-luna`) según el plan, salvo GB-021, que implementa la coordinación (enmienda de §3). La coordinación verifica los resultados.

## 9. Criterios de aceptación

1. El listado inicial y los filtrados solo por plataforma/año no muestran juegos por debajo del umbral de valoraciones. La búsqueda por nombre sigue encontrando juegos sin ese mínimo.
2. Sin sesión, la acción de favoritos muestra el elemento contextual de `mvp.md` §4.4 y no navega ni guarda; con sesión, guardar exige la confirmación de §4.5, en tarjeta y modal.
3. Catálogo, detalle, favoritos, autenticación y perfil siguen la dirección A (fichas póster 3:4 de la opción B según la enmienda de §3, sin huecos verticales ni pósters recortados; tira de índice en el detalle), en claro/oscuro, EN/ES, escritorio y móvil, sin pérdida de funcionalidad de `mvp.md`.
4. La acción primaria cumple contraste ≥ 4.5:1 en ambos temas y el modal usa scrim oscuro.
5. El Frontend usa solo Tailwind CSS 4, TanStack Query para estado de servidor y Server Components para la primera página pública. Se conservan healthchecks, Axios + RxJS y la ausencia de reintentos de mutaciones.
6. El adaptador IGDB queda dividido en consulta, mapeo validado y transporte, y las credenciales siguen siendo server-only.
7. AuthUser y Game cumplen §5 sin cambiar OpenAPI, esquemas ni migraciones, con todas sus pruebas en verde.
8. Las pruebas E2E Playwright pasan en CI con red simulada, en escritorio y móvil.
9. Los README EN/ES de los cuatro repositorios siguen §7.1 y los detalles operativos están en `docs/development*.md`.
10. Los cuatro repositorios tienen descripción, topics, homepage correcta y licencia MIT.
11. Los diagramas Archify 3.2 de §7.4 están publicados en Pages, con SVG EN/ES en los README y sin recibos `visual-check` versionados.
12. `specs/`, `plan/` y `tasks/` de esta fase están sincronizados en `GameBook.System`.
