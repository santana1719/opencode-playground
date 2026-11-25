# Product Requirements

> **Status:** draft — MVP requirements ready

## 1. Product overview

**UNSTUCK** is a self-reflection web application that helps people identify possible reasons for procrastination and discover practical strategies to start postponed tasks.

Unlike traditional productivity apps, UNSTUCK focuses on understanding personal obstacles and recognizing behavioral patterns rather than tracking completed tasks.

## 2. Users

The primary target audience includes:

- **Students** who struggle to begin assignments or prepare for exams.
- **Self-learners** working on courses and personal projects.
- **Individuals interested in self-improvement** who want to better understand their procrastination habits.

**MVP role:** A single User without registration or authentication. The User can read fictional Demo Data but all personal Procrastination Entries and Strategy Attempts stay in that browser.

## 3. User stories

1. **Record procrastination:** As a user, I want to record a task I am postponing and select a possible reason, so that I can better understand what prevents me from starting.

2. **Get a strategy:** As a user, I want to receive a practical strategy based on my selected reason, so that I can take a small, manageable first step.

3. **Evaluate the strategy:** As a user, I want to indicate whether the suggested strategy helped, partially helped, or did not help, so that I can reflect on what works for me.

4. **Identify patterns:** As a user, I want to see a chart of my most frequently reported procrastination reasons, so that I can recognize recurring obstacles.

## 4. Scope

### In scope (MVP)

- Record a Procrastination Entry with a Task Description and a Procrastination Reason chosen from the predefined catalog.
- Validate the Task Description with the exact messages in §6.4 and keep the form usable after an error.
- Store personal Procrastination Entries and Strategy Attempts in the user's browser so they survive a page refresh.
- Show a Strategy Suggestion and record Strategy Feedback on the same entry card; feedback becomes read-only after it is saved.
- Load fictional Demo Data from Neon through the read-only `GET /api/demo/entries` endpoint and show it in a separate `Demo Data` section.
- Keep `Demo Data` and `My entries` as separate sections with distinct headings; only `My entries` is interactive.
- Show `Basic Insights` with Reason Frequency calculated from personal entries only.
- Show the exact loading, empty, error and storage-warning messages in §6.4.
- Keep personal entries, the form and `Basic Insights` usable when the Demo Data request fails.
- Render a responsive layout at the two test viewports in §6.5.

### Out of scope

- User Accounts, registration, and authentication.
- Multi-user support.
- Sending personal Procrastination Entries, Strategy Attempts or Strategy Feedback to an API or database.
- A separate feedback page; Strategy Feedback is entered on the entry card.
- A reset or clear control for browser-stored entries.
- A second Strategy Attempt per entry, retries, or re-requesting a Strategy Suggestion.
- Entries with a Reason Code outside the predefined catalog; the UI never offers one.
- Counting Demo Data in `Basic Insights`.
- Device-specific layouts beyond the two test viewports in §6.5.
- Payments, subscriptions, and paid APIs.
- AI-generated recommendations.
- Medical or psychological diagnoses.
- Push notifications and reminders.
- Daily streaks and gamification.
- Native mobile applications.
- Automatic detection of procrastination.
- Advanced progress analytics and weekly comparisons.
- **Your Growth**, a future progress dashboard described in [Your Growth](features/your-growth.md).

## 5. Acceptance criteria

### Story 1 — Record procrastination

- **Success.** Given no personal Procrastination Entries and a clock fixed at `2026-11-25T12:00:00.000Z`, when the user enters `  Finish the report  `, selects `Fear of failure`, and submits, then one entry is stored in the browser with `taskDescription: "Finish the report"`, `procrastinationReason: "fear_of_failure"`, `recordedAt: "2026-11-25T12:00:00.000Z"`, a generated UUID `id`, and `ownerId: "public_demo"`; it appears under `My entries` only, not under `Demo Data`, and is still present after a page refresh.
- **Description validation.** Given no personal entries, a Task Description of `   ` or 501 characters, and a valid reason, when the user submits, then no entry is stored, `Enter a task description.` or `Task description must be 500 characters or fewer.` is shown respectively, and `You have no saved entries yet.` remains under `My entries`.
- **Reason validation.** Given a trimmed 1–500 character Task Description and no selected reason, when the user submits, then no entry is stored, `Choose a procrastination reason.` is shown, and the form remains usable.
- **Empty personal history.** Given no personal entries but a successful Demo Data load, when the page renders, then `You have no saved entries yet.` is shown under `My entries`, all 14 entries from `data/seed.json` are shown under `Demo Data` with no interactive controls, and the two sections have distinct headings.

### Story 2 — Get a strategy

