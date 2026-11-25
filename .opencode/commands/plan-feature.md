---
description: Plan a feature without changing code. Usage /plan-feature <feature description>
agent: plan
---

Create an implementation plan for this feature. Do NOT implement it and do not change any project files.

Feature request:

$ARGUMENTS

If the feature request above is empty, ask me to describe the feature and stop.

Work through these steps:

1. **Read the docs.** Read `AGENTS.md`, `docs/requirements.md`, `docs/architecture/README.md` and `docs/architecture/decisions.md`, then the relevant architecture section file and the source files in `apps/web/src` and `apps/api/src` that the feature touches.
2. **Analyse the request.** Restate the feature in one or two sentences and check that it fits the scope in `docs/requirements.md`. If it is outside the MVP scope, say so.
3. **Split responsibilities.** List what the frontend (`apps/web`) must do and what the backend (`apps/api`) must do.
4. **Propose the design.** Describe new or changed components, modules and data. For every API change give the method, path, request body, response body and error status codes. Prefer the simplest design that meets the requirements, and name any new dependency together with the reason it is needed.
5. **Define acceptance criteria.** Write a numbered list of observable, testable criteria ("Given … when … then …" is fine).
6. **Plan implementation and verification.** Break the work into small ordered steps that can each be finished and checked separately. For each step name the files involved, the tests to add, and the checks to run (`npm run lint`, `npm run typecheck`, `npm run test`, `npm run build`, `npm run test:e2e`). Mention which steps fit the `frontend` or `backend` subagent.
7. **Highlight ambiguities.** List open questions and risky assumptions, and recommend a default for each.

Finish with: "Review this plan. When it is ready, switch to the build agent and run `/implement` (or `/implement <step>` to do one step at a time)." Also mention whether `docs/requirements.md` or the relevant files in `docs/architecture/` should be updated before implementation.
