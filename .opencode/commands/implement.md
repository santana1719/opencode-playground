---
description: Implement a feature or the approved plan. Usage /implement [feature, plan step, or nothing to use the approved plan]
agent: build
---

Implement the following:

$ARGUMENTS

If the text above is empty, implement the most recent plan that was approved in this conversation. If there is no plan in this conversation and no description above, ask me what to implement and stop.

Follow these steps:

1. **Read first.** Read `AGENTS.md`, the relevant parts of `docs/requirements.md`, `docs/architecture/README.md`, `docs/architecture/decisions.md` and the relevant architecture section file, the plan (if there is one), and the code you will change. If the request contradicts the requirements or the plan, stop and ask before writing code.
2. **Implement in small steps.** Make the smallest change that fully delivers the feature. Follow the existing code style and conventions from `AGENTS.md`. Do not add dependencies, abstractions or features that were not requested.
3. **Delegate when it helps.** For substantial work, delegate to the `backend` subagent (Express API in `apps/api`) and the `frontend` subagent (React UI in `apps/web`). Do the backend first when the frontend depends on a new endpoint, and give each subagent the exact API contract (method, path, request and response shape). Do small or trivial changes yourself.
4. **Test.** Add or update unit tests for the new behaviour (success, error and edge cases). Add or extend a Playwright test in `e2e/` when the feature is an important user flow across frontend and backend.
5. **Document.** If you made a significant design decision (new endpoint group, data model, storage, new dependency, new folder structure), update `docs/architecture/decisions.md` and the relevant files in `docs/architecture/`.
6. **Verify.** Run `npm run lint`, `npm run typecheck`, `npm run test` and `npm run build`, and `npm run test:e2e` if you touched a user flow. Fix failures caused by your changes and run the checks again.
7. **Summarize.** Report:
   - what was implemented and which acceptance criteria are met,
   - files created or changed,
   - each check you ran with its real result (PASS/FAIL), and checks you did not run with the reason,
   - anything incomplete, known problems, and suggested next steps.

Never claim a check passed unless you ran it in this session. Do not commit or push; I will review and commit the changes myself.