- **Initial state.** Given a personal entry with a valid Reason Code and no Strategy Attempt, when the user views its card under `My entries`, then `Get strategy` is available, no Strategy display text is shown, and no feedback choices are shown.
- **Success.** Given that entry with reason `fear_of_failure`, when the user selects `Get strategy`, then exactly one Strategy Attempt is stored with `entryId` equal to the entry `id`, `strategyCode: "write_imperfect_first_draft"`, `feedback: null`, `feedbackRecordedAt: null`, and a generated UUID; `Write an imperfect first draft` appears on the card and `Get strategy` is no longer offered.
- **No duplicate after refresh.** Given an entry with an existing Strategy Attempt, when the user refreshes the page and views the card, then the same Strategy display text is shown, no second Strategy Attempt exists in browser storage, and `Get strategy` is not offered.
- **Storage unavailable.** Given browser storage is unavailable, when the user selects `Get strategy` on an entry, then the mapped Strategy display text and feedback choices are shown for the current session, no attempt is persisted, and `Personal entries cannot be saved in this browser. Changes will last only until you close this tab.` is shown once.

### Story 3 — Evaluate the strategy

- **Not yet evaluated.** Given a Strategy Attempt with `feedback: null`, when the user views the card, then no saved feedback or feedback date is shown and the choices `Helped`, `Partially helped`, `Did not help` and `Save feedback` are available.
- **Success.** Given the attempt and a clock fixed at `2026-11-25T12:34:00.000Z`, when the user selects `Helped` and `Save feedback`, then the attempt has `feedback: "helped"` and `feedbackRecordedAt: "2026-11-25T12:34:00.000Z"`, the controls are disabled, `Helped` and the recorded date are shown, and both survive a page refresh. The same rule applies to `Partially helped` → `partially_helped` and `Did not help` → `not_helped`.
- **Validation.** Given an attempt with `feedback: null`, when the user selects `Save feedback` without choosing a value, then `Choose whether the strategy helped.` is shown and the attempt remains `feedback: null` with `feedbackRecordedAt: null`.
- **Read-only after recording.** Given an attempt with recorded feedback, when the user refreshes the page and tries to select another choice, then the choices are disabled, no `Save feedback` action is available, and the stored feedback and its date are unchanged.

### Story 4 — Identify patterns

- **Counts.** Given personal entries with reasons `feeling_overwhelmed`, `feeling_overwhelmed`, and `tiredness`, where one has no Strategy Attempt, and with Demo Data loaded, when the user refreshes the page and `Basic Insights` loads, then the chart shows `Feeling overwhelmed 2` followed by `Tiredness 1`, no other reason, and no Demo Data reason is counted. Reasons are ordered by count descending, then by catalog order in §6.5; a tie between `low_motivation` and `tiredness` therefore lists `Low motivation` first.
- **Empty personal history.** Given no personal entries and a successful Demo Data load, when `Basic Insights` loads, then `No personal entries yet, so there are no insights to show.` is shown, no chart is shown, and the `Demo Data` section is unaffected.
- **Loading Demo Data.** Given `GET /api/demo/entries` has not responded, when the page renders, then `Loading Demo Data...` is shown in the `Demo Data` section, no Demo Data cards are shown, and personal entries, the form and `Basic Insights` render from browser data.
- **Failed Demo Data load.** Given `GET /api/demo/entries` responds with `500`, when the page renders, then `Could not load Demo Data. Your entries are still available.` is shown in the `Demo Data` section with no Demo Data cards, while `My entries`, the form and `Basic Insights` remain usable.

## 6. Data, messages and fixtures

### 6.1 Storage locations

| Data | Location | Rules |
| --- | --- | --- |
| Demo Data | Neon PostgreSQL, seeded from `data/seed.json` | Fictional only; read through `GET /api/demo/entries`; never receives personal data. |
| Personal `ProcrastinationEntry` and `StrategyAttempt` | Browser storage | Survive page refresh; never sent to any API or database. |
| In-memory session copy | Browser memory | Used when browser storage is unavailable or full; discarded on refresh. |

### 6.2 `ProcrastinationEntry`

Two stored entities exist, as defined in `docs/glossary.md`: `ProcrastinationEntry` and `StrategyAttempt`. Procrastination Reasons and Strategies are predefined values, not stored entities. The fields below apply to both Demo Data and personal entries.

| Field | Type | Required | Validation / rules |
| --- | --- | --- | --- |
| `id` | string (UUID) | Yes | Generated on import for Demo Data and locally for personal entries; unique. |
| `taskDescription` | string | Yes | Trimmed before storage; 1–500 characters. Corresponds to `taskDescription` in the seed. |
| `procrastinationReason` | Reason Code (string) | Yes | Must be one of the catalog values below. Seed display labels are converted to Reason Codes on import. |
| `recordedAt` | UTC timestamp (ISO 8601 string with milliseconds) | Yes | Personal entries: creation time. Demo Data: the seed's date-only `recordedOn` at `00:00:00.000Z`. |
| `ownerId` | Owner ID (string) | Yes | Always `public_demo` in the MVP. Reserved for future User Accounts; not a security boundary. |

### 6.3 `StrategyAttempt`

