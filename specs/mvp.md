# GameBook — Especificación técnica y funcional del MVP

## 1. Propósito y estado del documento

GameBook es un sistema web pequeño para un portfolio personal. Su público principal son reclutadores de tecnología. La experiencia debe ser sencilla, funcional, amigable y fácil de evaluar.

Este documento describe el comportamiento y las restricciones técnicas del MVP. Es la fuente de verdad para [`../plan/mvp.md`](../plan/mvp.md). Los tres repositorios de aplicación ya se crearon, pero **esta revisión documental no autoriza a generar código** ni a ejecutar subtareas con dependencias pendientes. La implementación avanzará solo según el estado de [`../tasks/mvp.md`](../tasks/mvp.md).

Las decisiones funcionales del MVP se encuentran en este documento. El plan de implementación concreta contratos y estructura técnica sin alterar el comportamiento aquí definido. Si para planificar o implementar hace falta una decisión que cambie el producto, se deberá preguntar al autor e incorporar primero su respuesta a esta especificación.

## 2. Alcance funcional

Una persona visitante podrá explorar juegos y consultar sus detalles con datos de la API de IGDB, sin crear una cuenta. Una persona con cuenta podrá iniciar sesión, guardar juegos como favoritos, consultar y filtrar su lista personal, eliminar favoritos y cambiar su contraseña.

Solo habrá tres criterios de filtro en el MVP: **nombre del juego, plataforma o consola, y año de lanzamiento**. Podrán aplicarse a la vista pública y a la lista de favoritos. No se han solicitado otros filtros.

El frontend permitirá elegir entre **modo claro y modo oscuro** y cambiar los textos propios de GameBook entre **inglés y español**. El idioma inicial de la interfaz será **inglés**. IGDB seguirá siendo la fuente externa de videojuegos.

## 3. Arquitectura y responsabilidades

| Proyecto / componente | Tecnología indicada | Responsabilidad |
| --- | --- | --- |
| `GameBook.Microservice.AuthUser` | NestJS, Prisma y PostgreSQL | Cuentas de usuario, inicio de sesión, emisión y validación de la autenticación JWT, y cambio de contraseña. |
| `GameBook.Microservice.Game` | NestJS, Prisma y PostgreSQL | Gestión y persistencia de los juegos favoritos asociados a cada usuario. |
| `GameBook.Frontend` | Next.js | Vistas públicas y autenticadas, interacción con los dos microservicios y consulta de datos de IGDB mediante código del servidor de Next.js. |
| IGDB API | Servicio externo | Catálogo, búsqueda, plataformas, imágenes y detalles de juegos; autenticación de aplicación OAuth con Twitch. Documentación: <https://api-docs.igdb.com/>. |

Cada uno de los tres proyectos tendrá un **repositorio público e independiente en GitHub** con el nombre indicado en la tabla, para que los reclutadores puedan revisar el código. Crear esos tres repositorios será una tarea independiente del plan de implementación. Los dos microservicios compartirán una misma base PostgreSQL, con un **esquema PostgreSQL propio por servicio** y migraciones Prisma independientes. Ambos usarán Prisma para acceder a ella.

En Neon, cada microservicio usará en ejecución un rol SQL restringido a su propio esquema. Las migraciones se aplicarán mediante GitHub Actions independientes de AuthUser y Game, con credenciales de migración separadas de las de ejecución local/Vercel. `GB-003.06` ya probó roles migradores SQL limitados en Neon `develop` y `production`; los migradores históricos que heredan `neon_superuser` **no** se utilizarán como secretos de Actions. La situación aprobada consta en la [`línea base contractual v3`](../contracts/contract-baseline-v3.md) y la [enmienda de acceso v2](../contracts/persistence-access-amendment-v2.md); los Actions pendientes figuran en [`../tasks/mvp.md`](../tasks/mvp.md).

Al concluir la implementación y publicación de las tres aplicaciones se creará un **cuarto repositorio público, solo de documentación**, llamado `GameBook.System`. No es un microservicio ni una cuarta aplicación desplegable. Su primera publicación reunirá la documentación general y la evidencia de la arquitectura final. Tendrá únicamente la rama `main` y se publicará una sola vez, con todo el contenido terminado; no seguirá el flujo `develop`/`feature` ni usará release-please.

Los proyectos de backend seguirán una arquitectura basada en DDD, con separación explícita de dominio, casos de uso, servicios y repositorios. La lógica de negocio no dependerá directamente de controladores, detalles de Prisma ni infraestructura externa. Los repositorios encapsularán la persistencia y los casos de uso coordinarán las operaciones. En el frontend se priorizarán componentes y módulos pequeños, con responsabilidades claras. Los archivos de todos los proyectos deberán ser legibles para reclutadores y no crecer de forma extrema.

