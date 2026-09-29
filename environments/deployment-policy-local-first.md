# Enmienda operativa — validación local y publicación por plataforma

Decisión del autor: 2026-09-23, actualizada el 2026-09-26. Esta enmienda prevalece sobre las referencias históricas a **Vercel Preview** y a pruebas remotas previas a producción en los contratos y registros de `GB-002` y `GB-003`. AuthUser y Game se publicarán en Render; Frontend podrá publicarse en Vercel desde `main`. No altera las huellas históricas ni reescribe la evidencia de los recursos anteriores.

## Ambientes y recursos

| Etapa | Aplicaciones | Neon | IGDB/Twitch |
| --- | --- | --- | --- |
| Desarrollo y pruebas | AuthUser, Game y Frontend ejecutados localmente; pruebas automatizadas, integración transversal y Docker Compose. Sin Vercel Preview. | Rama/recurso `develop` y roles limitados propios. | Credenciales de prueba/desarrollo en variables locales privadas, distintas de las de producción; nunca en Git ni en imágenes. |
| Producción | AuthUser y Game en Render, y Frontend en Vercel si corresponde, **solo después** de aprobar el MVP local; despliegues desde `main`. | Rama/recurso `production`, tras migraciones controladas de GitHub Actions. | Credenciales de producción configuradas en el proveedor dueño de cada aplicación; las credenciales IGDB/Twitch permanecen solo en el servidor de Frontend. |

Se mantienen los mismos **nombres** de variables por aplicación (`AUTH_DATABASE_URL`, `GAME_DATABASE_URL`, `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET`, JWT, URLs y CORS), con valores apropiados por ambiente. No se copian valores de producción al entorno local. La disponibilidad o validez de cada par IGDB de prueba y producción se acredita con llamadas reales en la etapa correspondiente, sin revelar valores. Si solo hay un par de credenciales, no se lo presenta como dos ambientes aislados: se consulta al autor antes de usarlo para ambos.

`AUTH_DATABASE_DIRECT_URL` y `GAME_DATABASE_DIRECT_URL` permanecen exclusivamente en los ambientes GitHub `develop` y `production` del backend dueño. Los Actions de `develop` aplican migraciones a Neon `develop`; los de `main`, a Neon `production`. Render y Vercel nunca reciben URLs de migración ni ejecutan migraciones en build o arranque.

## Integración local

`GB-012.10` entregará Docker Compose versionado como herramienta de integración local. Debe arrancar los tres repositorios independientes desde un layout de clones hermanos documentado, sin copiar código ni secretos entre ellos; exponer y documentar los puertos locales, las comprobaciones de salud, la secuencia de inicio y el apagado. Los Dockerfiles y el Compose que hagan falta se limitan a desarrollo. La persistencia **seguirá en Neon `develop`**; no se añade un PostgreSQL local que cambie el alcance acordado. Compose no ejecutará migraciones implícitas ni usará credenciales de producción. Se incluirán plantillas de nombres de variables ignorando sus valores reales y un procedimiento reproducible para configurar claves JWT de desarrollo compatibles entre servicios.

Las URLs locales serán `http://localhost:3000` (Frontend), `http://localhost:3001` (AuthUser) y `http://localhost:3002` (Game) si las aplicaciones conservan los puertos previstos. Si Compose publica otros puertos, la documentación y las variables deberán coincidir con los puertos efectivos; no inventar una URL para cerrar una tarea. El navegador usa las URLs públicas **locales** de los backends; la comunicación Game→AuthUser dentro de Compose puede usar el nombre DNS interno del servicio. `CORS_ALLOWED_ORIGINS` de ambos backends contendrá el origen exacto del frontend local (`http://localhost:3000` en el layout previsto), sin `*`. JWT, `AUTHUSER_URL` y URLs de navegador se comprueban con los contenedores reales.

## Puerta de publicación

`GB-012.01` y `GB-012.02` exigen el flujo completo y la auditoría local antes de cualquier publicación remota. `GB-012.11` prepara el inventario de variables y credenciales productivas, sin crear proyectos. `GB-012.05` y `GB-012.06` integran AuthUser y Game en `main`, esperan la migración productiva correspondiente y **después** crean/configuran sus servicios Render, verificando la función real. `GB-012.07` publica Frontend en Vercel si corresponde, configura sus variables Production y fija el origen exacto en ambos backends. Los servicios Render no deben recibir URLs de migración. El orden de migraciones, merges, despliegues y humo productivo figura en `tasks/mvp.md`; no basta que un build termine si la función no atiende solicitudes.

Los registros de `GB-003.03`–`GB-003.05` documentan una preparación **histórica** que se perderá al eliminar los proyectos. Se conservan `[ RESOLVED ]` por la evidencia de su cierre original; **no** constituyen una precondición de Vercel vigente ni prueban que los proyectos o secretos sigan existiendo. La nueva publicación depende de `GB-012.11` y de la validación local, no de reutilizar aquellos recursos.
