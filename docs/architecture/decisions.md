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

### 2026-11-25 — `TIMESTAMPTZ(3)` columns mapped to ISO strings

- **Context:** Review of the earlier SQL schema decision: `recorded_at` and `feedback_recorded_at` were `TEXT` ISO strings to avoid conversion code. `docs/requirements.md` fixes the API/browser representation as UTC ISO 8601 strings with milliseconds, not the physical SQL type, and future date grouping needs real instants.
- **Decision:** Use `TIMESTAMPTZ(3)` for `recorded_at` in `demo_procrastination_entries` and `feedback_recorded_at` in `demo_strategy_attempts`. `pg` returns `Date` objects; `repositories/demo-entries.ts` maps them with `toISOString()` so `GET /api/demo/entries` returns the ISO 8601 strings with milliseconds required by `docs/requirements.md`, and the in-memory fake exposes the same string shape. The seed importer converts date-only values to `YYYY-MM-DDT00:00:00.000Z` before insert. The `ProcrastinationEntry` and `StrategyAttempt` shapes are unchanged.
- **Options considered:** (1) `TIMESTAMPTZ(3)` with explicit `Date` → ISO string mapping; native date semantics and millisecond precision, but conversion code in the repository and fake. (2) Keep `TEXT`; exact round-trip with no conversion, but no database date semantics or constraints. (3) `TIMESTAMPTZ` without precision limit; native instants, but microseconds are silently truncated to milliseconds when mapped to `Date` and the API.
- **Rationale:** The user explicitly selected `TIMESTAMPTZ(3)` mapped to ISO strings.
- **Alternatives:** `TEXT` was not chosen because it gives up date semantics; unlimited `TIMESTAMPTZ` was not chosen because its microsecond precision cannot survive the API's millisecond representation.
- **Consequences:** SQL supports date constraints, ordering and future analytics; the repository and test fake must convert on every read, and tests should assert millisecond ISO strings. Personal browser data is unaffected.

### 2026-11-25 — Explicit immutable `seedKey` in Demo Data fixture

- **Context:** Review of the earlier seeding decision: deriving `seed_key` from `taskDescription|recordedOn` makes reseeding create a duplicate if either seed value is edited, silently breaking the 14-entry fixture.
- **Decision:** Add an explicit `seedKey` string to each of the 14 rows in `data/seed.json`. It is immutable, unique and never reused. `apps/api/scripts/seed.ts` reads it into `demo_procrastination_entries.seed_key` and uses that column for `ON CONFLICT`; no derivation from `taskDescription` or `recordedOn`. `seedKey` is import metadata only and is not mapped into the `ProcrastinationEntry` objects returned by the API.
- **Options considered:** (1) Explicit `seedKey` in the fixture; stable against text/date edits and row reordering, but the canonical fixture gains a field implementers must add and preserve. (2) Keep the derived key; no fixture change, but content edits cause duplicates on reseed. (3) Derive from array position; independent of text, but reordering or inserting rows rekeys entries and leaves stale rows.
- **Rationale:** The user explicitly selected the explicit immutable `seedKey`.
- **Alternatives:** Derived keys were not chosen because content edits break idempotency; positional keys were not chosen because row order is not a stable identity.
- **Consequences:** Descriptions and dates in the fixture can be corrected without manual database cleanup; `data/seed.json` must be updated by the implementer, and reviewers must treat every `seedKey` as permanent. The API contract in §6.6 is unchanged.

### 2026-11-25 — Stored-record validation in `validation.ts`

