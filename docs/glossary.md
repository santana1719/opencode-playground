# UNSTUCK — Glossary

**Status:** Draft

This glossary defines the key terms used in UNSTUCK. Use these names consistently in requirements, code, UI, and tests.

## 1. Users and Records

| Term | Meaning in this app | Not to be confused with | In code |
|---|---|---|---|
| User | A person using UNSTUCK. | User Account: not available in the MVP. | User (role only) |
| Procrastination Entry | A record of a postponed task and its possible cause. | Task Description: only the text describing the task. | `ProcrastinationEntry` |
| Task Description | The activity the user is postponing. | Procrastination Entry: the complete record. | `taskDescription` |
| Procrastination Reason | A possible reason selected by the user. | A confirmed psychological diagnosis. | `ProcrastinationReason` |
| Reason Code | A stable identifier for a Procrastination Reason, stored in `procrastinationReason`. | The display label of the Procrastination Reason, which may change. | `procrastinationReason` |
| Demo Data | Fictional Procrastination Entries and Strategy Attempts loaded from `data/seed.json` for the public demo. | Real user data: no real entries are stored in the public database without authentication. | `data/seed.json` |
| Owner ID | A stable identifier on stored records, reserved for future User Accounts. | User Account or authentication; in the MVP it is not a security boundary or user isolation. | `ownerId` |

## 2. Strategies and Feedback

| Term | Meaning in this app | Not to be confused with | In code |
|---|---|---|---|
| Strategy | A predefined action that may help the user get started. | Strategy Attempt: a record of trying it. | `Strategy` |
| Strategy Suggestion | A strategy recommended for a selected reason. | Strategy Feedback: the result after trying it. | `StrategySuggestion` |
| Strategy Attempt | A record linking an entry to a suggested strategy and feedback. | Strategy: the action itself. | `StrategyAttempt` |
| Strategy Feedback | The user's evaluation of whether a strategy helped. | Strategy Suggestion: the recommendation. | `StrategyFeedback` |
| Strategy Code | A stable identifier for a Strategy, stored in a Strategy Attempt. | The display label of the Strategy; a Strategy Attempt. | `strategyCode` |

## 3. Insights and Progress

| Term | Meaning in this app | Not to be confused with | In code |
|---|---|---|---|
| Reason Frequency | The number of entries with a particular reason. | Start Rate: a percentage of successful attempts. | `ReasonFrequency` |
| Basic Insights | The MVP dashboard showing common procrastination reasons. | Your Growth: the future progress dashboard. | `BasicInsights` |
| Your Growth | A future dashboard showing progress over time. | Basic Insights: statistics about reasons only. | `YourGrowth` |
| Start Rate | The percentage of evaluated attempts marked as Helped. | Task completion rate: not tracked in UNSTUCK. | `StartRate` |

## 4. Naming Rules

- The MVP has two stored entities: `ProcrastinationEntry` and `StrategyAttempt`.
- Reasons and strategies are predefined values, not separate database entities.
- Each entry can have one strategy attempt in the MVP.
- Feedback can be `helped`, `partially_helped`, `not_helped`, or `null` if not evaluated.
- User accounts and Your Growth are planned for future versions.
- Reported reasons are user-selected, not medical diagnoses.
- Reason Codes and Strategy Codes are stable identifiers; display labels are not used as identities.
- Demo Data is fictional; real user data is not stored in the public database in the MVP.
