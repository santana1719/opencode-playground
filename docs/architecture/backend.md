# Backend

## Backend (`apps/api`)

### Folder structure

| Path | Responsibility |
| --- | --- |
| `src/server.ts` | Creates and exports `app`, installs `express.json()`, keeps `GET /api/version`, mounts the Demo Data router, installs error middleware, defines `serveFrontend` and starts listening only in the `import.meta.main` block. |
| `src/routes/demo-entries.ts` | Defines the read-only `GET /api/demo/entries` route. No write routes. |
| `src/repositories/demo-entries.ts` | Repository interface, its `pg` raw-SQL Neon implementation and an in-memory fake for tests. Both read paths convert `Date` values from `TIMESTAMPTZ(3)` columns to the ISO string API shape. |
| `src/types/demo-entries.ts` | Explicit types shared by the route and repository. |
| `db/001_demo_entries.sql` | Checked-in Demo Data schema, applied manually before `npm run db:seed`; outside the compiled `src/` tree. |
| `scripts/seed.ts` | Manual `npm run db:seed` command: reads `data/seed.json`, uses its immutable `seedKey` for upserts and a mirror of the §6.5 Reason→Strategy mapping to convert labels to codes and dates to `TIMESTAMPTZ` values. |

There is no service layer and no write route in the MVP. Do not add folders or layers for future work; add a new route or repository only when a requirement needs it, and record significant decisions in `decisions.md`.

### Runtime and deployment

- In development Node.js runs the TypeScript file directly (`node --watch`); `npm run build` compiles it to `dist/`.
- In production (`npm start` runs `dist/server.js`) Express also serves the built frontend from `apps/web/dist` (`serveFrontend`). Every non-`/api` GET request gets `index.html`. The port comes from `PORT`, then `API_PORT`, then 3001.
- `export const app`, `serveFrontend` and the `import.meta.main` check stay in `server.ts` so tests can import the app without starting a server and production can serve the frontend.

## API endpoints

| Method | Path | Description | Response |
| --- | --- | --- | --- |
| GET | `/api/version` | API name and version | `200 { name, version }` |
| GET | `/api/demo/entries` | Fictional Demo Data from Neon, seeded from `data/seed.json` | `200 { entries }`; `500 { error: { code: "DEMO_DATA_UNAVAILABLE", message: "Could not load Demo Data." } }` |

## API conventions

- All endpoints live under `/api/...` so the Vite proxy forwards them; there are no write endpoints in the MVP.
- Requests and responses use JSON. Success bodies are objects with explicit keys: `{ name, version }` for `GET /api/version` and `{ entries }` for `GET /api/demo/entries`. Each `entries` item uses the `ProcrastinationEntry` fields plus an optional `strategyAttempt`, as defined in `docs/requirements.md`.
- Errors use `{ error: { code, message } }`. The Demo Data failure must be `500 { error: { code: "DEMO_DATA_UNAVAILABLE", message: "Could not load Demo Data." } }`.
- The only failure defined by the MVP API contract is Demo Data unavailability: respond with `500 { error: { code: "DEMO_DATA_UNAVAILABLE", message: "Could not load Demo Data." } }`. Never leak stack traces to clients.
- Route handlers call the repository interface only; SQL stays in `repositories/demo-entries.ts`.

## Backend rules

- Both MVP endpoints are parameterless GETs, so there is no request input to validate and no per-resource lookup. Do not add `400` or `404` paths that the API contract does not define; never leak stack traces to clients.
- Use one consistent JSON response and error shape across endpoints, and document it in `README.md`.
- Install a single error-handling middleware in `server.ts` now, as recorded in `decisions.md`. Express 5 forwards errors thrown in `async` handlers to it; the Demo Data route uses it to return the exact JSON failure from the API contract.
- Demo Data is read-only: there are no personal-write endpoints. Personal Procrastination Entries, Strategy Attempts and Strategy Feedback are never sent to the API.
- Demo Data database access sits behind the repository layer in `repositories/demo-entries.ts`, separating it from API logic, so tests can substitute an in-memory fake and the SQL implementation stays localized.
- Use the `pg` driver with explicit parameterized `SELECT` statements and read `DATABASE_URL` from the environment. Do not add an ORM or query builder for the MVP.
- Neon PostgreSQL is the agreed storage for fictional Demo Data only. Do not add a database, ORM, authentication or new dependency for personal data.
- Apply `db/001_demo_entries.sql` manually to Neon before `npm run db:seed`; the repository and seed import use only the tables and columns defined there (see `data-model.md`).
- Seeding `data/seed.json` is manual and idempotent, not an application-startup step. `npm run db:seed` runs `apps/api/scripts/seed.ts`, which copies each fixture row's immutable `seedKey` into `seed_key`, upserts entries on that column and attempts on `entry_id`; reruns do not create duplicates or change existing `id`s (see `decisions.md`). `seedKey` is import metadata and never appears in API responses.

## Tests

- Import `app` from `server.ts` with Supertest; do not open a network port in unit tests.
- Test routes with the in-memory repository fake rather than a live Neon database; the fake must return ISO 8601 strings with milliseconds, matching the real repository's `Date` → `toISOString()` mapping.
- Cover the success path and the exact Demo Data failure response with Supertest. Do not add invalid-input or not-found tests for endpoints that have neither.
