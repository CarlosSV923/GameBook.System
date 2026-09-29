# GB-003.02 — Enmienda de acceso a Neon y ejecución de migraciones

Esta decisión sustituye **solo** las afirmaciones sobre credenciales y
aislamiento de migración de `persistence-environments.md` en la línea base
`GB-002-CONTRACTS-v1`. No cambia tablas, campos, contratos HTTP ni el límite
funcional de los servicios. Véase `contract-baseline-v2.md`.

## 1. Decisión verificada

Una sola base Neon conserva los esquemas PostgreSQL `auth` y `game`. Cada
microservicio mantiene su propio Prisma Client y sus migraciones en su
repositorio; no hay claves foráneas entre esquemas.

| Ambiente Neon | Servicio | Rol de runtime | Rol de migración |
| --- | --- | --- | --- |
| `develop` y `production` | AuthUser | `gamebook_auth_app` | `gamebook_auth_migrator` |
| `develop` y `production` | Game | `gamebook_game_app` | `gamebook_game_migrator` |

Los roles `*_app` se crearon mediante SQL, tienen `LOGIN` y no heredan
`neon_superuser`. Cada uno tiene `USAGE` solo en su esquema y carece de
`CREATE` en la base y en ambos esquemas. Los grants por defecto del migrador
propio conceden al runtime `SELECT`, `INSERT`, `UPDATE` y `DELETE` sobre las
tablas futuras del esquema correspondiente, y `USAGE`/`SELECT` sobre sus
secuencias. Los dos antiguos roles `*_runtime` creados por la API de Neon,
que heredaban `neon_superuser`, fueron retirados en ambas ramas.

El aislamiento **enforced por PostgreSQL** se garantiza para las conexiones
runtime: Game no puede usar `auth` y AuthUser no puede usar `game`. Los roles
`*_migrator` existentes sí heredan `neon_superuser` por una limitación de los
roles creados mediante la API de Neon. Por tanto, no se afirma que PostgreSQL
impida a un migrador alterar el esquema ajeno. Su límite es **operativo**:
migraciones revisadas y versionadas por repositorio, conexión exclusiva del
Action correspondiente, verificación del esquema objetivo y ejecución
secuencial antes del despliegue. Un migrador comprometido podría afectar
ambos esquemas; es un riesgo aceptado para este MVP de portfolio y debe
figurar en la revisión de seguridad.

## 2. Variables y ubicación de credenciales

| Variable | Destino | Rol permitido |
| --- | --- | --- |
| `AUTH_DATABASE_URL` | Vercel AuthUser runtime | `gamebook_auth_app` |
| `GAME_DATABASE_URL` | Vercel Game runtime | `gamebook_game_app` |
| `AUTH_DATABASE_DIRECT_URL` | GitHub Actions AuthUser, por ambiente | `gamebook_auth_migrator` |
| `GAME_DATABASE_DIRECT_URL` | GitHub Actions Game, por ambiente | `gamebook_game_migrator` |

Los secretos de migración no se inyectan en funciones, builds ni previews de
Vercel. Cada Action obtiene el secreto del ambiente GitHub correspondiente;
los valores de `develop` y `production` son distintos. La generación de
archivos de migración puede hacerse localmente contra recursos de desarrollo,
pero la aplicación sobre las ramas Neon compartidas se hará mediante Actions.
No se publican URLs, contraseñas ni tokens en Markdown, PR o logs.

Los workflows concretos se implementarán junto con las migraciones Prisma en
`GB-004.03` y `GB-005.03`, una vez fijada la versión de Prisma. En `develop`,
podrán activarse tras integrar las migraciones en esa rama. En producción,
deberán ejecutarse de forma controlada desde `main` y completar con éxito
antes de permitir el despliegue Vercel del código dependiente. `GB-012`
deberá comprobar explícitamente ese orden; no se considera suficiente que un
merge y el Action se disparen aproximadamente al mismo tiempo. GitHub exige
que un workflow de ejecución manual exista en la rama predeterminada; hasta
la integración final, no se asumirá disponible el botón manual para
`develop`.

## 3. Criterios de aceptación

1. En ambas ramas Neon, cada `*_app` conecta como su propio usuario SQL, no
   pertenece a `neon_superuser`, tiene `USAGE` solo en su esquema y no tiene
   `CREATE` ni en la base ni en los esquemas.
2. Los grants por defecto de cada migrador apuntan solo a su `*_app` y su
   esquema; los antiguos roles `*_runtime` no existen.
3. Las migraciones y los secretos GitHub/Vercel se preparan en las tareas
   posteriores indicadas; este documento no acredita que ya se hayan
   ejecutado migraciones ni configurado Actions o variables de despliegue.

La evidencia de consultas y operaciones, sin secretos, está en
[`../environments/gb-003.02-schema-isolation.md`](../environments/gb-003.02-schema-isolation.md).
