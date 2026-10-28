# Desarrollo

Este repositorio coordina la integración local de las tres aplicaciones GameBook y documenta sus límites de despliegue. No contiene código fuente de las aplicaciones.

## Compose local

### Requisitos y estructura de clones

Instala Docker Desktop con Docker Compose v2. Clona los cuatro repositorios como directorios hermanos para que las rutas de build de `compose.yaml` se resuelvan:

```text
Portfolio/
├── GameBook.System/
├── GameBook.Frontend/
├── GameBook.Microservice.AuthUser/
└── GameBook.Microservice.Game/
```

Compose construye cada aplicación desde su repositorio hermano mediante el `Dockerfile` correspondiente. PostgreSQL usa la imagen `postgres:16-alpine`; un script de inicialización crea los esquemas `auth` y `game`. Los servicios son `postgres`, `authuser`, `game` y `frontend`, con comprobaciones de salud y dependencias de arranque.

| Servicio   | Dirección local         | Función                                       |
| ---------- | ----------------------- | --------------------------------------------- |
| Frontend   | `http://localhost:3000` | Aplicación web Next.js                        |
| AuthUser   | `http://localhost:3001` | API de cuentas y sesiones; Swagger en `/docs` |
| Game       | `http://localhost:3002` | API de favoritos; Swagger en `/docs`          |
| PostgreSQL | `localhost:5432`        | Base local para los esquemas `auth` y `game`  |

### Configuración

Copia `.env.example` a `.env` en `GameBook.System` y configura valores locales privados. No subas `.env` al repositorio. La plantilla documenta los nombres de configuración sin incluir credenciales.

| Nombres de variables                                                 | Uso                                                                                    |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_PORT` | Contenedor PostgreSQL local y puerto del host                                          |
| `AUTHUSER_PORT`, `GAME_PORT`, `FRONTEND_PORT`                        | Cambios opcionales de puertos del host                                                 |
| `AUTH_DATABASE_URL`, `GAME_DATABASE_URL`                             | Conexiones de runtime dirigidas al esquema de cada servicio                            |
| `JWT_PRIVATE_KEY`, `JWT_PUBLIC_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE`    | Firma y verificación RS256 local entre AuthUser y Game                                 |
| `AUTHUSER_URL`, `CORS_ALLOWED_ORIGINS`                               | Conexión interna de Game a AuthUser y lista de orígenes permitidos                     |
| `NEXT_PUBLIC_AUTHUSER_URL`, `NEXT_PUBLIC_GAME_URL`                   | Direcciones de servicio del navegador incluidas en la imagen Frontend durante el build |
| `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET`                               | Credenciales server-only de la aplicación Twitch/IGDB para el catálogo público         |

Usa un par de claves RS256 privado para desarrollo y credenciales de prueba de IGDB/Twitch. No incluyas la clave privada de firma ni el secreto del proveedor en el control de versiones. Si usas Neon en lugar de PostgreSQL local, configura explícitamente las URLs de runtime del ambiente correspondiente y mantén privadas sus credenciales.

Los valores predeterminados de Compose usan una cuenta PostgreSQL solo local y URLs separadas por esquema. Las conexiones de runtime desplegadas en Neon usan el rol limitado asignado al esquema de cada servicio.

### Primer inicio

Desde el directorio `GameBook.System`, construye e inicia todos los servicios y consulta su estado:

```bash
docker compose up --build -d
docker compose ps
```

Abre el Frontend en `http://localhost:3000`. AuthUser y Game exponen comprobaciones de salud en `/health` e interfaz Swagger en `/docs`. El catálogo necesita valores IGDB/Twitch; los flujos de cuenta y favoritos requieren el par de claves local y las bases de datos de los servicios inicializadas.

Compose construye las imágenes de las aplicaciones e inicializa los esquemas de base de datos, pero **no ejecuta migraciones Prisma**. Las migraciones se administran mediante los workflows dedicados de GitHub Actions en los repositorios AuthUser y Game, con credenciales exclusivas de migración. No agregues URLs de migración a Compose ni ejecutes migraciones como parte del build de imagen o el arranque del servicio. Consulta la [guía de desarrollo de AuthUser](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/docs/development.md#database-and-migrations) y la [guía de desarrollo de Game](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/docs/development.md#database-and-migrations) para conocer la política y los ambientes.

### Detener y restablecer

Detén los contenedores y conserva los datos locales:

```bash
docker compose down
```

Para eliminar el volumen local de base de datos y comenzar sin datos, ejecuta:

```bash
docker compose down --volumes
```

El siguiente `docker compose up --build -d` recrea PostgreSQL y sus esquemas iniciales. También puedes reconstruir las imágenes después de cambiar una aplicación ejecutando de nuevo el comando de primer inicio.

## Resumen de despliegue

Cada aplicación se despliega desde su rama `main`. Frontend funciona en Vercel; AuthUser y Game, en Render. Los enlaces a Swagger de los backends están en el [README del sistema](../README.es.md#demo-y-documentación-de-api). El nivel gratuito de Render puede demorar la primera solicitud mientras se activa un backend.

Neon tiene ambientes `develop` y `production` separados, con credenciales distintas. Los roles de runtime de las aplicaciones están limitados al esquema de su servicio. Las credenciales de migración son distintas y se usan en los workflows dedicados de GitHub Actions; las URLs de migración no se configuran en Vercel, Render ni Compose.

GitHub Pages publica los diagramas interactivos del sistema desde el directorio `architecture/`. Abre el [índice de diagramas](https://carlossv923.github.io/GameBook.System/) o la [arquitectura general en español](https://carlossv923.github.io/GameBook.System/GameBook.System-architecture-es.html).

La operación de cada aplicación está documentada en su repositorio:

- [Guía de desarrollo de Frontend](https://github.com/CarlosSV923/GameBook.Frontend/blob/main/docs/development.es.md)
- [Guía de desarrollo de AuthUser](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/docs/development.es.md)
- [Guía de desarrollo de Game](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/docs/development.es.md)

## Actualizar este repositorio

Este repositorio contiene solo documentación. Realiza los cambios documentales en una rama de funcionalidad y abre un pull request a `main`. El registro de tareas SDD del repositorio indica el alcance asignado y las dependencias requeridas; coordina con la persona responsable las actualizaciones del estado de la tarea.
