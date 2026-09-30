# GameBook.System

[Read this documentation in Spanish](README.es.md)

This README is the single English general document for the documentation-only repository. It describes the implementation that was present on the `main` branch of the three application repositories when the documentation staging area was prepared on 2026-09-29. It is an implementation record, not a replacement for the original requirements. The SDD files in `specs/`, `plan/`, and `tasks/` remain the historical traceability record.

## 1. Purpose and scope

GameBook is a small portfolio web system for discovering video games and managing a personal list of favorites. Visitors can browse an IGDB-backed public catalog without creating an account. Authenticated users can register, sign in, maintain a session, filter and page through their favorites, save or remove games, inspect details, change their password, and logically disable their account.

The production topology consists of three independent public repositories:

| Component | Responsibility | Main implementation |
| --- | --- | --- |
| [`GameBook.Frontend`](https://github.com/CarlosSV923/GameBook.Frontend) | Browser experience, account/session UI, catalog, favorites, and server-side IGDB proxy | Next.js, React, TypeScript, Axios, RxJS |
| [`GameBook.Microservice.AuthUser`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser) | Registration, login, session validation, password changes, JWT signing, and logical account disabling | NestJS, TypeScript, Prisma, PostgreSQL |
| [`GameBook.Microservice.Game`](https://github.com/CarlosSV923/GameBook.Microservice.Game) | Favorite persistence, filtering, suggestions, and favorite snapshot updates | NestJS, TypeScript, Prisma, PostgreSQL |

The services share a PostgreSQL deployment but use separate database schemas. The browser never receives the IGDB/Twitch application credentials.

## 2. Implemented architecture

The validated English Archify artifact for the final application topology is preserved in [`architecture/`](architecture/):

- [Architecture diagram](https://carlossv923.github.io/GameBook.System/GameBook.System-architecture-en.html)

![Dark GameBook system architecture](architecture/GameBook.System-architecture-en.visual-check.1440x900.dark.png)

The diagram shows the frontend, AuthUser, Game, Neon/PostgreSQL, Twitch OAuth, IGDB, JWT/session validation, and the main relationships between them. Archify's fixed viewer controls remain in English.

At runtime the system is organized as four cooperating boundaries:

1. The browser renders the Next.js application and keeps the GameBook access token in `sessionStorage`.
2. Next.js exposes server routes under `/api/igdb/*` and uses server-only credentials to obtain a Twitch application token and call IGDB.
3. AuthUser owns the `auth` schema, signs RS256 JWTs, and is the authority for user identity and persisted session validity.
4. Game owns the `game` schema, verifies the JWT locally, asks AuthUser to validate the session, and persists favorites using the UUID in the validated token.

## 3. Main user flows

### 3.1 Public catalog

1. A visitor opens the Next.js home page.
2. The browser calls the local frontend route `/api/igdb/games`.
3. The server-side IGDB client obtains or reuses a Twitch `client_credentials` application token.
4. The server calls IGDB, maps the provider response to the frontend contract, and returns catalog cards.
5. Name, platform, and release-year filters are sent as query parameters. Pagination uses `limit` and `offset`.
6. The adapter rejects incomplete catalog records when the card would be missing a required displayed value such as cover, release date, rating, or platform.

The browser does not call IGDB or Twitch directly. Detail, game-name suggestions, and platform suggestions use their corresponding Next.js routes. Provider failures are mapped to stable frontend error codes instead of exposing provider credentials or raw internals.

### 3.2 Registration and sign-in

1. The browser asks the AuthUser client to perform a `GET /health` readiness check.
2. Once AuthUser responds with HTTP `200`, the client sends `POST /v1/auth/register` or `POST /v1/auth/login`.
3. AuthUser validates the request, reads or writes the user in the `auth` schema, and returns the documented response.
4. On login, AuthUser signs an RS256 JWT containing the user UUID, session version, issuer, audience, issue time, and one-hour expiry.
5. The frontend stores the token in `sessionStorage` and validates it through `GET /v1/auth/session` before considering the session authenticated.

Disabled accounts are rejected on login with `ACCOUNT_DISABLED`. Registration with a disabled email is rejected with the same semantic code. The frontend treats revoked, expired, invalid, and disabled sessions as discardable and clears the stored token.

### 3.3 Favorites

1. A signed-in user opens the catalog or favorites view.
2. Before each AuthUser or Game operation, the corresponding frontend client performs the service healthcheck.
3. The frontend sends `Authorization: Bearer <token>` to Game.
4. Game verifies the RS256 signature and the issuer, audience, expiry, and UUID claims locally.
5. Game calls AuthUser's current-session endpoint to confirm that the token's session version is still valid and that the account is not disabled.
6. Game uses the validated UUID as the ownership key. It never accepts a client-supplied user ID for favorite ownership.
7. Prisma reads or writes the `game` schema. Filtering by name, platform, and release-year range is combined with AND semantics.

Create, snapshot update, and delete are explicit mutations. List and suggestion operations return only records owned by the authenticated subject. A revoked or disabled session is rejected before the favorite use case runs.

### 3.4 Password change and account disabling

`PATCH /v1/users/me/password` validates the current password and the new password policy. The password hash replacement increments `sessionVersion` atomically, which invalidates every previously issued JWT, including the token used for the request. The frontend clears the session and asks the user to sign in again.

`DELETE /v1/users/me` is a logical disable operation. It sets `isDisabled` to `true`, increments `sessionVersion`, preserves the user and favorite data, and invalidates existing tokens. There is no reactivation or physical purge operation in the MVP.

## 4. Implementation boundaries

### 4.1 Frontend

The Next.js App Router separates page composition, feature behavior, shared API types, and server-only provider integration:

- `src/app/` contains pages and the `/api/igdb/*` route handlers.
- `src/features/api/` contains typed clients for AuthUser, Game, and the frontend IGDB routes.
- `src/features/auth/`, `catalog/`, `favorites/`, `profile/`, and `preferences/` contain user-facing behavior.
- `src/shared/api/` contains the Axios transport, RxJS request pipelines, contracts, errors, healthchecks, and mocks.
- `src/server/igdb/` contains server-only Twitch/IGDB clients, query mapping, response validation, rate limiting, and provider error mapping.

The shared HTTP layer uses Axios for requests and RxJS for observable composition. Normal GET requests can retry transient failures twice with short delays. Mutations do not use that generic retry policy. AuthUser and Game clients disable generic operation retries and first wait for their service healthcheck.

The healthcheck pipeline uses `GET /health`, waits up to 15 seconds per attempt, and permits 15 additional healthcheck retries. The first failed attempt can show the bilingual Render warm-up alert; a later HTTP `200` recovery dismisses it before its normal timeout. An aborted request stops the observable pipeline without starting the protected operation.

Relevant evidence: [`auth-user-client.ts`](https://github.com/CarlosSV923/GameBook.Frontend/blob/main/src/features/api/auth-user-client.ts), [`game-client.ts`](https://github.com/CarlosSV923/GameBook.Frontend/blob/main/src/features/api/game-client.ts), [`healthcheck.ts`](https://github.com/CarlosSV923/GameBook.Frontend/blob/main/src/shared/api/healthcheck.ts), and [`http.ts`](https://github.com/CarlosSV923/GameBook.Frontend/blob/main/src/shared/api/http.ts).

### 4.2 AuthUser

AuthUser follows a DDD-oriented NestJS structure:

- `src/api/` contains controllers, request DTOs, validation, CORS, request IDs, exception mapping, health, and Swagger/OpenAPI.
- `src/application/` contains use cases, ports, dependency tokens, password and JWT boundaries.
- `src/domain/` contains the `User` aggregate, email value object, password policy, and domain errors.
- `src/infrastructure/` contains runtime configuration, RS256 cryptography, scrypt password hashing, and Prisma persistence.

The persistence model includes the user UUID, identity fields, password hash, `sessionVersion`, and `isDisabled`. Prisma migrations are separate from runtime startup and are executed through the dedicated migration workflow with migration-only credentials. Runtime access uses the AuthUser schema role.

AuthUser exposes `GET /health`, `POST /v1/auth/register`, `POST /v1/auth/login`, `GET /v1/auth/session`, `PATCH /v1/users/me/password`, and `DELETE /v1/users/me`. Swagger UI and its JSON document are available under `/docs` and `/docs/openapi.json`.

Relevant evidence: [`api.module.ts`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/src/api/api.module.ts), [`login-user.ts`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/src/application/use-cases/login-user.ts), [`validate-session.ts`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/src/application/use-cases/validate-session.ts), and [`schema.prisma`](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/blob/main/src/infrastructure/persistence/prisma/schema.prisma).

### 4.3 Game

Game uses the same major NestJS boundaries:

- `src/api/` contains favorite controllers, DTO validation, the JWT guard, CORS, request IDs, exception mapping, health, and Swagger/OpenAPI.
- `src/application/` contains favorite use cases and ports.
- `src/domain/` contains the `Favorite` aggregate, platform value behavior, repository port, and domain validation errors.
- `src/infrastructure/` contains the AuthUser session client, RS256 verification, runtime configuration, Prisma client, and favorite repository.

Game exposes `GET /health`, `GET /v1/favorites`, `GET /v1/favorites/suggestions`, `POST /v1/favorites`, `PATCH /v1/favorites/:igdbId/snapshot`, and `DELETE /v1/favorites/:igdbId`. Protected routes declare Bearer JWT security in OpenAPI. If AuthUser cannot be reached, Game returns `503` without executing the favorite use case.

The Prisma model uses a composite key of `(userId, igdbId)` for favorites and a related platform table. Both tables are explicitly assigned to the `game` PostgreSQL schema. The migration workflow uses a separate migration-only database variable and does not run migrations as part of the application build or startup.

Relevant evidence: [`favorites.controller.ts`](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/src/api/favorites/favorites.controller.ts), [`jwt-auth-guard.ts`](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/src/api/auth/jwt-auth-guard.ts), [`auth-user-session-client.ts`](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/src/infrastructure/auth/auth-user-session-client.ts), and [`schema.prisma`](https://github.com/CarlosSV923/GameBook.Microservice.Game/blob/main/src/infrastructure/persistence/prisma/schema.prisma).

## 5. API and security model

The two NestJS services expose Swagger UI and OpenAPI JSON without authentication so the portfolio contracts can be inspected. Protected operations declare HTTP Bearer JWT security. CORS accepts only the configured allow-list; wildcard origins are discarded by the runtime option builder.

AuthUser signs JWTs with RS256. Game receives the corresponding public key and independently checks the algorithm, signature, UUID subject, session version claim, issuer, audience, and expiry. Game then calls AuthUser to validate the persisted session version and disabled state. This two-step check allows Game to reject tokens that are cryptographically valid but have been revoked or belong to a disabled account.

Passwords are hashed and never returned. Private keys, public production values, database URLs, OAuth secrets, access tokens, and real credentials are environment-managed and are not part of this documentation snapshot.

## 6. Testing and observability

All three repositories use `pnpm` and include automated tests:

- Frontend unit tests cover API clients, Axios/RxJS retry behavior, healthcheck timeout/recovery, catalog and favorites state, authentication, localization, preferences, and UI states.
- AuthUser unit and integration tests cover domain rules, configuration, cryptography, registration, login, session validation, password changes, logical disabling, Prisma access, CORS, request IDs, and OpenAPI/bootstrap behavior.
- Game unit and integration tests cover the favorite domain and use cases, Prisma persistence, JWT verification, AuthUser session failures, ownership isolation, revocation, healthcheck, OpenAPI, and HTTP behavior.

The repositories keep CI, release-please, and migration workflows under `.github/workflows/`. The backend request pipeline adds a request ID, validates input, maps known errors to contract responses, and logs request completion. Logs are intended to be readable and correlatable while excluding passwords, JWTs, private keys, API keys, and database credentials.

## 7. Deployment and runtime configuration

The production services are configured outside the application repositories:

- Frontend: [Vercel](https://gamebook-frontend.vercel.app/), built from `main`.
- AuthUser: [Render Swagger](https://gamebook-microservice-authuser.onrender.com/docs), built from `main`.
- Game: [Render Swagger](https://gamebook-microservice-game.onrender.com/docs), built from `main`.
- PostgreSQL: Neon environments with separate `auth` and `game` schemas and environment-specific credentials.
- Provider: Twitch application OAuth obtains the token used by the server-only IGDB adapter.

Production service documentation is available at [AuthUser Swagger](https://gamebook-microservice-authuser.onrender.com/docs), [AuthUser OpenAPI](https://gamebook-microservice-authuser.onrender.com/docs/openapi.json), [Game Swagger](https://gamebook-microservice-game.onrender.com/docs), and [Game OpenAPI](https://gamebook-microservice-game.onrender.com/docs/openapi.json).

Runtime variables are supplied by the hosting environments. Direct database URLs used by Prisma migration workflows are not runtime variables and are not configured in Render or Vercel. The frontend keeps `IGDB_CLIENT_ID` and `IGDB_CLIENT_SECRET` server-only; browser-facing service URLs are the only public frontend configuration values.

## Local integration with Docker Compose

`compose.yaml` is the local integration entry point for the three sibling application repositories. Clone them next to `GameBook.System`:

```text
Portfolio/
├── GameBook.System/
├── GameBook.Frontend/
├── GameBook.Microservice.AuthUser/
└── GameBook.Microservice.Game/
```

Docker Desktop with Compose v2 is required. Copy `.env.example` to a private `.env` file in `GameBook.System` and add the local RS256 values and IGDB/Twitch test credentials. The Compose stack starts a local `postgres:16-alpine` container by default, creates the `auth` and `game` schemas, and connects AuthUser and Game through the internal `postgres` hostname. No Neon account is required for this flow. Neon can be used only by explicitly overriding `AUTH_DATABASE_URL` and `GAME_DATABASE_URL` in the private `.env` file.

Build and start the stack:

```bash
docker compose up --build -d
docker compose ps
```

The first start creates the schemas but does not run Prisma migrations implicitly. Apply the local migrations explicitly when the database volume is new:

```bash
docker compose run --rm -e AUTH_DATABASE_DIRECT_URL=postgresql://gamebook:gamebook-local@postgres:5432/gamebook?schema=auth authuser pnpm db:migrate:deploy
docker compose run --rm -e GAME_DATABASE_DIRECT_URL=postgresql://gamebook:gamebook-local@postgres:5432/gamebook?schema=game game pnpm db:migrate:deploy
```

The service URLs are `http://localhost:3000` (Frontend), `http://localhost:3001` (AuthUser), `http://localhost:3002` (Game), and `localhost:5432` (PostgreSQL). AuthUser and Game expose `/health` and `/docs`; the Compose healthchecks wait for those services before starting dependants. Stop the stack with `docker compose down`; add `-v` only when the local PostgreSQL data should also be removed.

## 8. Release history

Each application repository uses release-please to manage its versioned releases independently. `GameBook.System` is documentation-only and does not use release-please, `pnpm`, Vercel, or a `develop` branch.

| Repository | Releases | Production role |
| --- | --- | --- |
| GameBook.Frontend | [View releases](https://github.com/CarlosSV923/GameBook.Frontend/releases) | Vercel frontend |
| GameBook.Microservice.AuthUser | [View releases](https://github.com/CarlosSV923/GameBook.Microservice.AuthUser/releases) | Render identity service |
| GameBook.Microservice.Game | [View releases](https://github.com/CarlosSV923/GameBook.Microservice.Game/releases) | Render favorites service |

## 9. Deliberate limitations

- The MVP does not translate titles, descriptions, genres, or platform names returned by IGDB.
- Account disabling is logical and irreversible through the current UI; the account and its favorites are retained and there is no reactivation workflow.
- Game depends on AuthUser for persisted session validation, so an unavailable AuthUser blocks protected Game operations even when a JWT is otherwise well formed.
- AuthUser and Game run on Render's free plan. When those backend services have been idle, Render may suspend them and their next request can take longer while they wake up; the frontend healthcheck wait, retries, and warm-up message make that delay clearer but cannot remove the provider's cold-start latency.
- IGDB availability, OAuth token validity, provider rate limits, and the frontend's local API contract remain external failure boundaries.
- The system is intentionally split into independent repositories rather than a shared code package or monorepo.

## 10. Traceability sources

The implementation record is supported by the staged `contracts/`, `environments/`, `specs/`, `plan/`, and `tasks/` directories, the three public application repositories, and the validated Archify artifacts under `architecture/`.

| Area | Reference | Purpose |
| --- | --- | --- |
| Specification | [MVP specification](specs/mvp.md) | Original Spanish SDD requirements and acceptance scope. |
| Plan | [MVP plan](plan/mvp.md) | Original Spanish implementation strategy and quality gates. |
| Tasks | [MVP task registry](tasks/mvp.md) | Coordinated execution state and evidence. |
| Contracts | [Contract baseline v3](contracts/contract-baseline-v3.md) | Current contract baseline; older contract documents are retained as historical traceability. |
| Environments | [Local-first deployment policy](environments/deployment-policy-local-first.md) | Current environment and deployment policy. |
| Project instructions | [AGENTS.md](AGENTS.md) and [CLAUDE.md](CLAUDE.md) | Preserved project operating instructions in Spanish. |

The existing SDD, contract, environment, and project-instruction documents remain in Spanish according to the documented GB-014.03 decision. This README is the only English general documentation file in the repository; its complete Spanish counterpart is [README.es.md](README.es.md).
