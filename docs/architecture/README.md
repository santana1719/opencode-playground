# Architecture

This folder records **how** the application is built and **why**. Keep it short and up to date: AI agents read it before every task (`docs/architecture/README.md` and `docs/architecture/decisions.md` are loaded automatically through `opencode.jsonc`), and so will anyone who reviews your project.

| File | Purpose |
| --- | --- |
| `README.md` | Overview of the system parts, communication and cross-cutting rules. |
| `data-model.md` | Stored entities, relations, keys and field rules. |
| `backend.md` | Backend structure and rules. |
| `frontend.md` | Frontend structure and rules. |
| `decisions.md` | Decision log, including decisions intentionally left open. |

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

- **Errors:** API endpoints return JSON errors with correct status codes. The Demo Data endpoint's failure response is `500 { error: { code: "DEMO_DATA_UNAVAILABLE", message: "Could not load Demo Data." } }`; see `backend.md` for the endpoint contract.
- **Validation:** Validate input before using it. The fixed validation rules and exact UI messages are in `docs/requirements.md`. Procrastination Reasons and Strategies come only from the predefined catalog.
- **Configuration:** The API port comes from `PORT`, then `API_PORT`, then 3001. Development ports can be changed with `API_PORT` and `WEB_PORT`. Use `npm run build` to build and `npm start` to run the production service.
- **Environment:** Secrets such as the database URL (`DATABASE_URL`) come from environment variables, never from committed files. On Render they are configured in the dashboard, not in the repository.

## Decisions intentionally left open

These are yours to decide based on your requirements. Choose the simplest option that works, and record the choice below. Accepted choices belong in `docs/architecture/decisions.md`.

- **Backend structure** — when and how to split `server.ts` (routers, modules, validation).
- **Frontend structure** — component folders and layout inside the single page (no router is needed for the MVP).
- **State management** — React state and props first; add more only if needed.
- **Validation** — hand-written checks or a validation library; the rules and messages are fixed in `docs/requirements.md`.
- **Chart rendering** — the library and markup used for the Reason Frequency chart.
