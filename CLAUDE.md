# GameBook — Guía de contexto para asistentes de IA

## Visión del producto

GameBook es un sistema web pequeño concebido para un portfolio personal y dirigido a reclutadores de tecnología. Debe ser una demostración sencilla, funcional y profesional de desarrollo full stack.

La aplicación permite explorar información de videojuegos favoritos, como consolas o plataformas de lanzamiento, años de lanzamiento y otros detalles que proporciona IGDB. También permite crear una cuenta para guardar juegos favoritos y recuperarlos más adelante.

## Regla principal: desarrollo guiado por especificaciones

El proyecto utiliza **Spec Driven Development (SDD)**. Ejecutar solo la subtarea asignada y habilitada por sus dependencias; este contexto general no autoriza a implementar otras funcionalidades ni a anticipar tareas.

La especificación del producto se encuentra en `./specs/mvp.md` y es la fuente de verdad del MVP. `./plan/mvp.md` establece la estrategia; `./tasks/mvp.md` registra subtareas ejecutables, estados y dependencias. **No ejecutar una subtarea si alguna dependencia no está `[ RESOLVED ]`**. Este archivo y `AGENTS.md` establecen el contexto global de GameBook.

No asumir requisitos que no estén documentados. Si alguna decisión es necesaria para implementar, deberá especificarse primero en `./specs/mvp.md`.

## Componentes implementados

| Componente | Tecnología | Responsabilidad |
| --- | --- | --- |
| `GameBook.Microservice.AuthUser` | NestJS + Prisma | Gestión de usuarios y autenticación. |
| `GameBook.Microservice.Game` | NestJS + Prisma | Gestión y persistencia de juegos favoritos en PostgreSQL. |
| `GameBook.Frontend` | Next.js | Interfaz amigable; consume ambos microservicios y consulta IGDB desde el servidor de Next.js para proteger las credenciales OAuth de Twitch. |
| Base de datos | PostgreSQL | Una base compartida por ambos microservicios, con esquema y migraciones independientes por servicio. |
| Servicio externo | IGDB API | Información externa de videojuegos. Documentación: <https://api-docs.igdb.com/>. |

Los dos microservicios documentarán sus APIs con Swagger/OpenAPI y ofrecerán Swagger UI en los despliegues del portfolio.

## Convenciones y decisiones de la implementación

- Los tres repositorios de aplicaciones son públicos e independientes en GitHub y utilizan `pnpm` con un `pnpm-lock.yaml` propio.
- Ambos backends seguirán `src/` y `tests/` en la raíz; `src/` incluirá `api/`, `application/`, `domain/`, `infrastructure/` y `main.ts`. `tests/unit/` y `tests/integration/` contendrán las pruebas; `prisma/` podrá alojar esquema y migraciones.
- Para Neon/Prisma rige la v3 vigente junto con la [enmienda de acceso v2](./contracts/persistence-access-amendment-v2.md): runtime con roles SQL limitados `gamebook_auth_app`/`gamebook_game_app`. `GB-003.06` ya probó migradores SQL limitados; los migradores antiguos con privilegios amplios no se entregan a Actions. `GB-004.07`/`GB-005.07` implementarán los Actions por backend y ambiente. Sus URLs nunca se configuran en Render, Vercel ni Compose.
- `GB-003.03`–`GB-003.05` registran preparación Vercel **histórica**. No presuponer que esos proyectos o variables continúan disponibles. Según la [política local-first vigente](./environments/deployment-policy-local-first.md), las aplicaciones se prueban localmente con Neon `develop` e IGDB/Twitch de pruebas; AuthUser y Game se publican en Render desde `main` tras la aceptación local y Frontend puede publicarse en Vercel desde `main`. Los valores productivos se configuran solo después de la aceptación local, en `GB-012`, sin placeholders.
- Crear/publicar inicialmente los repositorios y gestionar los PR mediante GitHub CLI (`gh`); usar Git para commits y push posteriores.
- `main` y `develop` de las tres aplicaciones permanecerán sin protecciones de rama, aprobaciones obligatorias ni checks obligatorios de GitHub. Los PR y el CI siguen siendo prácticas de calidad. Registrar cada PR en `./tasks/mvp.md`; solo el autor lo fusionará o cerrará. Tras su aviso, verificar en GitHub que todos los PR de la subtarea estén fusionados en su destino y comprobar el entregable antes de marcar `[ RESOLVED ]`. Un PR abierto o cerrado sin merge no basta.
- Los repositorios de aplicaciones ya completaron la integración y publicación de su MVP. La implementación final se verifica en sus ramas `main`; AuthUser y Game se ejecutan en Render y Frontend en Vercel.
- Las tareas de UI del frontend utilizaron `interface-design`. `archify` generó y validó la documentación de la arquitectura realmente implementada; sus fuentes y HTML están en `./architecture/`.
- Cada aplicación tiene `README.md` en inglés y `README.es.md` en español. `GameBook.System` es un repositorio público exclusivamente documental, con solo `main` y una publicación inicial única. La documentación general nueva tiene pares EN/ES; los archivos SDD, contratos, ambientes, `AGENTS.md` y `CLAUDE.md` ya generados en español se conservan en español por decisión de GB-014.03. Su README principal en inglés enlaza los tres proyectos.

## Flujos funcionales de alto nivel

- Sin sesión: se podrá consultar información pública de videojuegos; solo el servidor Next.js usa Client ID y token OAuth de aplicación para acceder a IGDB.
- Con sesión: una persona podrá registrar una cuenta, iniciar sesión y guardar sus juegos favoritos.
- El frontend se comunica con los microservicios para las funciones propias de GameBook y con IGDB para los datos externos de videojuegos.
- El frontend permite alternar modo claro y oscuro y cambiar sus textos de interfaz entre inglés y español. El idioma inicial es inglés y la elección manual se recuerda; los datos recibidos de IGDB no se traducen en el MVP.
- La persistencia de favoritos corresponde al microservicio `GameBook.Microservice.Game` y PostgreSQL.

## Autenticación

- Se usa JWT.
- AuthUser firmará los JWT con una clave privada local de desarrollo y otra de producción configurada en Render; Game verificará con la clave pública correspondiente a cada ambiente. La clave privada jamás se versionará ni expondrá al cliente.
- Tras iniciar sesión, el frontend guardará el token en `sessionStorage`; el JWT tendrá una vigencia de una hora.
- El frontend enviará el JWT en toda petición que requiera autenticación.
- Para las operaciones de favoritos, el frontend enviará directamente a Game el JWT de usuario emitido por AuthUser en `Authorization: Bearer <token>`. Game verificará firma y vencimiento con la clave pública y usará el UUID del token validado para delimitar los favoritos del usuario.
- La consulta pública no exigirá cuenta ni JWT.

## Despliegue implementado

- Frontend y microservicios se ejecutan y prueban localmente; AuthUser y Game están publicados en Render desde `main`, y Frontend en Vercel desde `main`. No hay Vercel Preview para los backends.
- Neon mantiene `develop` para pruebas locales y `production` para el despliegue, con credenciales separadas; IGDB/Twitch también tendrá credenciales separadas para desarrollo/pruebas y producción.

El listado público inicial mostrará los juegos mejor puntuados según `total_rating` de IGDB. El contrato vigente de implementación es [`contracts/contract-baseline-v3.md`](./contracts/contract-baseline-v3.md); las versiones RAWG anteriores son históricas. Los requisitos del MVP están en `./specs/mvp.md` y el plan de implementación en `./plan/mvp.md`; el estado actual de las tareas está en `./tasks/mvp.md`.
