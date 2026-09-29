# GameBook — Contexto del proyecto

## Propósito

GameBook es un sistema web pequeño para el portfolio personal de su autor. Su público objetivo son reclutadores del sector tecnológico, por lo que comunica una solución clara y profesional: sencilla de entender, funcional y con una arquitectura moderna.

La aplicación permite consultar información sobre videojuegos favoritos, incluyendo datos como plataformas o consolas de salida, años de lanzamiento y otros detalles disponibles desde IGDB. Las personas usuarias pueden crear una cuenta, iniciar sesión y guardar sus juegos favoritos para consultarlos posteriormente.

## Estado y metodología de trabajo

Este proyecto sigue **Spec Driven Development (SDD)**. Ejecutar solo la subtarea asignada y habilitada por sus dependencias; este contexto general no autoriza a implementar otras funcionalidades ni a anticipar tareas.

Antes de implementar cualquier funcionalidad, la especificación correspondiente deberá estar documentada y aprobada en `./specs/mvp.md`. Ese archivo contiene los requisitos del MVP. `./plan/mvp.md` define la estrategia y `./tasks/mvp.md` registra subtareas ejecutables, estados y dependencias. **No iniciar una subtarea si cualquiera de sus dependencias no figura como `[ RESOLVED ]`**. Este archivo ofrece únicamente el contexto general; en caso de diferencia, prevalece `./specs/mvp.md`.

Al trabajar en el proyecto:

- No inventar requisitos de producto que no estén en las especificaciones.
- Mantener las decisiones técnicas alineadas con este contexto y con las futuras especificaciones.
- Priorizar una experiencia limpia, accesible y demostrable para reclutadores de tecnología.
- Mantener los servicios desacoplados y las responsabilidades claramente delimitadas.

## Arquitectura implementada

El sistema está compuesto por los siguientes elementos:

1. `GameBook.Microservice.AuthUser`: microservicio de usuarios y autenticación, implementado con NestJS y Prisma.
2. `GameBook.Microservice.Game`: microservicio de gestión y almacenamiento de juegos, implementado con NestJS. Se conectará a PostgreSQL mediante Prisma.
3. `GameBook.Frontend`: aplicación frontend implementada con Next.js. Consumirá ambos microservicios y consultará IGDB desde su servidor para proteger las credenciales OAuth de Twitch.
4. Una base de datos PostgreSQL compartida por ambos microservicios, con un esquema propio y migraciones Prisma independientes por servicio. Las conexiones runtime usan roles SQL limitados por esquema; las migraciones usan credenciales distintas y Actions controlados.
5. API externa de IGDB para obtener información de videojuegos: <https://api-docs.igdb.com/>. Su acceso se realiza con Client ID y token OAuth de aplicación emitido por Twitch; el uso no comercial es gratuito según la documentación oficial.

El contrato **vigente** para implementación es [`contracts/contract-baseline-v3.md`](./contracts/contract-baseline-v3.md). Los contratos v1/v2 y sus archivos RAWG son evidencia histórica; no deben usarse para implementar Game ni Frontend. La v3 cambia identidad externa a `igdbId` y puntuación a `total_rating` (0–100), sin alterar los contratos de AuthUser ni la enmienda de aislamiento Neon.

Ambos microservicios documentarán sus APIs con Swagger/OpenAPI y expondrán Swagger UI para consulta en los despliegues del portfolio.

## Convenciones y decisiones de la implementación

