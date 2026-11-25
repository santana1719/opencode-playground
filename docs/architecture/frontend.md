# Frontend

### Frontend (`apps/web`)

- `src/main.tsx` renders `src/App.tsx`, the starter page.
- `src/api.ts` contains the functions that call the backend.
- Styling uses Tailwind utility classes (`src/index.css` imports Tailwind).

## Frontend rules

- The frontend uses relative URLs such as `fetch('/api/version')` and `fetch('/api/demo/entries')`; all backend calls go through `/api/...`.
- Personal `ProcrastinationEntry` and `StrategyAttempt` records live only in browser storage. If storage is unavailable or full, keep them in memory for the current session and show the warning defined in `docs/requirements.md`; send nothing to the API.
- `Demo Data` is fictional, read-only and shown in a separate section from `My entries`. Only `My entries` is interactive.
- `Basic Insights` shows Reason Frequency from personal entries only; Demo Data is never counted.
- Handle loading, error and empty states for Demo Data without making personal entries, the form or `Basic Insights` depend on that request.
- Use function components and hooks, Tailwind utility classes, semantic HTML, labels for form fields and keyboard-usable controls. Design for small screens first and add `sm:`/`md:` breakpoints.
- Keep fetch functions in `src/api.ts` (or a small module next to it) and give responses explicit TypeScript types.
- Create new files in `apps/web/src` only when a component or helper is reused or the file is getting hard to read. No router is needed for the MVP.
- Chart rendering remains an open decision; do not choose a library or markup silently.
