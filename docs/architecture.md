# Architecture

This file records **how** the application is built and **why**. Keep it short and up to date: AI agents read it before every task (it is loaded automatically through `opencode.jsonc`), and so will anyone who reviews your project.

## Provided infrastructure

The starter contains infrastructure only, with no product features.

| Part | Location | Technology |
| --- | --- | --- |
| Frontend | `apps/web` | React 19, TypeScript, Vite, Tailwind CSS 4 |
| Backend | `apps/api` | Node.js, Express 5, TypeScript |
| Unit tests | `apps/*/src/**/*.test.ts(x)` | Vitest, React Testing Library, Supertest |
| End-to-end tests | `e2e/` | Playwright |
| Workspace | root `package.json` | npm workspaces |

### Frontend (`apps/web`)

- `src/main.tsx` renders `src/App.tsx`, the starter page.
- `src/api.ts` contains the functions that call the backend.
- Styling uses Tailwind utility classes (`src/index.css` imports Tailwind).

### Backend (`apps/api`)

- The whole API is in `src/server.ts`. It exports the Express `app` (so tests can use it without a network port) and starts listening only when the file is run directly.
- The starter has one endpoint: `GET /api/version` → `{ "name": "api", "version": "0.1.0" }`. The MVP adds the read-only `GET /api/demo/entries` described below and in `docs/requirements.md`.
- In development Node.js runs the TypeScript file directly (`node --watch`); `npm run build` compiles it to `dist/`.
- In production (`npm start` runs `dist/server.js`) Express also serves the built frontend from `apps/web/dist` (`serveFrontend`). Every non-`/api` GET request gets `index.html`. The port comes from `PORT`, then `API_PORT`, then 3001.

## How the applications communicate

```
Browser ──► Vite dev server (localhost:5173) ──/api/*──► Express (localhost:3001)
```

- The frontend uses relative URLs such as `fetch('/api/version')` and `fetch('/api/demo/entries')`.
- In development, the Vite proxy (`apps/web/vite.config.ts`) forwards every `/api/*` request to Express. The browser sees a single origin, so no CORS setup is needed.
- Ports can be changed with the `API_PORT` and `WEB_PORT` environment variables.
- In production there is no Vite: Express serves the frontend and the API from the same origin, so relative `/api` URLs keep working and still no CORS is needed. The whole application deploys as one Node.js web service (build: `npm run build`, start: `npm start`).

## Decisions intentionally left open

These are yours to decide based on your requirements. Choose the simplest option that works, and record the choice below.

- **Backend structure** — when and how to split `server.ts` (routers, modules, validation).
- **Frontend structure** — component folders and layout inside the single page (no router is needed for the MVP).
- **State management** — React state and props first; add more only if needed.
- **Validation** — hand-written checks or a validation library; the rules and messages are fixed in `docs/requirements.md`.
- **Chart rendering** — the library and markup used for the Reason Frequency chart.

## How to document a decision

Add a short entry for every significant decision: new endpoints, data model, storage, dependencies, folder structure. Newest first.

```markdown
### YYYY-MM-DD — Short title

- **Context:** What problem or requirement led to this decision?
- **Decision:** What did you choose?
- **Alternatives:** What else did you consider, and why not?
- **Consequences:** What becomes easier or harder?
```

Also keep the "API endpoints" section up to date.

## API endpoints

| Method | Path | Description | Response |
| --- | --- | --- | --- |
| GET | `/api/version` | API name and version | `200 { name, version }` |
| GET | `/api/demo/entries` | Fictional Demo Data from Neon, seeded from `data/seed.json` | `200 { entries }`; `500 { error: { code: "DEMO_DATA_UNAVAILABLE", message: "Could not load Demo Data." } }` |

## Decision log

### 2026-11-25 — Personal data stays in the browser; Neon holds Demo Data only

