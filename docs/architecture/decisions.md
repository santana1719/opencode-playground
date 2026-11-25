# Decision log

## How to document a decision

Add a short entry for every significant decision: new endpoints, data model, storage, dependencies, folder structure. Newest first.

```markdown
### YYYY-MM-DD — Short title

- **Context:** What problem or requirement led to this decision?
- **Decision:** What did you choose?
- **Alternatives:** What else did you consider, and why not?
- **Consequences:** What becomes easier or harder?
```

Also keep the "API endpoints" section in `docs/architecture/backend.md` up to date.

### 2026-11-25 — Semantic HTML and Tailwind bars for Reason Frequency

- **Context:** `Basic Insights` must show Reason Frequency from personal entries only, ordered by count descending and then catalog order. `docs/requirements.md` fixes counts and labels but leaves chart rendering and styling open.
- **Decision:** Render Reason Frequency in `BasicInsights.tsx` as a semantic ordered list of reasons. Each row shows the Procrastination Reason display label, its count and a simple bar styled with Tailwind utility classes whose width is proportional to the count. Do not add a chart library or hand-built SVG.
- **Options considered:** (1) Semantic HTML with Tailwind bars; no dependency and exact labels/counts are easy to test, but no axes, tooltips or animations. (2) Hand-built SVG; more visual control, but more code and accessibility work. (3) Chart library such as Recharts; polished features, but a dependency and configuration are needed to preserve exact labels and ordering.
- **Rationale:** The user explicitly selected semantic HTML with Tailwind bars.
- **Alternatives:** SVG was not chosen because it adds complexity without meeting extra requirements; a chart library was not chosen because the MVP needs counts and ordering, not chart features.
- **Consequences:** The chart stays accessible, responsive and easy to test from rendered text. Richer visual features would require a later decision; Demo Data is still excluded from all counts.

### 2026-11-25 — `pg` driver with raw parameterized SQL for Demo Data

- **Context:** The repository layer is chosen, and the decision log already rejected a full ORM for two stored entities. The README lists SQL access behind the repository as an open choice. Demo Data is read-only from Neon through `GET /api/demo/entries`.
- **Decision:** Use the `pg` (node-postgres) driver inside `repositories/demo-entries.ts`. Read the connection string from `DATABASE_URL`, write explicit parameterized `SELECT` statements, and map rows to the typed `ProcrastinationEntry` and `StrategyAttempt` shapes. Keep the in-memory repository fake for tests. Do not add an ORM or query builder for the MVP.
- **Options considered:** (1) `pg` with raw parameterized SQL; minimal dependency and visible SQL, but manual row mapping and type upkeep. (2) Kysely query builder; typed schema and queries, but extra dependency and concepts for two reads. (3) Neon serverless driver with raw SQL; provider-specific connection handling, but tighter coupling with little MVP benefit.
- **Rationale:** The user explicitly selected `pg` with raw SQL.
- **Alternatives:** Kysely was not chosen because its schema layer is unnecessary for two read queries; the Neon serverless driver was not chosen because it couples the repository to one provider without meaningful benefit here. A full ORM was already rejected in an earlier decision.
- **Consequences:** The SQL implementation stays small and localized behind the repository, and tests still use the fake without a live database. Developers must keep SQL parameters safe and row mapping aligned with `types/demo-entries.ts`; personal data is still never sent to Neon.

### 2026-11-25 — Tailwind utility classes only

- **Context:** Tailwind CSS 4 is already provided in `apps/web`, and the MVP must render correctly at 360 × 640 and 1280 × 800 without horizontal scrolling. The README lists UI library as an open choice.
- **Decision:** Build the UI with semantic HTML elements and Tailwind utility classes only. Do not add a headless primitive library or a full component library. Implement buttons, labels, selects, feedback choices and cards with native elements; handle keyboard use and accessible labels explicitly.
- **Options considered:** (1) Tailwind utilities only; no dependency and full control over exact text and layout, but repeated class patterns and manual accessibility. (2) Headless primitives plus Tailwind; better built-in interaction behaviour, but a new dependency for mostly native controls. (3) Full component library; ready-made components, but heavier, harder to combine with Tailwind and unnecessary for the MVP.
- **Rationale:** The user explicitly selected Tailwind only.
- **Alternatives:** Headless primitives and full component libraries were not chosen because the MVP uses ordinary semantic controls and no dependency is justified.
- **Consequences:** The bundle stays small and exact UI strings remain under direct control. Developers must keep styling consistent and verify accessibility by hand; this decision does not choose the Reason Frequency chart rendering.

