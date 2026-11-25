# Data model

This file records the agreed shape of the two stored entities and the physical Neon table and column names for Demo Data. No other storage layout is assumed.

## Stored entities

| Entity | Personal copy | Demo Data copy | Notes |
| --- | --- | --- | --- |
| `ProcrastinationEntry` | Browser storage | Neon PostgreSQL | Same shape in both places; fictional Demo Data is seeded from `data/seed.json`. |
| `StrategyAttempt` | Browser storage | Neon PostgreSQL | At most one Strategy Attempt per Procrastination Entry. |

Procrastination Reasons and Strategies are predefined values, not stored entities. Personal `ProcrastinationEntry` and `StrategyAttempt` records are never sent to the API or Neon. `ownerId` is always `public_demo` and is not a security boundary.

## `ProcrastinationEntry` table

| Field | Type | Required | Validation / rules |
| --- | --- | --- | --- |
| `id` | string (UUID) | Yes | Generated on import for Demo Data and locally for personal entries; unique. |
| `taskDescription` | string | Yes | Trimmed before storage; 1–500 characters. Corresponds to `taskDescription` in the seed. |
| `procrastinationReason` | Reason Code (string) | Yes | Must be one of the catalog values in `docs/requirements.md`. Seed display labels are converted to Reason Codes on import. |
| `recordedAt` | UTC timestamp (ISO 8601 string with milliseconds) | Yes | Personal entries: creation time. Demo Data: the seed's date-only `recordedOn` at `00:00:00.000Z`. |
| `ownerId` | Owner ID (string) | Yes | Always `public_demo` in the MVP. Reserved for future User Accounts; not a security boundary. |

## `StrategyAttempt` table

| Field | Type | Required | Validation / rules |
| --- | --- | --- | --- |
| `id` | string (UUID) | Yes | Generated when the attempt is created; unique. |
| `entryId` | string (UUID) | Yes | References `ProcrastinationEntry.id`; at most one Strategy Attempt per entry. |
| `strategyCode` | Strategy Code (string) | Yes | Must be the Strategy mapped to the entry's Reason Code. Seed display labels are converted to Strategy Codes on import. |
| `feedback` | Strategy Feedback (`helped`, `partially_helped`, `not_helped`, or `null`) | No | `null` until evaluated; recorded once and never changed afterwards. |
| `feedbackRecordedAt` | UTC timestamp (ISO 8601 string with milliseconds) or `null` | No | Required when `feedback` is not `null`. Demo Data: the seed's date-only `feedbackRecordedOn` at `00:00:00.000Z`. `null` otherwise. |

## Relations and keys

- `ProcrastinationEntry.id` is the unique key for an entry.
- `StrategyAttempt.id` is the unique key for an attempt.
- `StrategyAttempt.entryId` references `ProcrastinationEntry.id` and is unique; an entry has zero or one attempt in the MVP.
- Reason Codes and Strategy Codes are immutable identifiers; display labels may change only in the UI.
- `ownerId` prepares the data model for future User Accounts but provides no isolation on its own.

## Constraints

- `id` values are generated UUIDs and unique within their entity.
- `taskDescription` is trimmed before storage and must be 1–500 characters.
- `procrastinationReason` must be one of the catalog Reason Codes in `docs/requirements.md`; the UI never offers an out-of-catalog value.
- `strategyCode` must be the Strategy mapped to the entry's Reason Code.
- `feedback` is `helped`, `partially_helped`, `not_helped` or `null`; once recorded it cannot change.
- `feedbackRecordedAt` is required when `feedback` is not `null` and is `null` otherwise.
- `recordedAt` and non-null `feedbackRecordedAt` are UTC ISO 8601 strings with milliseconds.
- `ownerId` is always `public_demo` in the MVP.
- Seed display labels and date-only values are converted on import to stable codes and `00:00:00.000Z` timestamps.

## Storage edge behavior

- If browser storage is unavailable or full, keep records in memory for the current session, show the warning defined in `docs/requirements.md` only once, and send nothing to the API.
- If stored data is malformed or missing required fields, skip the invalid records, show the corrupted-data warning, and continue with the valid records. `storage.ts` applies the pure stored-record guards in `apps/web/src/validation.ts` on load; the exact rule set is recorded in `decisions.md`.

## Database access

Demo Data rows are read from Neon with `pg` and explicit parameterized SQL in the repository; `DATABASE_URL` comes from the environment. No ORM or query builder is used. Personal data is never written to Neon.

## SQL schema (Neon Demo Data)

The schema is defined in the checked-in `apps/api/db/001_demo_entries.sql`, applied manually before `npm run db:seed` (see `decisions.md`). Timestamps are `TIMESTAMPTZ(3)` instants; `pg` returns `Date` objects, and the repository converts them with `toISOString()` to the ISO 8601 strings with milliseconds used by the API and the in-memory fake. The seed importer converts date-only values to `YYYY-MM-DDT00:00:00.000Z` before insert. The repository and seed import map these snake_case columns to the camelCase entity fields above.

`demo_procrastination_entries`

| Column | SQL type | Notes |
| --- | --- | --- |
| `id` | `UUID PRIMARY KEY` | Generated on import; unchanged by reseeding. |
| `seed_key` | `TEXT NOT NULL UNIQUE` | Seed identity for idempotent import: copied from the immutable `seedKey` field in `data/seed.json`; import metadata only, not part of the API entity. |
| `task_description` | `TEXT NOT NULL` | Maps to `taskDescription`; trimmed and 1–500 characters on import. |
| `procrastination_reason` | `TEXT NOT NULL` | Maps to `procrastinationReason`; Reason Code from the catalog. |
| `recorded_at` | `TIMESTAMPTZ(3) NOT NULL` | Maps to `recordedAt`; read as `Date` then converted to an ISO 8601 string with milliseconds by the repository. |
| `owner_id` | `TEXT NOT NULL` | Maps to `ownerId`; `public_demo`, enforced by a database check. |

`demo_strategy_attempts`

| Column | SQL type | Notes |
| --- | --- | --- |
| `id` | `UUID PRIMARY KEY` | Generated when the attempt is created. |
| `entry_id` | `UUID NOT NULL UNIQUE` | Maps to `entryId`; foreign key to `demo_procrastination_entries(id)`, enforcing one attempt per entry. |
| `strategy_code` | `TEXT NOT NULL` | Maps to `strategyCode`; Strategy Code mapped from the entry's Reason Code. |
| `feedback` | `TEXT NULL` | Maps to `feedback`; `helped`, `partially_helped`, `not_helped` or `NULL`, enforced by a database check. |
| `feedback_recorded_at` | `TIMESTAMPTZ(3) NULL` | Maps to `feedbackRecordedAt`; read as `Date` then converted to an ISO 8601 string with milliseconds by the repository; non-null exactly when `feedback` is non-null, enforced by a database check. |
