# Product Requirements

> **Status:** draft

## 1. Product overview


## 1. Product overview

**UNSTUCK** is a self-reflection web application that helps people identify possible reasons for procrastination and discover practical strategies to start postponed tasks.

Unlike traditional productivity apps, UNSTUCK focuses on understanding personal obstacles and recognizing behavioral patterns rather than tracking completed tasks.

## 2. Users

The primary target audience includes:

- **Students** who struggle to begin assignments or prepare for exams.
- **Self-learners** working on courses and personal projects.
- **Individuals interested in self-improvement** who want to better understand their procrastination habits.

**MVP role:** A single user without registration or authentication.

## 3. User stories

1. **Record procrastination:** As a user, I want to record a task I am postponing and select a possible reason, so that I can better understand what prevents me from starting.

2. **Get a strategy:** As a user, I want to receive a practical strategy based on my selected reason, so that I can take a small, manageable first step.

3. **Evaluate the strategy:** As a user, I want to indicate whether the suggested strategy helped, partially helped, or did not help, so that I can reflect on what works for me.

4. **Identify patterns:** As a user, I want to see a chart of my most frequently reported procrastination reasons, so that I can recognize recurring obstacles.

## 4. Scope

### In scope (MVP)

- Record a Procrastination Entry with a task description and a selected Procrastination Reason.
- Choose a reason from a predefined list.
- Receive a rule-based Strategy Suggestion based on the selected reason.
- Record Strategy Feedback: helped, partially helped, or not_helped.
- Save entries and feedback so they remain available after refreshing the page.
- View Basic Insights showing Reason Frequency in a chart.
- Display clear validation messages, loading states, and empty states.
- Provide a responsive interface for desktop and mobile browsers.

### Out of scope

- User Accounts, registration, and authentication.
- Multi-user support.
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

<!--
For each user story, list checks that prove it works. They become your tests.

Story 1:
- Given ..., when ..., then ...
- ...
-->

## 6. Data

<!--
What information does the application store or show? List the main entities and their fields.
Example: Task — title (required, max 100 characters), due date (optional), done (true/false).
Where the data is stored is an architecture decision: see docs/architecture.md.
-->

## 7. Open questions

<!-- Things you have not decided yet. -->
