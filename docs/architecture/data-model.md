# Data model

This file records the agreed shape of the two stored entities. It does not choose physical SQL table names, column types, indexes or constraints beyond what is recorded here.

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

- If browser storage is unavailable or full, keep records in memory for the current session, show the warning defined in `docs/requirements.md`, and send nothing to the API.
- If stored data is malformed or missing required fields, skip the invalid records, show the corrupted-data warning, and continue with the valid records.

## Database access

Demo Data rows are read from Neon with `pg` and explicit parameterized SQL in the repository; `DATABASE_URL` comes from the environment. No ORM or query builder is used. Physical table names, column types, indexes and constraints beyond the keys and rules above are not decided here. Personal data is never written to Neon.