- **Context:** QA found that the requirements mixed browser-stored personal records with a public Neon database. The public demo has no authentication, and Task Description text can be personal. The MVP also needs to stay small and testable.
- **Decision:** Store personal `ProcrastinationEntry` and `StrategyAttempt` records only in the user's browser, where they survive a page refresh. Use Neon PostgreSQL only for fictional Demo Data seeded from `data/seed.json`, exposed read-only through `GET /api/demo/entries`. Never send personal entries, attempts or feedback to the API. If browser storage is unavailable or full, keep records in memory for the session, show the warning defined in `docs/requirements.md`, and do not call the API.
- **Alternatives:** Storing personal records in Neon (exposes personal text in a public demo and needs authentication first); combining personal and Demo Data in one list or chart (contradicts the separation requirement); adding a clear/reset UI (extra MVP scope).
- **Consequences:** Privacy risk is reduced and the backend has no write endpoints. Personal data is tied to one browser and cannot support Your Growth later; that remains a future decision requiring User Accounts. Tests can use the fixed seed fixture for Demo Data and browser-storage fakes for personal data.

### 2026-11-25 — Neon PostgreSQL for fictional Demo Data

- **Context:** The public demo needs repeatable fictional sample data that is safe to show to anyone. Manual, idempotent seeding from `data/seed.json` is already decided.
- **Decision:** Use Neon PostgreSQL for Demo Data only. Connection settings come from environment variables (such as `DATABASE_URL`), never from committed files. No write endpoint exists in the MVP.
- **Alternatives:** Serving `data/seed.json` as a static asset (no Neon source and no realistic fetch failure); in-memory arrays (lost on restart); a JSON file on the host (ephemeral on Render's free tier).
- **Consequences:** Demo Data survives restarts and deployments and can be tested with the fixed seed fixture. The cost is a database dependency and schema-management work; tests must use a repository fake instead of the real database.

### 2026-11-25 — Render Free Web Service for hosting

- **Context:** The starter already deploys as one Node.js web service that serves the built frontend and the `/api` routes on a single port. A future public demo needs a host that supports this shape.
- **Decision:** Host UNSTUCK on a Render Free Web Service. Use `npm run build` to build and `npm start` to run, and honor the `PORT` environment variable provided by the host. Deployment is future work, not part of the MVP implementation.
- **Alternatives:** Local-only development (no public demo); static hosting (cannot serve the API); another platform (more setup without an advantage for this single-service app).
- **Consequences:** Relative `/api` URLs keep working from one origin, so no CORS is needed. The free tier sleeps and has an ephemeral filesystem, which rules out file-based storage and makes database storage necessary; first requests after sleep may be slow. Secrets such as the database URL are configured in the Render dashboard, not in the repository.

### 2026-11-25 — Manual, idempotent `npm run db:seed`

- **Context:** Development and the public demo need repeatable fictional sample data. Running setup twice must not create duplicate Procrastination Entries or Strategy Attempts.
- **Decision:** Provide a manual `npm run db:seed` command that imports `data/seed.json` and is safe to run repeatedly without creating duplicates. Seeding is an explicit command, not an application-startup step.
- **Alternatives:** Automatic seeding or migrations on startup (unpredictable, duplicate risk, and races across instances); hand-written one-off SQL (not repeatable); a non-idempotent script (duplicates on every rerun).
- **Consequences:** Setup is explicit and reviewable, and reruns are harmless. `data/seed.json` must stay consistent with the stored model: it currently uses display labels and date-only values, so the command must map them to stable codes and UTC timestamps. Developers must remember to run the command against a fresh database.

### 2026-11-25 — Two stored entities, one Strategy Attempt per Entry

- **Context:** The glossary defines exactly two stored entities, `ProcrastinationEntry` and `StrategyAttempt`, and Your Growth reuses them without adding new main entities.
- **Decision:** Store only `ProcrastinationEntry` and `StrategyAttempt`; an entry has at most one attempt in the MVP. Personal copies live in browser storage and Demo Data copies live in Neon, but the shape is the same. Procrastination Reasons and Strategies remain predefined values rather than stored entities.
- **Alternatives:** Tables for reasons and strategies (contradicts the glossary); multiple attempts per entry (outside MVP scope).
- **Consequences:** The schema stays small and API logic stays straightforward. Enforcing the one-attempt rule in both the browser store and the Demo Data schema means supporting retries later would need changes in both places, not a rewrite of the entities.

### 2026-11-25 — Stable codes, UTC timestamps, feedback values and feedback dates

- **Context:** Basic Insights counts Reason Frequency from browser-stored personal entries. Your Growth may later group evaluated Strategy Attempts by the date their feedback was recorded and calculate Start Rate from the feedback values.
- **Decision:** Store stable codes for the selected Procrastination Reason and the suggested Strategy, UTC timestamps, Strategy Feedback values (`helped`, `partially_helped`, `not_helped`, or `null`), and the date on which feedback was recorded.
- **Alternatives:** Storing display labels or free text (renaming a label would break history); local-time timestamps (ambiguous week boundaries); reconstructing feedback dates later (not reliably possible).
- **Consequences:** The stored shape is ready for future analytics without a rewrite, but the MVP keeps analytics to personal-only Reason Frequency. Codes are immutable identifiers; label text may change only in the UI. Seed data must be translated from its current labels to these codes and its date-only values to `00:00:00.000Z`.

### 2026-11-25 — Repository layer between storage and API logic

- **Context:** Route handlers should not contain Demo Data queries, and unit tests must run without a live Neon database.
- **Decision:** Put Demo Data database access behind a repository layer that separates it from API logic.
- **Alternatives:** Queries directly in routes (couples the API to SQL and complicates testing); a full ORM (extra dependency and concepts not justified by two entities).
- **Consequences:** Tests can substitute an in-memory fake and the SQL implementation stays localized. The trade-off is one additional module boundary and explicit types at that boundary.

### 2026-11-25 — Data model ready for future User Accounts, no authentication in the MVP

- **Context:** User Accounts are out of scope now but planned for a future version. Your Growth continues to serve a single User without authentication.
- **Decision:** Prepare the data model for future User Accounts, including an `ownerId` field, but do not implement registration, login or authentication in the MVP.
- **Alternatives:** Omitting any owner field (a future account system would have to backfill every row); implementing accounts now (contradicts the MVP scope).
- **Consequences:** Future multi-user work starts from existing records rather than a data backfill. The `ownerId` field is not a security boundary: on its own it provides no user isolation.

### 2026-11-25 — Public demo uses fictional data only

- **Context:** The app will be publicly reachable on Render without authentication, and Task Description text can be personal.
- **Decision:** The public demo must use fictional data only. An `ownerId` field alone does not provide user isolation or security. Personal entries stay in the browser and are never written to the API or Neon; `ownerId` is always `public_demo`.
- **Alternatives:** Allowing real personal records in the demo guarded only by `ownerId` (unsafe); adding authentication now (out of scope for the MVP).
- **Consequences:** Seeded and demo records are safe to show to anyone, no real procrastination records are exposed, and the API has no personal-write path. Handling real user data later requires User Accounts and authentication first.

### 2026-11-25 — Your Growth stays out of the MVP

- **Context:** Your Growth is a planned dashboard described in `docs/features/your-growth.md` and depends on historical feedback data.
- **Decision:** Do not implement Your Growth in the MVP; include only the data prerequisites it needs, such as stable codes and feedback recording dates.
- **Alternatives:** Building the dashboard in the MVP (expands scope and contradicts `docs/requirements.md`).
- **Consequences:** The MVP stays small and testable. Weekly Start Rate, strategy effectiveness and progress comparisons are added later through new endpoints and UI using the existing entities and stored feedback dates.
