# Frontend

## Frontend (`apps/web`)

- `src/main.tsx` renders `src/App.tsx`, the starter page.
- `src/api.ts` contains the functions that call the backend.
- Styling uses Tailwind utility classes (`src/index.css` imports Tailwind).

## Folder structure

| Path | Responsibility |
| --- | --- |
| `src/App.tsx` | Page layout and composition of the three sections. No storage implementation. |
| `src/components/EntryForm.tsx` | Task Description input and Procrastination Reason selection. |
| `src/components/EntryCard.tsx` | One personal Procrastination Entry, its Strategy Suggestion and Strategy Feedback controls. |
| `src/components/MyEntriesSection.tsx` | The interactive `My entries` section and its empty state. |
| `src/components/DemoDataSection.tsx` | The read-only `Demo Data` section, including loading and error states. |
| `src/components/BasicInsights.tsx` | Reason Frequency from personal entries only. |
| `src/catalog.ts` | Predefined Reason Codes, Strategy Codes and display labels from `docs/requirements.md`. |
| `src/validation.ts` | Pure validation helpers returning the exact messages from `docs/requirements.md`. |
| `src/types.ts` | Explicit TypeScript types shared across components and helpers. |
| `src/api.ts` | Fetch functions for backend calls. |
| `src/storage.ts` | Browser storage access and the in-memory fallback. |
| `src/use-personal-data.ts` | Personal entries, Strategy Attempts, storage warnings and the handlers that change them. |
| `src/use-demo-data.ts` | Demo Data entries and loading/error state, using the fetch functions in `api.ts`. |

Do not add `features/` folders, a router, Context, a reducer, an external state-management library or a component library in the MVP. `App.tsx` calls both hooks and passes values and handlers to components as props.

## State and data flow

- `use-personal-data.ts` loads personal `ProcrastinationEntry` and `StrategyAttempt` records from `storage.ts` and exposes handlers that create an entry, add a Strategy Attempt and record Strategy Feedback.
- `use-demo-data.ts` calls the fetch functions in `api.ts` and exposes Demo Data entries plus loading and error state. No retry action is added in the MVP.
- `App.tsx` does not implement storage or fetching; it passes hook results to section components. Personal entries, the form and `Basic Insights` do not depend on the Demo Data request.

## Frontend rules

- The frontend uses relative URLs such as `fetch('/api/version')` and `fetch('/api/demo/entries')`; all backend calls go through `/api/...`.
- Validate on submit with `validation.ts`: trim the Task Description, check the 1–500 limit, require a catalog Reason Code and require a Strategy Feedback value before saving it. Show the exact messages from `docs/requirements.md` and keep the form usable after an error.
- Use native controlled forms: `EntryForm.tsx` keeps Task Description, Reason Code and error messages in `useState`; `EntryCard.tsx` keeps the selected Strategy Feedback in `useState` and disables the controls once feedback is recorded. Do not add a form library.
- Personal `ProcrastinationEntry` and `StrategyAttempt` records live only in browser storage. If storage is unavailable or full, keep them in memory for the current session and show the warning defined in `docs/requirements.md`; send nothing to the API.
- If stored data is malformed or has missing required fields, skip invalid records, show the corrupted-data warning and continue with valid records.
- `Demo Data` is fictional, read-only and shown in a separate section from `My entries`. Only `My entries` is interactive.
- `Basic Insights` shows Reason Frequency from personal entries only; Demo Data is never counted. Render it as a semantic ordered list with the Procrastination Reason display label, the count and a proportional Tailwind bar, ordered by count descending then catalog order. Show the exact empty message from `docs/requirements.md` when there are no personal entries.
- Handle loading, error and empty states for Demo Data without making personal entries, the form or `Basic Insights` depend on that request; use the exact strings in `docs/requirements.md`.
- Use function components and hooks, semantic HTML and Tailwind utility classes only; do not add a headless or full component library. Label form fields, keep controls keyboard-usable, and design for small screens first with `sm:`/`md:` breakpoints.
- Keep fetch functions in `src/api.ts` and give responses explicit TypeScript types.
- Create new files under `src/components/` only for the UI concerns above or when a component is reused or a file becomes hard to read.
