# AGENTS.md

Instructions for AI coding agents working in this repository.

## Project

A full-stack TypeScript starter. It contains working infrastructure but no product features yet. The application to build is described in `docs/requirements.md`.

```
apps/web/   React 19 + Vite + Tailwind CSS 4 frontend (Vitest + React Testing Library)
apps/api/   Express 5 API on Node.js, a single file: src/server.ts (Vitest + Supertest)
e2e/        Playwright end-to-end tests (playwright.config.ts at the root)
docs/       requirements.md (what to build), architecture/ (overview, rules and decisions), glossary.md (domain terms),
            development.md (how to run),
            features/ (planned features: do not implement unless asked)
.opencode/  OpenCode agents: analyst (requirements, docs only), subagents frontend, backend, reviewer;
            commands /plan-feature, /implement, /verify
```

- npm workspaces, one `package.json` at the root. Run commands from the root.
- The browser calls the API with relative `/api/...` URLs. In development Vite (port 5173) proxies them to Express (port 3001). In production (`npm run build && npm start`) Express serves the built frontend and the API on one port.

## Before implementing a feature

1. Read `docs/requirements.md`. Implement only what it asks for. If it is empty or unclear, ask instead of guessing.
2. Read `docs/architecture/README.md` and `docs/architecture/decisions.md` and follow the decisions recorded there. Files in `docs/features/` describe planned features: keep the design compatible with them, but do not build them until asked.
3. Read the code you are about to change.
4. For anything larger than a small fix, propose a short plan first (or use `/plan-feature`).

## Conventions

- Use the terms from `docs/glossary.md` exactly — in code, UI texts and tests. Do not mix up terms the glossary marks as different. If a needed term is missing, ask instead of inventing one.
- TypeScript strict mode everywhere. Avoid `any`; type API request and response shapes.
- Prefer simple, readable code over abstractions. Do not add files, layers or features "for later".
- Do not add dependencies unless clearly needed. Explain every new dependency and why it is needed.
- Use the data storage, hosting and other technical decisions recorded in `docs/architecture/decisions.md`. If a decision the task needs is missing, propose options and ask instead of choosing silently.
- Secrets (database URLs, API keys) come from environment variables. Never hard-code them, never commit `.env`, and keep `.env.example` up to date with variable names and fake values.
- Frontend: function components and hooks, Tailwind utility classes, handle loading/error/empty states, accessible semantic HTML.
- Backend: all routes under `/api`, validate input, return JSON errors with correct status codes. Node.js runs the `.ts` files directly, so relative imports use the `.ts` extension and only erasable TypeScript syntax is allowed (no `enum`, `namespace` or parameter properties).
- Keep `export const app`, `serveFrontend` and the `import.meta.main` check in `apps/api/src/server.ts` so tests can import the app without starting a server and production can serve the frontend.
- Put tests next to the code: `*.test.ts(x)`. E2E tests go in `e2e/*.spec.ts`.

## Testing expectations

- Every new behaviour gets tests: success, error and edge cases.
- API endpoints: Supertest tests in `apps/api/src`.
- UI: React Testing Library tests that query by role, label or text; mock `fetch` with `vi.stubGlobal`.
- Important user flows across frontend and backend: a Playwright test in `e2e/`.

## Verification commands

```bash
npm run lint        # ESLint for both apps
npm run typecheck   # TypeScript for both apps and e2e
npm run test        # Vitest unit tests for both apps
npm run build       # production builds
npm run test:e2e    # Playwright (starts both servers automatically)
npm run verify      # lint + typecheck + test + build
```

Use `-w apps/web` or `-w apps/api` to run a script for one app only. Do not run `npm run dev` yourself: it starts long-running servers and never exits.

## After implementing

- Update `docs/architecture/decisions.md` and the relevant files in `docs/architecture/` after significant changes (new endpoints, data model, storage, folder structure, dependencies).
- Run the relevant verification commands and fix failures you caused.
- Report honestly: list the commands you ran with their real results. Never claim tests passed unless you ran them. Clearly state anything incomplete, skipped or failing.
- Do not commit or push unless the user asks. Never run `git push`.
