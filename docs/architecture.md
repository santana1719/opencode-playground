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
- One endpoint exists: `GET /api/version` → `{ "name": "api", "version": "0.1.0" }`.
- In development Node.js runs the TypeScript file directly (`node --watch`); `npm run build` compiles it to `dist/`.
- In production (`npm start` runs `dist/server.js`) Express also serves the built frontend from `apps/web/dist` (`serveFrontend`). Every non-`/api` GET request gets `index.html`. The port comes from `PORT`, then `API_PORT`, then 3001.

## How the applications communicate

```
Browser ──► Vite dev server (localhost:5173) ──/api/*──► Express (localhost:3001)
```

- The frontend uses relative URLs such as `fetch('/api/version')`.
- In development, the Vite proxy (`apps/web/vite.config.ts`) forwards every `/api/*` request to Express. The browser sees a single origin, so no CORS setup is needed.
- Ports can be changed with the `API_PORT` and `WEB_PORT` environment variables.
- In production there is no Vite: Express serves the frontend and the API from the same origin, so relative `/api` URLs keep working and still no CORS is needed. The whole application deploys as one Node.js web service (build: `npm run build`, start: `npm start`).

## Decisions intentionally left open

These are yours to decide based on your requirements. Choose the simplest option that works, and record the choice below.

- **Data storage** — in memory, a JSON file, browser `localStorage`, or a database.
- **API design** — resources, URLs, request/response shapes, error format.
- **Backend structure** — when and how to split `server.ts` (routers, modules, validation).
- **Frontend structure** — pages, component folders, routing (whether you need a router at all).
- **State management** — React state and props first; add more only if needed.
- **Validation** — hand-written checks or a validation library.
- **Authentication** — only if your requirements really need it.
- **Deployment** — whether and where the application runs outside your computer (one Node.js web service is enough).

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

## Decision log

### 2026-11-25 — Neon PostgreSQL for persistent storage

- **Context:** The MVP requires Procrastination Entries and Strategy Feedback to remain available after a page refresh. Your Growth will need historical records that can be grouped over time. In-memory state disappears on restart or redeploy.
- **Decision:** Use Neon PostgreSQL for persistent data. Connection settings come from environment variables (such as `DATABASE_URL`), never from committed files.
- **Alternatives:** In-memory arrays (quick to prototype, but data is lost on restart and weekly comparisons become impossible); browser `localStorage` (client-only, no shared server-side queries); a JSON file (ephemeral on hosting platforms).
- **Consequences:** Data survives refreshes, restarts and deployments, and SQL aggregation supports Reason Frequency now and Start Rate later. The cost is a database dependency and schema-management work; tests must use a separate database or a repository fake instead of the real database.

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
- **Decision:** Store only `ProcrastinationEntry` and `StrategyAttempt`; an entry has at most one attempt in the MVP. Procrastination Reasons and Strategies remain predefined values rather than stored entities.
- **Alternatives:** Tables for reasons and strategies (contradicts the glossary); multiple attempts per entry (outside MVP scope).
- **Consequences:** The schema stays small and API logic stays straightforward. Enforcing the one-attempt rule at the database level means supporting retries later would need a migration, not a rewrite of the entities.

### 2026-11-25 — Stable codes, UTC timestamps, feedback values and feedback dates

- **Context:** Basic Insights counts Reason Frequency. Your Growth groups evaluated Strategy Attempts by the date their feedback was recorded and calculates Start Rate from the feedback values.
- **Decision:** Store stable codes for the selected Procrastination Reason and the suggested Strategy, UTC timestamps, Strategy Feedback values (`helped`, `partially_helped`, `not_helped`, or `null`), and the date on which feedback was recorded.
- **Alternatives:** Storing display labels or free text (renaming a label would break history); local-time timestamps (ambiguous week boundaries); reconstructing feedback dates later (not reliably possible).
- **Consequences:** Historical data supports weekly comparisons and "most helpful strategies" without a storage rewrite. Codes are immutable identifiers; label text may change only in the UI. Seed data must be translated from its current labels to these codes.

### 2026-11-25 — Repository layer between storage and API logic

- **Context:** Route handlers should not contain database queries, and unit tests must run without a live Neon database.
- **Decision:** Put database access behind a repository layer that separates it from API logic.
- **Alternatives:** Queries directly in routes (couples the API to SQL and complicates testing); a full ORM (extra dependency and concepts not justified by two entities).
- **Consequences:** Tests can substitute an in-memory fake and the SQL implementation stays localized. The trade-off is one additional module boundary and explicit types at that boundary.

### 2026-11-25 — Data model ready for future User Accounts, no authentication in the MVP

- **Context:** User Accounts are out of scope now but planned for a future version. Your Growth continues to serve a single User without authentication.
- **Decision:** Prepare the data model for future User Accounts, including an `ownerId` field, but do not implement registration, login or authentication in the MVP.
- **Alternatives:** Omitting any owner field (a future account system would have to backfill every row); implementing accounts now (contradicts the MVP scope).
- **Consequences:** Future multi-user work starts from existing records rather than a data backfill. The `ownerId` field is not a security boundary: on its own it provides no user isolation.

### 2026-11-25 — Public demo uses fictional data only

- **Context:** The app will be publicly reachable on Render without authentication, and Task Description text can be personal.
- **Decision:** The public demo must use fictional data only. An `ownerId` field alone does not provide user isolation or security.
- **Alternatives:** Allowing real personal records in the demo guarded only by `ownerId` (unsafe); adding authentication now (out of scope for the MVP).
- **Consequences:** Seeded and demo records are safe to show to anyone, and no real procrastination records are exposed. Handling real user data later requires User Accounts and authentication first.

### 2026-11-25 — Your Growth stays out of the MVP

- **Context:** Your Growth is a planned dashboard described in `docs/features/your-growth.md` and depends on historical feedback data.
- **Decision:** Do not implement Your Growth in the MVP; include only the data prerequisites it needs, such as stable codes and feedback recording dates.
- **Alternatives:** Building the dashboard in the MVP (expands scope and contradicts `docs/requirements.md`).
- **Consequences:** The MVP stays small and testable. Weekly Start Rate, strategy effectiveness and progress comparisons are added later through new endpoints and UI using the existing entities and stored feedback dates.
