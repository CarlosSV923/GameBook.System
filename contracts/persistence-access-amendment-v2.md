# GB-003.06 — Enmienda de acceso a Neon para migraciones limitadas

Estado: verificación completada el 2026-09-23 para `GB-003.06`. Esta revisión
complementa y sustituye únicamente la parte de credenciales de migración de
[`persistence-access-amendment.md`](persistence-access-amendment.md). No
cambia tablas, contratos HTTP, JWT ni el aislamiento runtime ya acreditado.

## 1. Roles limitados

En las ramas Neon `develop` y `production` se crearon mediante SQL los roles:

| Servicio | Rol migrador limitado | Esquema permitido | Runtime receptor de grants |
| --- | --- | --- | --- |
| AuthUser | `gamebook_auth_migrator_limited` | `auth` | `gamebook_auth_app` |
| Game | `gamebook_game_migrator_limited` | `game` | `gamebook_game_app` |

Ambos roles tienen `LOGIN`, `NOINHERIT`, `NOSUPERUSER`, `NOCREATEDB`,
`NOCREATEROLE`, `NOREPLICATION` y `NOBYPASSRLS`. Tienen `CONNECT` a la base,
pero no `CREATE` en la base. Cada rol posee `USAGE` y `CREATE` únicamente en
su esquema y no tiene `USAGE` ni `CREATE` en el esquema ajeno.

Los esquemas siguen perteneciendo a los migradores históricos
`gamebook_auth_migrator` y `gamebook_game_migrator`. Estos roles heredan
`neon_superuser`, no se usarán en GitHub Actions y se conservan solo como
propietarios administrativos de los esquemas hasta una operación explícita de
traspaso. `neondb_owner` creó los nuevos roles por SQL y queda como autoridad
para su rotación y administración; ninguna credencial se copia a este
repositorio.

## 2. Grants por defecto y ubicación futura

Los nuevos migradores conceden por defecto a su runtime correspondiente
`SELECT`, `INSERT`, `UPDATE`, `DELETE` sobre tablas y `USAGE`, `SELECT` sobre
secuencias futuras. No conceden privilegios al runtime del otro servicio.

| Variable | Rol permitido | Destino posterior |
| --- | --- | --- |
| `AUTH_DATABASE_DIRECT_URL` | `gamebook_auth_migrator_limited` | Secreto de GitHub Actions AuthUser |
| `GAME_DATABASE_DIRECT_URL` | `gamebook_game_migrator_limited` | Secreto de GitHub Actions Game |
| `AUTH_DATABASE_URL` | `gamebook_auth_app` | Runtime Render AuthUser |
| `GAME_DATABASE_URL` | `gamebook_game_app` | Runtime Render Game |

`GB-003.04` configurará los secretos por ambiente. No se usan los migradores
históricos ni se colocan URLs directas en Render o Vercel.

## 3. Evidencia de pruebas

- En `develop` se comprobó conexión real de ambos roles y sus atributos SQL:
  `neon_superuser=false`, `CREATEDB=false`, `CREATEROLE=false`,
  `CREATE` de base=false y `NOINHERIT`.
- Cada rol creó, alteró y eliminó una tabla efímera dentro de su esquema.
- La lectura y el DDL en el esquema ajeno fallaron con `permission denied` en
  ambas direcciones.
- Se comprobó que no quedaron tablas efímeras y que los ACL por defecto solo
  apuntan al runtime propio.
- Tras el éxito de `develop`, se repitieron las mismas comprobaciones en
  `production`, sin ejecutar migraciones Prisma ni crear tablas de aplicación.

## 4. Límite de esta revisión

Esta enmienda acredita la preparación de credenciales limitadas, no la
configuración de secretos GitHub/Render/Vercel ni la ejecución de migraciones. Los
workflows controlados pertenecen a `GB-004.07` y `GB-005.07`.

