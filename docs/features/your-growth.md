# Your Growth

> **Status:** planned — do not implement as part of the MVP

## 1. Feature Overview

Your Growth is a personal progress dashboard for UNSTUCK.

It helps users track how their ability to start postponed tasks changes over time and discover which strategies work best for them.

## 2. User Stories

1. As a user, I want to see how often strategies helped me start tasks, so that I can track my progress.

2. As a user, I want to compare my Start Rate across different weeks, so that I can recognize improvements over time.

3. As a user, I want to identify my most helpful strategies, so that I can make better decisions in the future.

## 3. Acceptance Criteria

- Calculate Start Rate as the percentage of evaluated Strategy Attempts marked `helped`.
- Exclude attempts without feedback from calculations.
- Group evaluated attempts by the date their feedback was recorded.
- Display a weekly progress chart.
- Allow users to compare their results across weeks.
- Show the number of evaluated attempts behind each percentage.
- Display the most helpful strategies based on recorded feedback.
- Show positive messages when the data indicates improvement.
- Handle missing or insufficient data correctly.
- Do not present results as psychological diagnoses.

## 4. Affected Application Areas

### Data

Reuse the existing entities:
- `ProcrastinationEntry`
- `StrategyAttempt`

No new main entities are required.

### API

Add endpoints for:
- Weekly Start Rate statistics.
- Strategy effectiveness calculations.
- Progress comparisons over time.

### UI

Create a Your Growth dashboard with:
- Progress summary cards.
- Weekly progress chart.
- Most helpful strategies.
- Data-based progress insights.

### Roles

The application continues to support a single User without authentication.

User Accounts may be added in a future version.

## 5. Out of Scope

- AI-generated psychological analysis.
- Medical diagnoses.
- Social comparisons and leaderboards.
- Daily streaks and gamification.
- User registration and authentication.