| Field | Type | Required | Validation / rules |
| --- | --- | --- | --- |
| `id` | string (UUID) | Yes | Generated when the attempt is created; unique. |
| `entryId` | string (UUID) | Yes | References `ProcrastinationEntry.id`; at most one Strategy Attempt per entry. |
| `strategyCode` | Strategy Code (string) | Yes | Must be the Strategy mapped to the entry's Reason Code. Seed display labels are converted to Strategy Codes on import. |
| `feedback` | Strategy Feedback (`helped`, `partially_helped`, `not_helped`, or `null`) | No | `null` until evaluated; recorded once and never changed afterwards. |
| `feedbackRecordedAt` | UTC timestamp (ISO 8601 string with milliseconds) or `null` | No | Required when `feedback` is not `null`. Demo Data: the seed's date-only `feedbackRecordedOn` at `00:00:00.000Z`. `null` otherwise. |

### 6.4 Validation, edge behavior and exact UI strings

Validation rules:

- Task Description is required, trimmed before storage, and limited to 1–500 characters after trimming.
- A Procrastination Reason must be selected; the UI offers only the catalog below, so an out-of-catalog reason cannot be entered in the MVP.
- Strategy Feedback is one of `helped`, `partially_helped`, `not_helped`, or `null` until submitted.

Browser storage edge behavior:

- If browser storage is unavailable or a write fails because storage is full, show the storage warning once, keep the entry or attempt in memory for the current session, and send nothing to the API.
- If stored data is malformed or has missing required fields, skip the invalid records, show the corrupted-data warning, and continue with the valid records.
- No clear/reset control is provided in the MVP; tests clear browser storage between runs.

Exact strings to use in the UI and in tests:

| Purpose | Exact text |
| --- | --- |
| Empty Task Description | `Enter a task description.` |
| Task Description over 500 characters | `Task description must be 500 characters or fewer.` |
| Missing Procrastination Reason | `Choose a procrastination reason.` |
| Missing Strategy Feedback | `Choose whether the strategy helped.` |
| Storage unavailable or full | `Personal entries cannot be saved in this browser. Changes will last only until you close this tab.` |
| Corrupted stored data | `Some saved entries could not be read and were skipped.` |
| `My entries` empty | `You have no saved entries yet.` |
| `Basic Insights` with no personal entries | `No personal entries yet, so there are no insights to show.` |
| Demo Data loading | `Loading Demo Data...` |
| Demo Data request failed | `Could not load Demo Data. Your entries are still available.` |

Section headings are `Demo Data`, `My entries`, and `Basic Insights`. Action labels are `Get strategy` and `Save feedback`. Feedback choice labels are `Helped`, `Partially helped`, and `Did not help`.

### 6.5 Predefined catalog and fixtures

These are predefined catalog values, not stored entities.

| Procrastination Reason (display) | Reason Code | Strategy (display) | Strategy Code |
| --- | --- | --- | --- |
| Feeling overwhelmed | `feeling_overwhelmed` | Break the task into a five-minute step | `break_into_five_minute_step` |
| Fear of failure | `fear_of_failure` | Write an imperfect first draft | `write_imperfect_first_draft` |
| Low motivation | `low_motivation` | Commit to just two minutes | `commit_to_two_minutes` |
| Not knowing where to start | `not_knowing_where_to_start` | Write down the first concrete action | `write_down_first_concrete_action` |
| Tiredness | `tiredness` | Take a short restorative break | `take_short_restorative_break` |

`data/seed.json` is the canonical fictional test fixture: 14 Procrastination Entries, two with `strategyAttempt: null`, and one Strategy Attempt with `feedback: null` and `feedbackRecordedAt: null`. Display labels are converted to the codes above on import, and date-only `recordedOn` and `feedbackRecordedOn` values become UTC timestamps at `00:00:00.000Z`. Demo Data is read-only in the UI.

Feedback display labels map to stored values: `Helped` → `helped`, `Partially helped` → `partially_helped`, `Did not help` → `not_helped`.

Responsive test viewports are 360 × 640 (mobile) and 1280 × 800 (desktop). The layout must not require horizontal scrolling at either size.

### 6.6 Demo Data API contract

| Method | Path | Description | Response |
| --- | --- | --- | --- |
| GET | `/api/demo/entries` | Read fictional Demo Data from Neon | `200 { "entries": [ ... ] }`; `500 { "error": { "code": "DEMO_DATA_UNAVAILABLE", "message": "Could not load Demo Data." } }` |

Each item in `entries` uses the `ProcrastinationEntry` fields in §6.2 plus an optional `strategyAttempt` object with the `StrategyAttempt` fields in §6.3. There are no write endpoints in the MVP.

## 7. Deferred open questions

These are intentionally not resolved for the MVP:

- How to migrate personal browser data if real user data is ever supportable (requires User Accounts and authentication first).
- Whether to add a retry action for a failed Demo Data request.
- The human-readable date format used for `recordedAt` and `feedbackRecordedAt`; only the stored UTC format is fixed.
- The chart rendering approach and styling beyond the counts and ordering in Story 4.
- A reset/clear control for browser-stored entries (currently out of scope).
- Detailed Your Growth metrics and dashboards (described in `docs/features/your-growth.md`, not implemented).
