# Architecture

This folder records **how** the application is built and **why**. Keep it short and up to date: AI agents read it before every task (`docs/architecture/README.md` and `docs/architecture/decisions.md` are loaded automatically through `opencode.jsonc`), and so will anyone who reviews your project.

| File | Purpose |
| --- | --- |
| `README.md` | Overview of the system parts, communication and cross-cutting rules. |
| `data-model.md` | Stored entities, relations, keys and constraints. |
| `backend.md` | Backend structure, API conventions and rules. |
| `frontend.md` | Frontend structure, components, state management and rules. |
| `decisions.md` | Decision log, newest first. |

## Provided infrastructure

The starter contains infrastructure only, with no product features.

| Part | Location | Technology |
| --- | --- | --- |
| Frontend | `apps/web` | React 19, TypeScript, Vite, Tailwind CSS 4 |
| Backend | `apps/api` | Node.js, Express 5, TypeScript |
| Unit tests | `apps/*/src/**/*.test.ts(x)` | Vitest, React Testing Library, Supertest |
| End-to-end tests | `e2e/` | Playwright |
| Workspace | root `package.json` | npm workspaces |

## How the applications communicate

```
Browser ──► Vite dev server (localhost:5173) ──/api/*──► Express (localhost:3001)
```

- The frontend uses relative URLs such as `fetch('/api/version')` and `fetch('/api/demo/entries')`.
- In development, the Vite proxy (`apps/web/vite.config.ts`) forwards every `/api/*` request to Express. The browser sees a single origin, so no CORS setup is needed.
- Ports can be changed with the `API_PORT` and `WEB_PORT` environment variables.
- In production there is no Vite: Express serves the frontend and the API from the same origin, so relative `/api` URLs keep working and still no CORS is needed. The whole application deploys as one Node.js web service (build: `npm run build`, start: `npm start`).

## Cross-cutting rules

- **Personal data boundary:** Personal `ProcrastinationEntry` and `StrategyAttempt` records stay in browser storage and are never sent to an API or Neon. Neon holds fictional Demo Data only; the API is read-only.
- **Errors:** API endpoints return JSON errors with correct status codes. The Demo Data endpoint's failure response is `500 { error: { code: "DEMO_DATA_UNAVAILABLE", message: "Could not load Demo Data." } }`; see `backend.md` for endpoint conventions.
- **Validation:** Validate input before using it. The fixed validation rules and exact UI messages are in `docs/requirements.md`. Procrastination Reasons and Strategies come only from the predefined catalog.
- **Configuration:** The API port comes from `PORT`, then `API_PORT`, then 3001. Development ports can be changed with `API_PORT` and `WEB_PORT`. Use `npm run build` to build and `npm start` to run the production service.
- **Environment:** Secrets such as the database URL (`DATABASE_URL`) come from environment variables, never from committed files. On Render they are configured in the dashboard, not in the repository.
- **Code reuse and consistency:** Use the terms in `docs/glossary.md` exactly in code, UI and tests. Reuse the shared catalog, validation helpers, storage helper, API functions and Demo Data repository instead of duplicating constants, messages or I/O code. Define explicit TypeScript types in each app, keep their field names and codes aligned with `docs/requirements.md`, and avoid `any`. Do not add a dependency without an explicit recorded decision.

## Libraries

| Area | Choice |
| --- | --- |
| State management | Custom React hooks; no Context, reducer or external state library. |
| Validation | Hand-written helpers in `src/validation.ts`; no validation library. |
| Forms | Native controlled React forms; no form library. |
| UI | Semantic HTML and Tailwind utility classes only; no component library. |
| Reason Frequency chart | Semantic ordered list with Tailwind bars; no chart library or SVG. |
| Database access | `pg` with raw parameterized SQL; no ORM or query builder. |

Each choice is recorded in `decisions.md`.

## Decisions intentionally left open

All architecture choices previously listed here are now recorded in `decisions.md`. Questions explicitly deferred in `docs/requirements.md` remain deferred and are not designed in the MVP, including Your Growth dashboards.