### 2026-11-25 — Native controlled React forms

- **Context:** The MVP needs a Task Description form, a Procrastination Reason selector and Strategy Feedback choices with exact validation messages. `validation.ts` is already chosen; the README lists form handling as an open choice.
- **Decision:** Use native controlled React forms. `EntryForm.tsx` holds Task Description, Reason Code and error messages in `useState`, calls `validation.ts` on submit, preserves values after an error and passes the trimmed Task Description to the personal-data hook. `EntryCard.tsx` uses the same pattern for the selected Strategy Feedback and its error, and disables the controls once feedback is recorded. Do not add a form library or `FormData`-based handling in the MVP.
- **Options considered:** (1) Native controlled React form; no dependency and direct integration with `validation.ts`, but field state and error clearing are manual. (2) Uncontrolled `FormData`; fewer renders, but harder to preserve values and clear specific errors. (3) React Hook Form; mature state handling, but adds a dependency and resolver code for exact messages.
- **Rationale:** The user explicitly selected native controlled React forms.
- **Alternatives:** Uncontrolled forms were not chosen because error and value state would be harder to manage; React Hook Form was not chosen because it is unnecessary for one small form and one feedback control.
- **Consequences:** Form behaviour stays dependency-free and testable with React Testing Library. Developers must handle field reset and error clearing explicitly; this decision does not choose the UI library or chart rendering.

### 2026-11-25 — Hand-written validation functions

- **Context:** The MVP validates a Task Description (required, trimmed, 1–500 characters), a selected Procrastination Reason from a fixed catalog, and Strategy Feedback before recording it. `docs/requirements.md` fixes the exact error messages, and the README lists validation as an open choice.
- **Decision:** Put pure validation helpers in `apps/web/src/validation.ts`. They accept form values, trim the Task Description before checking its length, and return the exact message from `docs/requirements.md` or no error. Use `src/catalog.ts` for the allowed Reason Codes and Strategy Codes. Components call the helpers on submit and keep the form usable after an error. Do not add a validation library.
- **Options considered:** (1) Hand-written functions; exact strings are explicit, no dependency, easy to unit test, but rules must be updated manually. (2) Zod schemas; declarative and typed, but adds a dependency and message-mapping code for three simple rules. (3) Native HTML constraints only; insufficient because browser messages are not the required strings and `maxLength` does not count trimmed length.
- **Rationale:** The user explicitly selected hand-written functions.
- **Alternatives:** Zod was not chosen because the rule set is small and the dependency is unnecessary; native constraints were not chosen because they cannot satisfy the exact messages or trimmed-length rule.
- **Consequences:** Validation behaviour is easy to test directly and errors are guaranteed to use the required strings. Future rule changes must be made in `validation.ts`; this decision does not choose form handling, UI library or chart rendering.

### 2026-11-25 — Custom hooks own frontend state

- **Context:** The components-by-concern decision leaves `App.tsx` to compose sections, but the MVP needs state for personal Procrastination Entries, Strategy Attempts, storage warnings and Demo Data loading/error/empty states. The README lists state management as an open choice and the repo guidance prefers React state and props first.
- **Decision:** Use two custom hooks at `apps/web/src/`: `use-personal-data.ts` owns personal entries, Strategy Attempts, storage warnings and the handlers that create an entry, add a Strategy Attempt and record Strategy Feedback, using `storage.ts` for browser storage and the in-memory fallback; `use-demo-data.ts` owns Demo Data entries and loading/error state and calls the fetch functions in `api.ts`. `App.tsx` calls both hooks and passes values and handlers to components as props. Do not add Context, a reducer or an external state-management library.
- **Options considered:** (1) React state in `App.tsx` with props; simplest and matches the repo guidance, but leaves many state variables and handlers in one component. (2) Custom hooks; separate state ownership from rendering and make hooks directly testable, with more files and hook APIs to design. (3) Context and reducer; avoids prop passing, but adds boilerplate and obscures one-page data flow.
- **Rationale:** The user explicitly selected custom hooks.
- **Alternatives:** Plain state in `App.tsx` was not chosen because hook boundaries keep storage and Demo Data state separate; Context and a reducer were not chosen because they add unnecessary structure for the MVP.
- **Consequences:** `App.tsx` stays focused on composition; hook behaviour can be tested independently. Props still pass from `App.tsx` to section components. Fetch function location remains `api.ts`, and this decision does not choose validation, form handling, UI library or chart rendering.

