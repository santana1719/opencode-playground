# Backend

### Backend (`apps/api`)

- The whole API is in `src/server.ts`. It exports the Express `app` (so tests can use it without a network port) and starts listening only when the file is run directly.
- The starter has one endpoint: `GET /api/version` → `{ "name": "api", "version": "0.1.0" }`. The MVP adds the read-only `GET /api/demo/entries` described below and in `docs/requirements.md`.
- In development Node.js runs the TypeScript file directly (`node --watch`); `npm run build` compiles it to `dist/`.
- In production (`npm start` runs `dist/server.js`) Express also serves the built frontend from `apps/web/dist` (`serveFrontend`). Every non-`/api` GET request gets `index.html`. The port comes from `PORT`, then `API_PORT`, then 3001.

## API endpoints

| Method | Path | Description | Response |
| --- | --- | --- | --- |
| GET | `/api/version` | API name and version | `200 { name, version }` |
| GET | `/api/demo/entries` | Fictional Demo Data from Neon, seeded from `data/seed.json` | `200 { entries }`; `500 { error: { code: "DEMO_DATA_UNAVAILABLE", message: "Could not load Demo Data." } }` |

## Backend rules

- All endpoints live under `/api/...` so the Vite proxy forwards them.
- The starter is intentionally a single file. When the application grows, introduce structure step by step (for example a router per resource, or a module for business logic) and record the decision in `decisions.md`. Do not add layers "for later".
- Validate request input (body, params, query) before using it. Respond with `400` and a clear JSON error for invalid input, `404` for missing resources, and never leak stack traces to clients.
- Use one consistent JSON response and error shape across endpoints, and document it in `README.md`.
- Express 5 forwards errors thrown in `async` handlers to error middleware; add a single error-handling middleware once you need it.
- `export const app`, `serveFrontend` and the `import.meta.main` check stay in `server.ts` so tests can import the app without starting a server and production can serve the frontend.
- Demo Data is read-only: there are no personal-write endpoints. Personal Procrastination Entries, Strategy Attempts and Strategy Feedback are never sent to the API.
- Demo Data database access sits behind a repository layer separating it from API logic, so tests can substitute an in-memory fake and the SQL implementation stays localized.
- Seeding `data/seed.json` is manual and idempotent, not an application-startup step; reruns must not create duplicates.
- Storage is an open decision. Do not add a database, ORM, authentication or new dependency for personal data.
