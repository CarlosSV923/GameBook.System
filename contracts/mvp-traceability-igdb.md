# MVP requirements traceability

This is the IGDB revision of the **GB-002.01** traceability matrix. It maps the 16 acceptance criteria in [`../specs/mvp.md`](../specs/mvp.md) to operations and data owners in [`../plan/mvp.md`](../plan/mvp.md). The current field-level contracts are [`game-igdb.openapi.yaml`](game-igdb.openapi.yaml) and [`igdb-mapping.md`](igdb-mapping.md); the RAWG matrix is historical only.

## Criteria to operations

| # | Required behavior | Operations or boundary involved | Data owner(s) | Relevant error cases |
| --- | --- | --- | --- | --- |
| 1 | Public catalog ordered by IGDB `total_rating`, three combined filters, infinite scroll, and detail | Next.js server routes for catalog, platforms, and detail; public UI pagination/filter state | IGDB owns catalog data; Next.js owns proxy parameters and public UI state | IGDB unavailable, rate-limit response, malformed or missing fields, empty result |
| 2 | Visitor trying to save receives login/register choices and does not save | Frontend session guard and navigation to AuthUser register/login; no Game write | Frontend owns the unauthenticated interaction; Game owns favorites | No protected request is sent; UI must preserve the public state |
| 3 | Account creation and login work with the requested fields | `POST /v1/auth/register`; `POST /v1/auth/login` | AuthUser owns account identity and credential verification | `400` invalid fields/password/confirmation; `409` email already used; generic credential failure on login |
| 4 | Authenticated user confirms and saves a favorite | Frontend confirmation; `POST /v1/favorites` with the AuthUser Bearer JWT | Game owns the favorite and its stored snapshot; Frontend owns confirmation state | `401` missing/invalid/revoked JWT; `409` duplicate favorite; `400` invalid snapshot; `503` AuthUser unavailable |
| 5 | Authenticated user lists, filters, and opens favorite details | `GET /v1/favorites`; `GET /v1/favorites/suggestions`; Next.js detail route | Game owns the personal list and filterable snapshot; IGDB owns full detail | `401` invalid session; `400` invalid filters/page; `503` AuthUser unavailable; IGDB detail failure |
| 6 | Authenticated user confirms deletion from card or modal | Frontend confirmation; `DELETE /v1/favorites/{igdbId}` | Game owns deletion of the authenticated user’s favorite | `401` invalid session; `404` own favorite not found; `503` AuthUser unavailable |
| 7 | Profile changes only the password | `PATCH /v1/users/me/password` | AuthUser owns password hash and session version | `400` invalid new password; `401` invalid JWT or current password; persistence failure must not partially update hash/version |
| 8 | Logout returns to public catalog | Frontend clears `sessionStorage`, private state, and navigates to public view | Frontend owns browser session state; AuthUser remains account authority | Local cleanup must be safe if no token exists; no new backend capability is introduced |
| 9 | Protected operations cannot read or modify another account | Bearer guard on AuthUser and every Game operation; identity comes from JWT `sub` | AuthUser owns identity/session validity; Game scopes every query and mutation by `sub` | `401` absent/invalid/expired/revoked token; no cross-user `404`/`409` disclosure; dependency failure `503` |
| 10 | Expired JWT closes session and requires login again | AuthUser session validation; Game local JWT validation plus AuthUser session check; Frontend `401` handling | AuthUser owns token/session validity; Frontend owns session removal | `401` expired/invalid token; no automatic refresh; public catalog remains available |
| 11 | IGDB failure preserves favorite; successful detail updates basic snapshot | Next.js detail route; conditional `PATCH /v1/favorites/{igdbId}/snapshot` after successful IGDB response | IGDB owns current detail; Game owns the local basic snapshot | IGDB unavailable/quota error leaves local favorite unchanged; `401`/`404`/`400` on snapshot update |
| 12 | Theme follows system initially and manual choice persists | Frontend theme preference boundary (system media query plus local persistence) | Frontend/browser owns theme preference; no backend operation | No network dependency; preference failure must not close session or remove filters |
| 13 | English default, EN/ES switch, persistence, IGDB text unchanged | Frontend translation catalog and language preference boundary | Frontend owns system text and selected language; IGDB owns external text | No network dependency; missing translation falls back safely without translating IGDB |
| 14 | Both microservices expose inspectable Swagger/OpenAPI | AuthUser and Game OpenAPI documents and Swagger UI for all exposed operations | Each service owns its API documentation; contracts are coordinated by GB-002 | Documentation must identify auth/errors without secrets; malformed contract is a release blocker |
| 15 | Game accepts the AuthUser Bearer JWT and isolates by UUID | Game guard: local RS256 verification followed by AuthUser session validation on each protected route | AuthUser owns signing/session version; Game owns authorization boundary and favorites | `401` absent/invalid/expired/revoked; `503` when AuthUser cannot confirm validity; fail closed |
| 16 | Password change revokes all previous JWTs immediately | `PATCH /v1/users/me/password`; subsequent `GET /v1/auth/session` and Game guard checks | AuthUser owns atomic hash + session-version increment; Game consumes session validity; Frontend clears token | `401` for old token before expiry; failed change must not increment version or partially update credentials |

## Operation and data ownership inventory

| Boundary | Planned operations/data | Owner |
| --- | --- | --- |
| AuthUser HTTP API | Register, login, current session, password change; user UUID, normalized email, password hash, session version, JWT signing | `GameBook.Microservice.AuthUser` |
| Game HTTP API | Save, list/filter/page, suggestions, snapshot update, delete; favorite identity `(userId, igdbId)`, basic snapshot, platforms | `GameBook.Microservice.Game` |
| Next.js server | Catalog, detail, and platform proxy; Twitch Client ID/Secret, application token renewal, and safe IGDB query parameters | `GameBook.Frontend` server runtime |
| Frontend browser | View/filter/pagination state, confirmation state, theme/language preferences, JWT in `sessionStorage` | `GameBook.Frontend` |
| IGDB | External catalog, ratings, dates, platforms, images, and details | IGDB API |
| PostgreSQL | `auth` and `game` schemas with independent Prisma ownership | Neon database; each service migrates only its schema |

## Error and dependency inventory

These are the error categories to carry into the contract tasks; their exact stable codes and response shape are not frozen here.

| Category | Applies to | Required handling |
| --- | --- | --- |
| `400` validation | AuthUser inputs, Game snapshots/filters, Next.js proxy parameters | Reject before side effects; expose a stable code for frontend translation |
| `401` authentication/session | Bearer missing, malformed, invalid, expired, or revoked | Never read or mutate protected data; Frontend clears session and requests login |
| `404` own resource absent | Favorite snapshot update/delete or other own-resource lookup | Do not reveal another user’s resource existence |
| `409` uniqueness conflict | Registering an existing email or saving an existing `(userId, igdbId)` | Keep existing data; show a comprehensible duplicate result |
| `503` AuthUser dependency unavailable | Game protected operations after local token verification | Fail closed; do not read or modify favorites |
| IGDB authentication failure/rate limit/missing data | Next.js public catalog/detail/platform routes | Show loading/error/empty or missing-field states; never expose client secret or application token or invent data |

## Scope guard

GB-002.01 introduces no endpoint, field, filter, storage rule, or user-visible behavior. Any mismatch found while closing the contract subtasks must be resolved by updating the specification first, one decision at a time.