### 2026-11-25 — Components organized by UI concern

- **Context:** `apps/web/src/App.tsx` is the starter page, but the MVP needs a Task Description form, `My entries`, `Demo Data` and `Basic Insights`. The README lists frontend structure as an open choice; no router is needed for the MVP, and state management, data fetching, validation, forms, UI library and chart rendering remain separate open choices.
- **Decision:** Use `apps/web/src/components/` for UI components by concern: `EntryForm.tsx`, `EntryCard.tsx`, `MyEntriesSection.tsx`, `DemoDataSection.tsx` and `BasicInsights.tsx`. Keep shared code at the `src/` root in `catalog.ts`, `types.ts`, `api.ts` and `storage.ts`. `App.tsx` composes the sections and does not implement storage. Do not add `features/` folders or a router in the MVP.
- **Options considered:** (1) Keep most UI in `App.tsx` and extract only when reused or hard to read; fewest files, but a large page and less obvious section boundaries. (2) Components folder by UI concern; clear homes for each required area and isolated tests, with more files than option 1. (3) Feature folders with `shared/`; scales for Your Growth, but adds structure and cross-cutting imports before the MVP needs them.
- **Rationale:** The user explicitly selected components organized by UI concern.
- **Alternatives:** Keeping most UI in `App.tsx` was not chosen because section boundaries would be less clear; feature folders were not chosen because they add structure for a future feature.
- **Consequences:** Each required UI area has an obvious file and can be tested in isolation. `App.tsx` stays focused on composition, but props may pass through it depending on the later state-management choice. Storage implementation, fetch behaviour and the chart library are not decided here.

### 2026-11-25 — Router and repository folders for the backend

- **Context:** `apps/api/src/server.ts` started as a single file, but the MVP must expose `GET /api/demo/entries` through a repository layer whose implementation can be faked in tests. The decision log already separates read-only Demo Data from browser-stored personal data. The README lists backend structure as an open choice, and Your Growth may add statistics endpoints later, but is out of scope now.
- **Decision:** Use a router-and-repository structure in `apps/api/src`: `server.ts` creates and exports `app`, installs `express.json()`, keeps `GET /api/version`, mounts the Demo Data router, installs error middleware, defines `serveFrontend` and starts listening only in the `import.meta.main` block; `routes/demo-entries.ts` defines the read-only `GET /api/demo/entries` route; `repositories/demo-entries.ts` holds the repository interface, its Neon/SQL implementation and an in-memory fake for tests; `types/demo-entries.ts` holds explicit types shared by the route and repository. Do not add a service layer or write routes in the MVP.
- **Options considered:** (1) Minimal two-module split — `server.ts` plus one repository module; simplest and satisfies the repository boundary, but leaves all routes in `server.ts`. (2) Router plus repository folders — explicit HTTP layer and straightforward future routes, but more files for two GET endpoints. (3) Feature slices with a service layer — scales best for Your Growth, but adds layers before the MVP needs them and conflicts with the preference for simple, readable code.
- **Rationale:** The user explicitly selected the router-and-repository structure.
- **Alternatives:** The minimal two-module split was not chosen because it keeps routes in `server.ts`; feature slices with services were not chosen because they add structure for a future feature.
- **Consequences:** Route and storage concerns are visibly separate, and a future Your Growth route can be added beside the Demo Data router without rewriting the repository. The cost is four source files for a small API; developers must avoid adding folders or layers before a requirement needs them. Tests can substitute the in-memory repository fake, and no personal data path is introduced.

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
