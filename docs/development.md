# Development

This repository coordinates local integration for the three GameBook applications and documents their deployment boundaries. It does not contain application source code.

## Local Compose

### Requirements and checkout layout

Install Docker Desktop with Docker Compose v2. Clone all four repositories as siblings so the build contexts in `compose.yaml` resolve:

```text
Portfolio/
├── GameBook.System/
├── GameBook.Frontend/
├── GameBook.Microservice.AuthUser/
└── GameBook.Microservice.Game/
```

Compose builds each app from its sibling checkout using that repository's `Dockerfile`. PostgreSQL uses the `postgres:16-alpine` image; an initialization script creates the `auth` and `game` schemas. The services are `postgres`, `authuser`, `game` and `frontend`, with health checks and startup dependencies.

| Service    | Host address            | Purpose                                          |
| ---------- | ----------------------- | ------------------------------------------------ |
| Frontend   | `http://localhost:3000` | Next.js web app                                  |
| AuthUser   | `http://localhost:3001` | Account and session API; Swagger at `/docs`      |
| Game       | `http://localhost:3002` | Favorites API; Swagger at `/docs`                |
| PostgreSQL | `localhost:5432`        | Local database for the `auth` and `game` schemas |

### Configuration

Copy `.env.example` to `.env` in `GameBook.System` and set private local values. Do not commit `.env`. The template documents the configuration names without providing credentials.

| Variable names                                                       | Used by                                                                 |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_PORT` | Local PostgreSQL container and host port                                |
| `AUTHUSER_PORT`, `GAME_PORT`, `FRONTEND_PORT`                        | Optional host port overrides                                            |
| `AUTH_DATABASE_URL`, `GAME_DATABASE_URL`                             | Runtime database connections targeting each service's schema            |
| `JWT_PRIVATE_KEY`, `JWT_PUBLIC_KEY`, `JWT_ISSUER`, `JWT_AUDIENCE`    | Local RS256 signing and verification between AuthUser and Game          |
| `AUTHUSER_URL`, `CORS_ALLOWED_ORIGINS`                               | Game's internal AuthUser connection and browser origin allowlist        |
| `NEXT_PUBLIC_AUTHUSER_URL`, `NEXT_PUBLIC_GAME_URL`                   | Browser-facing service URLs baked into the Frontend image at build time |
| `IGDB_CLIENT_ID`, `IGDB_CLIENT_SECRET`                               | Server-only Twitch/IGDB application credentials for the public catalog  |

Use a private development RS256 key pair and test IGDB/Twitch credentials. Keep the signing private key and provider secret out of source control. If using Neon instead of the default local PostgreSQL, explicitly configure the runtime URLs for the intended environment and keep its credentials private.

The Compose defaults use a local-only PostgreSQL account and separate schema URLs. Deployed Neon runtime connections use the limited role assigned to each service schema.

### First run

From the `GameBook.System` directory, build and start all services, then check their state:

```bash
docker compose up --build -d
docker compose ps
```

Open the Frontend at `http://localhost:3000`. AuthUser and Game expose health checks at `/health` and Swagger UI at `/docs`. The frontend catalog needs IGDB/Twitch values; account and favorites flows need the local signing pair and initialized service databases.

Compose builds the application images and initializes database schemas, but **does not run Prisma migrations**. Migrations are managed through the dedicated GitHub Actions workflows in the AuthUser and Game repositories, using their migration-only credentials. Do not add migration URLs to Compose or run migrations as part of image build or service startup. See the respective [AuthUser development guide](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/docs/development.md#database-and-migrations) and [Game development guide](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/docs/development.md#database-and-migrations) for migration policy and environment details.

### Stop and reset

Stop the containers while keeping local database data:

```bash
docker compose down
```

To remove the local database volume and start with empty data, use:

```bash
docker compose down --volumes
```

The next `docker compose up --build -d` recreates PostgreSQL and its initial schemas. The images can also be rebuilt after changing an application checkout by running the first-run command again.

## Deployment overview

Each application is deployed from its `main` branch. Frontend runs on Vercel; AuthUser and Game run on Render. The backend Swagger pages are linked from the [system README](../README.md#live-demo-and-api-docs). The Render free tier may delay the first request while a backend warms up.

Neon has separate `develop` and `production` environments and credentials. Application runtime roles are limited to their service schema. Migration credentials are separate and used by the dedicated GitHub Actions workflows; migration URLs are not configured in Vercel, Render or Compose.

GitHub Pages hosts the interactive system diagrams from the `architecture/` directory. Open the [diagram index](https://carlossv923.github.io/GameBook.System/) or the [English system architecture](https://carlossv923.github.io/GameBook.System/GameBook.System-architecture-en.html).

Application operations are documented in their repositories:

- [Frontend development guide](https://github.com/CarlosSV923/GameBook.Frontend/blob/main/docs/development.md)
- [AuthUser development guide](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/docs/development.md)
- [Game development guide](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/docs/development.md)

## Updating this repository

This is a documentation-only repository. Make documentation changes on a feature branch and submit a pull request to `main`. The repository's SDD task record identifies the assigned scope and required dependencies; coordinate task status updates with the project owner.