- **Context:** Finding 8 of the architecture review: `docs/requirements.md` requires malformed stored records to be skipped with the corrupted-data warning, but the rules and their location were undefined; `validation.ts` was documented for form values only.
- **Decision:** Add pure guards to `apps/web/src/validation.ts` for stored records and call them from `storage.ts` on load. A stored `ProcrastinationEntry` is valid only if it has a string UUID `id`, a trimmed `taskDescription` of 1–500 characters, a `procrastinationReason` from the catalog, a `recordedAt` ISO 8601 UTC string with milliseconds and `ownerId` equal to `public_demo`. A stored `StrategyAttempt` is valid only if it has a string UUID `id`, an `entryId` referencing a kept entry, the Strategy Code mapped to that entry's Reason Code, `feedback` of `helped`/`partially_helped`/`not_helped`/`null`, and `feedbackRecordedAt` that is `null` exactly when `feedback` is `null`, otherwise an ISO 8601 UTC string with milliseconds. Skip later records with duplicate `id`s, skip attempts whose `entryId` has no kept entry, and keep only the first valid attempt per entry. Unparseable JSON or a non-array root yields no records. Skipped records trigger the exact corrupted-data warning; valid records continue to load. Do not add a validation library or a separate stored-data module.
- **Options considered:** (1) Pure guards in `validation.ts`, called by `storage.ts`; one testable validation home, but the module covers both forms and stored records. (2) Guards inside `storage.ts`; validation stays with I/O but mixes concerns and separates rules from the existing helpers. (3) A separate `stored-data.ts`; clean separation but one more file for a small MVP.
- **Rationale:** The user explicitly selected extending `validation.ts`.
- **Alternatives:** In-storage guards were not chosen because they mix parsing with validation; a separate module was not chosen because the rule set is small enough to keep beside the form helpers.
- **Consequences:** Corrupted-data handling is deterministic and unit-testable from rendered text and pure helpers; form validation and stored-record validation must be kept consistent with the catalog and the data model in one file. The in-memory fallback is created by the app itself and is not revalidated.

### 2026-11-25 — Mirrored Reason-to-Strategy mapping for frontend and seed import

- **Context:** Finding 5 of the architecture review: `docs/requirements.md` §6.5 defines Reason Codes, Strategy Codes, display labels and the Reason→Strategy mapping. The frontend needs it for Strategy Suggestions and Basic Insights ordering; the seed import needs it to translate display labels into codes, but only `apps/web/src/catalog.ts` was documented.
- **Decision:** Keep the catalog in `apps/web/src/catalog.ts` with Reason Codes, Strategy Codes, display labels and the Reason→Strategy mapping. Mirror the same five-row mapping in `apps/api/scripts/seed.ts` for seed translation. Both are copied from §6.5; do not add a shared workspace package, a JSON catalog file or a catalog API endpoint in the MVP.
- **Options considered:** (1) Mirror the mapping in the frontend catalog and the seed script; no new workspace or config and simplest for five rows, but two copies can drift. (2) Shared workspace package `packages/catalog` imported by both apps; one typed source, but a new workspace with tsconfig/build wiring. (3) Root `data/catalog.json` imported by both apps; one data file, but JSON import configuration in Vite/TypeScript and Node.
- **Rationale:** The user explicitly selected mirroring the mapping in two modules.
- **Alternatives:** The shared package and root JSON file were not chosen because they add build/configuration structure disproportionate to five fixed rows.
- **Consequences:** No cross-app dependency or new tooling; a catalog change must be applied to both modules and to `docs/requirements.md` §6.5, so reviewers must check them together. UI labels remain independent of stored codes.

### 2026-11-25 — Checked-in SQL migration for the Demo Data schema

> **Superseded detail:** timestamp columns are now `TIMESTAMPTZ(3)` mapped to ISO strings; see the newest entry above.