- Los tres repositorios de aplicaciones son públicos en GitHub e independientes; cada uno usa `pnpm` y versiona su propio `pnpm-lock.yaml`.
- Los backends partirán de `src/` y `tests/` en la raíz. Dentro de `src/`: `api/`, `application/`, `domain/`, `infrastructure/` y `main.ts`. Las pruebas unitarias e integraciones vivirán en `tests/unit/` y `tests/integration/`; Prisma podrá mantener esquema y migraciones en `prisma/`.
- Para persistencia, aplicar la v3 vigente y la [enmienda de acceso v2](./contracts/persistence-access-amendment-v2.md): los roles runtime `gamebook_auth_app` y `gamebook_game_app` no acceden al esquema ajeno. `GB-003.06` ya probó migradores SQL limitados; los antiguos migradores Neon con privilegios amplios **no deben usarse en Actions**. `GB-004.07` y `GB-005.07` implementarán los Actions de cada backend. No incluir URLs de migración en Render, Vercel o Compose ni ejecutar migraciones al construir o arrancar los servicios.
- `GB-003.03`–`GB-003.05` son cierres **históricos** de la preparación Vercel inicial. No asumir que esos proyectos o secretos existen ni recuperarlos. La [política vigente local-first](./environments/deployment-policy-local-first.md) sustituye el calendario histórico: aplicaciones y Compose se prueban localmente con Neon `develop` e IGDB/Twitch de pruebas; AuthUser y Game se publican en Render desde `main` tras la aceptación local, y Frontend se publica en Vercel desde `main` si corresponde. No crear Vercel Preview para los backends.
- La creación/publicación inicial de repositorios y la gestión de PR se harán con GitHub CLI (`gh`); Git se usará para commits y push posteriores.
- `main` y `develop` de los tres repositorios de aplicación no tendrán protecciones de rama, aprobaciones obligatorias ni checks obligatorios de GitHub. Se mantienen los PR y el CI como prácticas de calidad. Registrar cada PR en `./tasks/mvp.md`; el autor lo fusionará o cerrará personalmente. Tras su aviso, verificar en GitHub el merge y la rama de destino, además del entregable, antes de marcar la subtarea `[ RESOLVED ]`. Un PR abierto o cerrado sin merge no basta.
- Los repositorios de aplicaciones ya completaron la integración y publicación de su MVP. La implementación final se verifica en sus ramas `main`; AuthUser y Game se ejecutan en Render y Frontend en Vercel.
- Las tareas de diseño y revisión visual del frontend utilizaron `interface-design`. La documentación de la arquitectura realmente implementada se generó y validó con `archify`; sus fuentes y HTML se conservan en `./architecture/`.
- Los tres repositorios de aplicaciones tienen `README.md` en inglés y `README.es.md` en español. `GameBook.System` es el cuarto repositorio público **solo documental**, con rama `main` y una publicación inicial única. La documentación general nueva tiene versiones EN/ES; los documentos SDD, contratos, ambientes, `AGENTS.md` y `CLAUDE.md` ya generados en español se conservan en español por decisión de GB-014.03.

## Comportamiento implementado

- La consulta pública de información de juegos no requerirá cuenta ni JWT de GameBook; el servidor de Next.js autenticará sus propias llamadas a IGDB con un token OAuth de aplicación.
- Guardar y consultar juegos favoritos propios requerirá crear una cuenta e iniciar sesión.
- El frontend consumirá los microservicios para las acciones propias de GameBook y IGDB para la información externa de videojuegos.
- El frontend permite elegir modo claro u oscuro y cambiar sus textos de interfaz entre inglés y español. El idioma inicial es inglés y la elección manual se recuerda; los datos recibidos de IGDB no se traducen en el MVP.
- El microservicio de juegos es responsable de la persistencia de los juegos favoritos asociados a las cuentas.

## Autenticación y seguridad

- La autenticación utiliza JWT.
- AuthUser firmará los JWT con una clave privada de desarrollo cargada de forma privada en local y otra de producción configurada como variable de entorno en Render; Game verificará con la clave pública correspondiente a cada ambiente. La clave privada nunca se incluirá en código fuente, documentación pública ni control de versiones.
- Tras el inicio de sesión, el frontend almacenará el token en `sessionStorage`; el JWT tendrá una vigencia de una hora.
- Para las peticiones autenticadas, el frontend enviará el token JWT en cada solicitud a los microservicios que lo requieran.
- El frontend enviará a Game el mismo JWT emitido por AuthUser mediante `Authorization: Bearer <token>`. Game verificará firma y vencimiento con la clave pública y obtendrá del token validado el UUID del usuario para acceder a sus favoritos.
- Si una persona no ha creado una cuenta ni iniciado sesión, no será necesaria autenticación de GameBook: podrá usar las consultas públicas del frontend, que accederá a IGDB desde el servidor con credenciales propias no expuestas.

## Despliegue implementado

- Frontend y microservicios: el MVP se ejecuta y prueba localmente; AuthUser y Game están desplegados en Render desde `main`. Frontend está desplegado en Vercel desde `main`; no hay Vercel Preview para los backends. Los proyectos Vercel históricos no son destino de los microservicios.
- Base de datos PostgreSQL: Neon `develop` para pruebas y `production` para el despliegue, con credenciales de entorno separadas; también se separan las credenciales IGDB/Twitch de prueba y producción.

El listado público muestra los juegos mejor puntuados según `total_rating` de IGDB. Los requisitos acordados están en `./specs/mvp.md` y la estrategia de ejecución en `./plan/mvp.md`. El estado actual de ejecución está exclusivamente en `./tasks/mvp.md`.