Para ambos backends se prefiere una organización reconocible con `src/` y `tests/` en la raíz, `main.ts` dentro de `src/`, y áreas `api/`, `application/`, `domain/` e `infrastructure/` dentro de `src/`. `api/` representa la capa de presentación HTTP y Swagger. `tests/` contendrá pruebas unitarias y de integración. Se permite ajustar la organización interna si se justifica una alternativa más clara que conserve esos límites DDD; el plan concretará la propuesta. Los archivos propios de Prisma, como esquema y migraciones, podrán ubicarse aparte en la raíz conforme a sus convenciones.

Los tres proyectos, backend y frontend, usarán **pnpm** para gestionar dependencias y ejecutar sus scripts, con un archivo de bloqueo versionado por repositorio. Las interfaces exactas, modelos, rutas y contratos entre componentes se concretarán mediante la tarea de contratos del plan.

### 3.1 Documentación de los microservicios

- `GameBook.Microservice.AuthUser` y `GameBook.Microservice.Game` documentarán sus APIs con **Swagger/OpenAPI** y ofrecerán una interfaz Swagger UI consultable en los despliegues de `main`, de forma que los reclutadores puedan examinar sus contratos.
- La documentación incluirá todas las operaciones expuestas, propósito, entradas, salidas, validaciones relevantes, respuestas de éxito y de error, e indicará cuáles operaciones requieren autenticación.
- Las operaciones protegidas se documentarán con el esquema **HTTP Bearer JWT**. Swagger UI permitirá proporcionar manualmente un JWT válido para probarlas; la documentación no omitirá la validación de seguridad de la API.
- La ruta concreta de cada documentación y los contratos de los endpoints se fijarán durante la tarea de contratos del plan de implementación. No se publicarán llaves privadas, API keys, contraseñas ni tokens reales en los ejemplos.

### 3.2 Tema del frontend

- `GameBook.Frontend` ofrecerá un control para alternar entre modo claro y modo oscuro.
- Ambos modos visuales deberán cubrir las vistas públicas y autenticadas, incluidos formularios, tarjetas, filtros, modal, navegación y mensajes de estado.
- El tema podrá cambiarse sin cerrar la sesión ni perder la vista o los filtros activos.
- En la primera visita, el tema inicial seguirá la preferencia de modo claro u oscuro configurada en el sistema de la persona.
- Si la persona elige manualmente un tema, esa elección prevalecerá sobre la preferencia del sistema y se conservará al recargar o regresar a GameBook. El mecanismo de persistencia se decidirá en el plan de implementación; la preferencia debe funcionar también sin cuenta.

### 3.3 Idiomas de la interfaz

- `GameBook.Frontend` ofrecerá un control para alternar entre **English** y **Español**, disponible tanto para visitantes como para personas con sesión.
- En la primera visita, los textos de la interfaz aparecerán en inglés, independientemente del idioma configurado en el navegador.
- El idioma seleccionado se aplicará a los textos propios del sistema en todas las vistas: navegación, formularios, filtros, botones, tooltips, confirmaciones, errores, estados vacíos, estados de carga y mensajes de éxito.
- Las etiquetas escritas en español en las demás secciones de este documento corresponden a la versión española de la interfaz; la versión inglesa mostrará textos equivalentes con el mismo significado y las mismas acciones.
- Cambiar de idioma no cerrará la sesión ni eliminará la vista o los filtros activos.
- Los datos recibidos de IGDB, incluidos títulos, descripciones, nombres de géneros y plataformas, se mostrarán como los entregue esa API. El MVP **no traducirá el contenido de IGDB** ni añadirá un servicio de traducción.
- Cuando una persona elija manualmente un idioma, GameBook recordará esa elección al recargar o regresar a la página, también si no tiene cuenta. El mecanismo de persistencia se decidirá en el plan de implementación.

La localización de los datos externos de videojuegos podrá evaluarse en otra fase, previa ampliación explícita de esta especificación.

### 3.4 Herramientas de diseño y documentación visual

- Las tareas relacionadas con el diseño, construcción o revisión visual de la interfaz del frontend utilizan el skill `interface-design`. La dirección visual, los componentes y los estados se evalúan con ese método antes de considerarlos terminados; las instrucciones del skill pertenecen al entorno de trabajo y no se publican en este repositorio documental.
- Al finalizar el desarrollo se utiliza el skill `archify` para generar, como mínimo, un diagrama general de la **arquitectura efectivamente implementada** que abarque el frontend, los dos microservicios, Neon, el flujo OAuth de aplicación con Twitch, IGDB y sus relaciones. Se publican variantes equivalentes del diagrama con etiquetas y explicaciones **en inglés y en español** en `GameBook.System`, basadas en evidencias de los tres repositorios y del despliegue, no solo en la arquitectura prevista por esta especificación. La interfaz fija del visor Archify puede permanecer en inglés en la variante española; esta limitación se indica junto al enlace. Las instrucciones del skill pertenecen al entorno de trabajo y no se publican aquí.
- La ejecución de ambos skills corresponde a sus futuras tareas; esta fase documental no genera interfaz ni diagramas.