- **Context:** Finding 3 of the architecture review: `pg` with raw parameterized SQL is already decided, but `data-model.md` explicitly left physical table names, column types and schema creation undefined, so `/implement` would have to guess the database layout.
- **Decision:** Define the Neon Demo Data schema in a checked-in SQL file at `apps/api/db/001_demo_entries.sql`, applied manually before `npm run db:seed`. Use two tables: `demo_procrastination_entries` (`id UUID PRIMARY KEY`, `task_description TEXT NOT NULL`, `procrastination_reason TEXT NOT NULL`, `recorded_at TEXT NOT NULL`, `owner_id TEXT NOT NULL`) and `demo_strategy_attempts` (`id UUID PRIMARY KEY`, `entry_id UUID NOT NULL UNIQUE REFERENCES demo_procrastination_entries(id)`, `strategy_code TEXT NOT NULL`, `feedback TEXT NULL`, `feedback_recorded_at TEXT NULL`). Store UTC timestamps as ISO 8601 strings with milliseconds in `TEXT` columns so `pg` rows map directly to the API strings; do not use `TIMESTAMP`/`TIMESTAMPTZ`. Database checks constrain `feedback` to `helped`, `partially_helped`, `not_helped` or `NULL`, require `feedback_recorded_at` exactly when `feedback` is non-null, and keep `owner_id` as `public_demo`. Text length and trimming rules are enforced on import, not in SQL. The repository and seed import map these column names to the entity fields in `data-model.md`.
- **Options considered:** (1) Checked-in SQL schema/migration file applied manually; reviewable and reproducible, but one more file and a manual step. (2) `CREATE TABLE IF NOT EXISTS` executed by the seed command; always present when seeding and fewer files, but DDL is mixed into data code and harder to version. (3) Create tables in the Neon dashboard and list columns/types in docs; fewest files, but unreproducible and prone to drift.
- **Rationale:** The user explicitly selected the checked-in SQL migration file.
- **Alternatives:** The seed-command DDL and dashboard-only options were not chosen because mixing DDL with data import and manual environment setup would make schema drift harder to detect; fake-based tests would not catch it.
- **Consequences:** The SQL in `repositories/demo-entries.ts` has a single reviewable source of truth; schema changes require a new migration file and a `data-model.md` update. `TEXT` timestamps avoid `Date` conversion but give up database date functions, which the MVP does not need. The seed script location and idempotency mechanism are recorded in the next entry.

### 2026-11-25 — Stable seed key and upsert for idempotent Demo Data seeding

> **Superseded detail:** `seed_key` now comes from an explicit immutable `seedKey` in `data/seed.json`; see the newest entry above.

- **Context:** Finding 4 of the architecture review: `decisions.md` requires a manual, idempotent `npm run db:seed`, but no script file is documented and "IDs generated on import" has no mechanism that prevents duplicates on rerun.
- **Decision:** Implement the seed command in `apps/api/scripts/seed.ts`, wired from the root `npm run db:seed`. Add `seed_key TEXT NOT NULL UNIQUE` to `demo_procrastination_entries` in `apps/api/db/001_demo_entries.sql`; derive it as the trimmed seed `taskDescription` plus `|` plus the seed's date-only `recordedOn`. On every run, convert display labels to codes and date-only values to `00:00:00.000Z` UTC strings, generate a random UUID for each new entry, and use `INSERT ... ON CONFLICT (seed_key) DO UPDATE` so reruns refresh Demo Data without duplicating it or changing entry `id`s. For a seed row's optional `strategyAttempt`, upsert `demo_strategy_attempts` on its unique `entry_id`, preserving an existing attempt `id`; when the seed row has no attempt, write none. Use only `pg` and Node built-ins; no new dependency.
- **Options considered:** (1) Stable seed key plus upsert at `apps/api/scripts/seed.ts`; deterministic identity without a new dependency or destructive writes, but one extra schema column and mapping rule. (2) Deterministic UUIDs derived from seed rows plus upsert on `id`; no extra column, but a UUID-v5 helper/dependency is needed and IDs change if seed identity changes. (3) Truncate both Demo Data tables and reinsert all 14 entries on every run; simplest, but destructive, changes IDs and exposes a brief read gap.
- **Rationale:** The user explicitly selected the stable seed key plus upsert.
- **Alternatives:** Deterministic UUIDs were not chosen because they require extra UUID handling and unstable IDs; delete-and-reinsert was not chosen because it is destructive and briefly leaves Demo Data unavailable.
- **Consequences:** Seeding is repeatable and tolerates display-label changes, but each seed row needs a stable `taskDescription`/`recordedOn` identity. The migration gains one column; attempts are refreshed in place rather than duplicated. Personal browser data is unaffected.

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
- **Decision:** Store personal `ProcrastinationEntry` and `StrategyAttempt` records only in the user's browser, where they survive a page refresh. Use Neon PostgreSQL only for fictional Demo Data seeded from `data/seed.json`, exposed read-only through `GET /api/demo/entries`. Never send personal entries, attempts or feedback to the API. If browser storage is unavailable or full, keep records in memory for the session, show the warning defined in `docs/requirements.md` only once, and do not call the API.
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
