# Backend

## Backend (`apps/api`)

### Folder structure

| Path | Responsibility |
| --- | --- |
| `src/server.ts` | Creates and exports `app`, installs `express.json()`, keeps `GET /api/version`, mounts the Demo Data router, installs error middleware, defines `serveFrontend` and starts listening only in the `import.meta.main` block. |
| `src/routes/demo-entries.ts` | Defines the read-only `GET /api/demo/entries` route. No write routes. |
| `src/repositories/demo-entries.ts` | Repository interface, its `pg` raw-SQL Neon implementation and an in-memory fake for tests. |
| `src/types/demo-entries.ts` | Explicit types shared by the route and repository. |

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
- Use `400` for invalid input, `404` for missing resources and `500` for unexpected failures; never leak stack traces to clients.
- Route handlers call the repository interface only; SQL stays in `repositories/demo-entries.ts`.

## Backend rules

- Validate request input (body, params, query) before using it. Respond with `400` and a clear JSON error for invalid input, `404` for missing resources, and never leak stack traces to clients.
- Use one consistent JSON response and error shape across endpoints, and document it in `README.md`.
- Express 5 forwards errors thrown in `async` handlers to error middleware; add a single error-handling middleware once you need it.
- Demo Data is read-only: there are no personal-write endpoints. Personal Procrastination Entries, Strategy Attempts and Strategy Feedback are never sent to the API.
- Demo Data database access sits behind the repository layer in `repositories/demo-entries.ts`, separating it from API logic, so tests can substitute an in-memory fake and the SQL implementation stays localized.
- Use the `pg` driver with explicit parameterized `SELECT` statements and read `DATABASE_URL` from the environment. Do not add an ORM or query builder for the MVP.
- Neon PostgreSQL is the agreed storage for fictional Demo Data only. Do not add a database, ORM, authentication or new dependency for personal data.
- Seeding `data/seed.json` is manual and idempotent, not an application-startup step; reruns must not create duplicates.

## Tests

- Import `app` from `server.ts` with Supertest; do not open a network port in unit tests.
- Test routes with the in-memory repository fake rather than a live Neon database.
- Cover the success path, invalid input and not-found cases for every endpoint you add or change.