## 4. Experiencia pública de consulta

### 4.1 Primera visita

Al abrir la página principal por primera vez, la persona verá un listado de juegos presentados como los más populares de la historia. Para este MVP, «más populares» significa **mejor puntuados según la valoración combinada de usuarios y crítica de IGDB**, el campo `total_rating` (escala 0–100). El listado inicial usará `sort total_rating desc` y excluirá los juegos sin esa valoración. No se equiparará esta puntuación con ventas ni con número de jugadores.

La consulta pública no requiere registro, sesión ni JWT de GameBook. IGDB exige un Client ID y un token OAuth de aplicación obtenido mediante `client_credentials` de Twitch. Las consultas a IGDB se realizarán desde el servidor de Next.js, localmente durante el desarrollo y en Vercel solo en producción; el navegador pedirá los datos al frontend, que obtendrá y renovará el token de aplicación según su expiración y consultará IGDB. `IGDB_CLIENT_ID` e `IGDB_CLIENT_SECRET` serán variables de entorno privadas del frontend, sin prefijo `NEXT_PUBLIC_`; el secreto y el token de aplicación nunca se enviarán al navegador ni se confundirán con el JWT del usuario de GameBook. [IGDB Getting Started](https://api-docs.igdb.com/), [variables de servidor en Next.js](https://nextjs.org/docs/app/getting-started/server-and-client-components).

### 4.2 Filtros

- **Nombre:** campo de texto que muestra sugerencias de juegos a medida que la persona escribe y permite elegir la opción deseada.
- **Plataforma o consola:** campo que muestra sugerencias de plataformas a medida que la persona escribe y permite elegir una opción.
- Las sugerencias se podrán seleccionar mediante clic o teclado aunque el input pierda el foco durante la interacción; al seleccionar una opción se conservará el texto o ID elegido y se cerrará el desplegable.
- **Año de lanzamiento:** permite seleccionar un año concreto o un intervalo de años.
- Los filtros de nombre, plataforma y año se podrán **combinar simultáneamente**; el listado mostrará los juegos que cumplan todos los criterios activos.
- Los tres filtros estarán disponibles tanto en la consulta pública como en la vista de favoritos y tendrán el mismo comportamiento visible en ambas vistas.
- Ambas vistas cargarán los siguientes resultados mediante **desplazamiento infinito** cuando haya más juegos disponibles. La interfaz mostrará que se están cargando más resultados y dejará de solicitarlos cuando se alcance el final.
- Si no hay juegos que cumplan los filtros, se mostrará un estado vacío comprensible.
- No se agregan más filtros al MVP.

IGDB v4 permite búsqueda textual de juegos y plataformas, filtros combinados en `/games` y paginación por `limit`/`offset`. Un año concreto se expresa como el intervalo UTC del 1 de enero al 1 de enero del año siguiente, con límite superior exclusivo, sobre `first_release_date`; dos años delimitan de la misma manera un rango inclusivo de años. La búsqueda textual puede seguir relevancia; el listado inicial sin búsqueda usa la clasificación por `total_rating`. [Documentación oficial de IGDB](https://api-docs.igdb.com/).

El listado inicial se ordenará por `total_rating` descendente. La presentación exacta de las sugerencias y de los estados de carga se resolverá en el diseño de interfaz, respetando el comportamiento descrito aquí.

### 4.3 Tarjetas y detalle

- Los resultados se mostrarán como tarjetas con **imagen, nombre, año de lanzamiento, plataformas y puntuación** recibidos de IGDB, cuando esos datos estén disponibles.
- Cada tarjeta tendrá una acción visible para guardar en favoritos, representada por una estrella o un corazón; la elección visual final queda pendiente.
- Al seleccionar una tarjeta, se abrirá un modal con los datos de la tarjeta y, además, **descripción, géneros, desarrolladores y fecha completa de lanzamiento**, obtenidos de IGDB cuando estén disponibles con esa precisión. Si IGDB solo informa un año o una fecha aproximada, no se inventará día ni mes. El modal tendrá una acción para cerrarlo y otra para guardar en favoritos.
- Si IGDB no entrega alguno de esos campos, el diseño deberá mostrar el resto de la información sin inventar datos.

### 4.4 Acciones de favoritos sin sesión

Si una persona sin sesión pulsa la acción de favoritos en una tarjeta o en el modal, se mostrará un tooltip o elemento contextual junto a la acción. En español contendrá el texto **«Para guardar en favoritos debes:»** y las opciones **«Iniciar sesión»** y **«Crear cuenta»**. En inglés mostrará los textos equivalentes. Se usará el mismo mensaje en la tarjeta y en el modal para cada idioma.

Ninguna de estas acciones guardará un favorito mientras no haya sesión.

### 4.5 Acciones de favoritos con sesión

Si la persona ha iniciado sesión y pulsa la acción de favoritos en la tarjeta o en el modal, aparecerá una confirmación contextual. Solo tras confirmar se solicitará el guardado. El resultado mostrará un mensaje de éxito o de error.

## 5. Cuenta y autenticación

### 5.1 Navegación y formularios

- La barra de navegación superior mostrará **«Iniciar sesión»** y **«Crear cuenta»** cuando no haya sesión.
- El formulario de creación de cuenta solicitará nombre completo, correo electrónico, contraseña y confirmación de contraseña.
- Los campos obligatorios no podrán enviarse vacíos; el correo deberá tener un formato válido y la confirmación de contraseña deberá coincidir con la contraseña. Los errores de validación se mostrarán de forma comprensible.
- Cada correo electrónico corresponderá a una sola cuenta. El registro de un correo ya utilizado deberá mostrar un error comprensible y no crear otra cuenta.
- Tras crear una cuenta correctamente, la persona irá al formulario de inicio de sesión; el registro no iniciará la sesión automáticamente.
- El formulario de inicio de sesión solicitará correo electrónico y contraseña.
- Tanto el registro como el cambio de contraseña exigirán **al menos ocho caracteres**, con **al menos una letra mayúscula, un número y un carácter especial** (por ejemplo, `$`, `%` o `@`). No se ha exigido una letra minúscula.
- Tras iniciar sesión, la barra superior mostrará un icono de usuario. Al activarlo, estarán disponibles las opciones **«Perfil»** y **«Juegos favoritos»**.
- En **Perfil**, la única actualización de datos permitida por el MVP será la contraseña. Para cambiarla se solicitarán la **contraseña actual** y la nueva contraseña.
- Al cerrar sesión, la persona volverá a la página principal de consulta pública.

### 5.3 Desactivación lógica de cuentas

- Una persona autenticada podrá solicitar la desactivación de su cuenta desde la sección **Perfil** mediante `DELETE /v1/users/me`. La operación será lógica: AuthUser conservará el registro y el parámetro booleano persistente `isDisabled` indicará que la cuenta está deshabilitada; una respuesta correcta será `204` sin cuerpo.
- Al desactivar una cuenta, AuthUser invalidará los JWT emitidos para ella y el frontend retirará el token actual, cerrará la sesión y regresará a la consulta pública.
- Un intento de iniciar sesión con el correo de una cuenta deshabilitada no autenticará a la persona y devolverá un error estable que el frontend mostrará en inglés y español indicando que la cuenta está deshabilitada y que debe usar otro correo o crear una cuenta nueva.
- Un intento de registrar una cuenta con el correo de una cuenta deshabilitada no reactivará ni duplicará el registro y mostrará el mismo estado de cuenta deshabilitada, con la indicación de usar otro correo o crear una cuenta nueva.
- Game rechazará operaciones protegidas cuando el JWT corresponda a una cuenta deshabilitada; no se accederá ni modificará la lista de favoritos de esa cuenta.
- No habrá reactivación de cuentas en el MVP. La cuenta y sus datos, incluidos sus favoritos, se conservarán indefinidamente tras la desactivación lógica; no habrá purga automática ni física durante el MVP.

### 5.4 Disponibilidad de AuthUser y Game en Render

- `GameBook.Microservice.AuthUser` y `GameBook.Microservice.Game` expondrán un endpoint público `GET /health` que responderá exactamente con HTTP `200` cuando el proceso del servicio esté listo para recibir solicitudes. El healthcheck no requerirá JWT ni dependerá de una operación protegida.
- Antes de cada llamada del frontend a cualquier endpoint de AuthUser o Game, el cliente correspondiente solicitará `GET /health` del mismo servicio. La llamada prevista al servicio solo se ejecutará después de recibir HTTP `200` del healthcheck.
- Cada intento del healthcheck tendrá un tiempo máximo de espera de **15 segundos**. Si no recibe HTTP `200`, el frontend podrá reintentar únicamente ese `GET /health` hasta **15 reintentos adicionales** (16 intentos como máximo contando el inicial). Las llamadas previstas a AuthUser o Game no se reintentarán mediante este mecanismo.
- Mientras espera el healthcheck, el frontend no mostrará una interacción adicional al usuario; conservará los estados de carga y error ya definidos para la operación que motivó la llamada. Si no obtiene HTTP `200` después del límite, no ejecutará la llamada prevista y propagará el error de indisponibilidad correspondiente.

### 5.5 Transporte HTTP del frontend

- `GameBook.Frontend` utilizará Axios como adaptador de transporte HTTP y RxJS como capa de composición de las solicitudes, incluyendo las llamadas a AuthUser, Game, los healthchecks y las rutas server-side de Next.js que consultan IGDB o Twitch.
- La migración tecnológica no cambiará los contratos HTTP ni el comportamiento visible: se conservarán URLs, métodos, cabeceras, cuerpos, JWT, códigos y mapeos de error, cancelación, estados de carga, caché y límites de reintento definidos para cada proveedor.
- Las operaciones del frontend podrán conservar una interfaz basada en `Promise` en sus límites de dominio o de UI; RxJS se utilizará internamente para componer, cancelar, transformar y reintentar flujos HTTP sin obligar a los componentes a adoptar Observables.
- La política de reintentos seguirá siendo segura: las solicitudes solo se reintentarán cuando la regla vigente lo permita; nunca se repetirán mutaciones por una conversión genérica de transporte. El healthcheck conservará su límite de 15 segundos por intento y hasta 15 reintentos adicionales.
- Axios y RxJS se incorporarán como dependencias explícitas, con pruebas que demuestren que la migración conserva el flujo funcional y no expone credenciales de IGDB/Twitch ni modifica la separación server-only de esos secretos.

### 5.2 JWT

- La autenticación utilizará JWT con vigencia de **una hora**.
- Los JWT usarán firma asimétrica. `GameBook.Microservice.AuthUser` firmará con la **clave privada** almacenada como variable de entorno de su proyecto en Vercel. `GameBook.Microservice.Game` verificará la firma con la **clave pública** correspondiente, configurada en su entorno. La clave privada no se compartirá con Game ni se incluirá en repositorios o en el navegador.
- El frontend guardará el token en `sessionStorage` tras un inicio de sesión válido.
- El frontend enviará el JWT en toda petición autenticada a los microservicios que lo requieran.
- Para guardar, listar, filtrar, sincronizar o eliminar favoritos, el frontend enviará a `GameBook.Microservice.Game` el **mismo JWT de usuario emitido por AuthUser**, en la cabecera `Authorization: Bearer <token>`, como cliente directo de esa API. Game no recibirá un token distinto emitido para comunicación entre servicios.
- `GameBook.Microservice.Game` verificará por sí mismo la firma con la clave pública y la vigencia del JWT antes de ejecutar cada operación protegida. Obtendrá el UUID de usuario del JWT validado para delimitar sus favoritos; no confiará en un identificador de usuario enviado por el cliente como sustituto de esa identidad.
- Las solicitudes protegidas sin token, con token inválido o vencido serán rechazadas y no modificarán datos.
- Al vencer el JWT, el frontend cerrará la sesión, retirará el token de `sessionStorage` y solicitará un nuevo inicio de sesión para acciones protegidas. No se renovará automáticamente.
- Al cambiar la contraseña, se invalidarán de inmediato **todos** los JWT previamente emitidos para esa cuenta, incluido el de la sesión actual. El frontend retirará el token y solicitará un nuevo inicio de sesión. AuthUser mantendrá el estado mínimo de revocación necesario; Game comprobará ese estado con AuthUser además de verificar localmente la firma y vigencia del JWT, sin confiar únicamente en la expiración. Si la comprobación de revocación no está disponible, Game no ejecutará operaciones protegidas.
- Las solicitudes públicas de consulta de IGDB no requerirán JWT.
- Los identificadores de usuario serán **UUID**.

Las reglas anteriores son las validaciones funcionales exigidas para el MVP. Los textos exactos de los mensajes y la validación técnica de cada campo se concretarán durante el diseño de interfaz y contratos.

## 6. Juegos favoritos

- Guardar un favorito requiere una cuenta con sesión iniciada.
- Cada usuario podrá tener una sola entrada por juego. Si intenta guardar un juego que ya está en su lista, no se creará un duplicado y se mostrará un aviso de que ya existe.
- El microservicio `GameBook.Microservice.Game` almacenará en PostgreSQL, mediante Prisma, **solo la información mínima necesaria** para identificar el juego y hacer funcionar los filtros y las tarjetas de favoritos: identificador del usuario, identificador del juego en IGDB, nombre, fecha de lanzamiento, plataformas, URL de imagen y puntuación. El año para el filtro se obtendrá de la fecha de lanzamiento cuando exista. La forma física de representar las plataformas se definirá durante el diseño del esquema, sin ampliar los datos guardados.
- La lista de favoritos será personal: mostrará los juegos guardados por la persona autenticada.
- La vista de favoritos usará tarjetas y filtros con el mismo diseño y comportamiento que la consulta pública.
- La tarjeta de un favorito tendrá una acción para eliminarlo, en una ubicación equivalente a la acción de guardar en la vista pública.
- El modal de detalle de un favorito también incluirá la acción **«Eliminar de favoritos»**.
- Tanto la acción de la tarjeta como la del modal solicitarán confirmación antes de eliminar el favorito.
- Al abrir el modal, el frontend consultará IGDB para obtener los datos completos del juego; la base local no sustituirá esa consulta de detalle.
- Cuando se abra el detalle de un favorito y IGDB responda correctamente, los campos básicos almacenados se sincronizarán con los valores actuales de IGDB si cambiaron. No habrá sincronización periódica ni sincronización masiva al entrar a la lista.
- Si IGDB falla al consultar el detalle de un favorito, la tarjeta seguirá visible con los datos almacenados. El modal indicará que el detalle no está disponible en ese momento, sin borrar el favorito.

La información de cada usuario y sus favoritos debe mantenerse separada de la de otras cuentas.

## 7. Calidad, pruebas y observabilidad

### 7.1 Pruebas

- Cada microservicio implementará pruebas unitarias y de integración para sus casos de uso y los límites de infraestructura relevantes.
- El frontend implementará pruebas unitarias y, si resulta viable, también pruebas de integración.
- Las pruebas deberán cubrir los flujos principales y los resultados de éxito y error definidos en este documento, sin depender de llamadas reales a IGDB en las pruebas unitarias.
- Las pruebas de integración de Game verificarán que un JWT válido de AuthUser permite las operaciones del usuario correspondiente y que los tokens ausentes, inválidos o vencidos no permiten acceder a favoritos.

### 7.2 Logs de backend

- Los procesos de backend producirán logs detallados y legibles, preferentemente en inglés.
- Los logs reflejarán operaciones exitosas y errores, con información suficiente para seguir una operación.
- Cuando exista contexto de usuario, se procurará incluir su identificador para correlacionar los eventos asociados a la operación.
- Los logs no deberán revelar contraseñas, JWT, llaves privadas ni API keys.

## 8. Repositorios, ramas, versiones y despliegue

- Cada proyecto tendrá su propio repositorio público en GitHub.
- Cada repositorio de aplicación tendrá un `README.md` principal **en inglés** y una versión completa **en español** enlazada desde él. Durante el desarrollo ambos README se comprobarán en `develop`; el README inglés será visible al entrar en GitHub, cuya rama predeterminada es `main`, **después de la integración final para producción**. Ambas versiones enlazarán los repositorios relacionados cuando existan.
- La creación y publicación inicial de los repositorios, así como la creación y gestión de pull requests, se hará mediante **GitHub CLI (`gh`)**, que ya está disponible en el equipo. Los commits y los push posteriores utilizarán Git; `gh` gestionará los PR y las operaciones de GitHub correspondientes.
- Las ramas principales de cada **repositorio de aplicación** serán `main` (predeterminada) y `develop`.
- Las ramas de trabajo de las aplicaciones seguirán el patrón `feature/[task-number]/[resumen-de-tarea]`.
- En las aplicaciones, las ramas `feature/...` se integrarán mediante pull request en `develop`. Todas las comprobaciones de entregables durante el desarrollo se harán sobre `develop`, no sobre `main`; no se exigirá que los README ni el código en curso estén en `main`. **Solo al final de todas las implementaciones y de la validación previa a producción**, `develop` se integrará en `main`, que será la rama desplegada. `GameBook.System` tiene la excepción documental descrita en §8.1.
- `main` y `develop` de los tres repositorios de aplicación no tendrán protección de rama, revisiones obligatorias ni checks obligatorios para fusionar PR. Se mantienen los PR y la ejecución de CI como práctica de calidad, sin convertirlos en restricciones de GitHub.
- El autor fusionará o cerrará personalmente los PR. Cada subtarea registrará en `tasks/mvp.md` los PR que produzca. Tras la notificación del autor, el agente comprobará en GitHub que todos los PR necesarios estén fusionados en su rama de destino y que el entregable cumpla sus criterios antes de marcarla `[ RESOLVED ]`; un PR abierto o cerrado sin merge no basta.
- Los commits y las descripciones de los pull requests estarán en inglés y detallarán el objetivo de la tarea. Los mensajes de commit seguirán **Conventional Commits** para que release-please pueda calcular versiones y notas de publicación.
- El plan de implementación elegirá la estrategia de merge de los PR, conservando mensajes Conventional Commits legibles en el historial de `main`.
- Los tres proyectos usarán **release-please** para el versionamiento, con `main` como rama objetivo de sus pull requests de versión. Según el flujo documentado de release-please, las versiones y notas de publicación se preparan en un release PR y la publicación se produce al integrarlo. [Documentación oficial de release-please](https://github.com/googleapis/release-please-action).
- El MVP completo se probará localmente antes de crear los proyectos Vercel definitivos. Un Docker Compose permitirá levantar juntos los tres repositorios independientes para la integración local; no habrá ambiente ni despliegues **Vercel Preview**.
- La rama desplegada en Vercel será exclusivamente `main` para el frontend y los dos microservicios, después de la aceptación local. Las ramas `feature/...` y `develop` no se desplegarán allí.
- Las variables de producción de las aplicaciones se administrarán en Vercel solo al publicar. Desarrollo/pruebas usarán variables locales privadas. Neon mantendrá ambientes `develop` y `production` y las credenciales de IGDB/Twitch también se separarán por ambiente; ningún secreto de producción se usará para pruebas locales.
- La base PostgreSQL compartida por ambos microservicios se alojará en **Neon** tanto para pruebas como para producción, con recursos/ramas separados. Las migraciones seguirán a cargo de los GitHub Actions de cada backend con secretos separados por ambiente.

El autor eliminará manualmente los tres proyectos Vercel iniciales. La nueva política operativa está en [`../environments/deployment-policy-local-first.md`](../environments/deployment-policy-local-first.md) y prevalece sobre menciones históricas a previews en la línea base contractual congelada. La creación de los repositorios, configuración de ramas, automatización de versiones, recursos de despliegue y base de datos se organiza como trabajo separado en el plan de implementación. El avance real de cada subtarea consta en `tasks/mvp.md`.

### 8.1 Repositorio de documentación `GameBook.System`

- Se creará **después** de terminar las implementaciones, mediante GitHub CLI, como repositorio público con solo `main` y una única publicación inicial del contenido completo. Los ajustes documentales posteriores, como automatizaciones o enlaces de publicación, podrán integrarse mediante PR hacia `main`, sin crear `develop` ni configurar release-please. No recibirá despliegue Vercel.
- Documentará el propósito del sistema, sus funcionalidades públicas y autenticadas, la forma en que se implementaron los componentes, los contratos e integraciones, los datos y la seguridad, las pruebas, el despliegue, las limitaciones conocidas y la arquitectura final. Debe describir lo que realmente quedó implementado, distinguiéndolo de las decisiones históricas de SDD.
- Publicará los archivos de contexto y trazabilidad generados en esta conversación y durante el trabajo: `AGENTS.md`, `CLAUDE.md`, `specs/`, `plan/`, `tasks/` cuando exista, y otros documentos pertinentes. Se revisará el inventario antes de copiar para excluir secretos, cachés, dependencias y archivos locales ajenos al proyecto.
- Tendrá un `README.md` principal en inglés y su versión española. El principal incluirá enlaces a los tres repositorios de aplicación. Los README de las aplicaciones enlazarán de vuelta a `GameBook.System` cuando exista.
- Los HTML standalone de `architecture/` se publicarán como contenido estático mediante un workflow de GitHub Actions y GitHub Pages. `README.md` enlazará únicamente el diagrama HTML en inglés y `README.es.md` únicamente el diagrama HTML en español mediante sus URLs públicas localizadas; el workflow no ejecutará código de aplicación ni utilizará secretos.
- **Todos los archivos Markdown publicados en `GameBook.System` tendrán versiones completas en inglés y español.** La convención será nombre base `.md` para inglés y el sufijo `.es.md` para español: `README.md`/`README.es.md`, `AGENTS.md`/`AGENTS.es.md`, `CLAUDE.md`/`CLAUDE.es.md`, `specs/mvp.md`/`specs/mvp.es.md`, `plan/mvp.md`/`plan/mvp.es.md` y el mismo patrón para `tasks/` y cualquier otro Markdown. Las traducciones conservarán significado, enlaces y decisiones; ninguna versión será un resumen de la otra.
- En los tres repositorios de aplicaciones se exige **al menos el README bilingüe** (`README.md` inglés y `README.es.md` español), pero no duplicar obligatoriamente cada otro archivo Markdown. Los documentos de trabajo actuales pueden mantenerse en español hasta preparar la publicación bilingüe de `GameBook.System`.

## 9. Restricciones de la integración con IGDB

IGDB v4 requiere Client ID y token OAuth de aplicación de Twitch en las cabeceras de sus solicitudes `POST` con cuerpo de consulta APICalypse. No permite acceso directo desde JavaScript del navegador por CORS; el servidor de Next.js actuará como intermediario. La documentación cubre catálogo, búsqueda, plataformas, puntuaciones, filtros combinados, `limit`/`offset`, detalles e imágenes. El mapeo vigente está en [`../contracts/igdb-mapping.md`](../contracts/igdb-mapping.md). [Documentación oficial de IGDB](https://api-docs.igdb.com/).

La documentación vigente publica un límite de **4 solicitudes por segundo** y **8 simultáneas**; las respuestas `429` deberán mostrarse como indisponibilidad temporal con reintentos acotados. IGDB indica gratuidad para uso no comercial según sus términos de Twitch y solicita atribución visible; antes del despliegue se volverán a verificar las condiciones de uso, especialmente si el sitio se monetiza. El frontend deberá contemplar datos o imágenes ausentes y caídas del proveedor. No se almacenarán ni mostrarán credenciales reales en documentación, repositorios, logs o respuestas. [IGDB API y FAQ](https://api-docs.igdb.com/).

## 10. Criterios de aceptación funcional preliminares

1. Una visita sin sesión muestra los juegos ordenados por puntuación descendente de IGDB, permite combinar los tres filtros, cargar resultados adicionales mediante desplazamiento infinito y abrir el detalle de un juego.
2. Una persona sin sesión que intenta guardar un juego recibe las opciones para iniciar sesión o crear cuenta y no modifica favoritos.
3. Es posible crear una cuenta con los cuatro campos solicitados e iniciar sesión con correo y contraseña.
4. Una persona con sesión puede confirmar el guardado desde una tarjeta o desde el modal y recibe un resultado visible de éxito o error.
5. Una persona con sesión puede abrir su lista de favoritos, aplicar los mismos tres filtros y ver detalles obtenidos de IGDB.
6. Una persona con sesión puede eliminar un favorito desde la tarjeta o el modal, tras confirmar la acción.
7. La opción de perfil permite actualizar la contraseña, sin otros cambios de perfil previstos en el MVP.
8. Al cerrar sesión se regresa a la página principal pública.
9. Los endpoints de favoritos y perfil protegidos no deben entregar ni modificar datos de otra cuenta.
10. Al vencer el JWT, la sesión se cierra y las operaciones protegidas exigen un nuevo inicio de sesión.
11. Si falla IGDB al abrir el detalle de un favorito, la tarjeta guardada permanece y se informa que el detalle no está disponible. Si IGDB responde con cambios, los campos básicos guardados se actualizan.
12. En la primera visita se aplica el tema preferido por el sistema. La persona puede alternar entre modo claro y oscuro en cualquier vista; la elección manual persiste al recargar o regresar y ambas modalidades mantienen legibles y utilizables todos los controles y mensajes.
13. La primera visita muestra la interfaz en inglés. La persona puede cambiarla a español y volver al inglés en cualquier vista sin perder la sesión, la vista o los filtros. Su elección se conserva al recargar o regresar. Todos los textos propios de GameBook siguen el idioma seleccionado; los datos entregados por IGDB permanecen tal como se recibieron.
14. Ambos microservicios ofrecen documentación Swagger/OpenAPI consultable que identifica operaciones, contratos, respuestas y requisitos de autenticación sin revelar secretos.
15. Game acepta el JWT de usuario emitido por AuthUser cuando el frontend lo envía como Bearer; valida el token y limita las operaciones al UUID autenticado. Las peticiones protegidas sin JWT válido son rechazadas.
16. Tras cambiar la contraseña, los JWT anteriores dejan de autorizar operaciones en AuthUser y Game, incluso antes de su vencimiento; la sesión actual vuelve al inicio de sesión.
17. Una persona autenticada puede solicitar desde Perfil la desactivación lógica de su cuenta; el registro permanece almacenado con un indicador booleano de cuenta deshabilitada.
18. Tras desactivar la cuenta, el token actual se retira, la sesión termina y el acceso vuelve a la consulta pública.
19. El login y el registro con el correo de una cuenta deshabilitada no crean sesión ni duplicado, y muestran el mensaje traducido de cuenta deshabilitada.
20. Game rechaza con seguridad las operaciones protegidas de una cuenta deshabilitada, incluidos los favoritos, aunque el JWT no haya vencido.

Los contratos verificables y casos de prueba concretos se detallan en el plan de implementación sin cambiar el alcance de estos criterios.

## 11. Decisiones de diseño del plan

El plan de implementación detalla la estrategia para contratos de API, nombres de rutas, estructura de carpetas, esquema físico de Prisma y merge de pull requests sin ampliar el alcance funcional fijado aquí. La presentación visual exacta se resolverá durante la tarea de diseño del frontend. Cualquier requisito nuevo o cambio de comportamiento deberá incorporarse primero a esta especificación.
