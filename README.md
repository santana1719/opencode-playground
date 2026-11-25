# AI-Ready Full-Stack Starter

A starter repository for learning **AI-first development** with [OpenCode](https://opencode.ai) and free AI models.

It gives you a working React + Express app with tests and a configured AI environment, but **no product features**. You come up with your own idea and build it with an AI coding agent: requirements → plan → code → checks → deploy.

Everything here is free. You do not need a paid AI subscription.

## The guide

Follow the steps in order. Every step ends with a concrete result you can check.

Step 2 is about the product: do it in any AI chat. **From Step 3 on, work in OpenCode** inside your repository: the agent reads your requirements and planned feature, explains the trade-offs and records decisions in your files.

| Step | What you do | Result |
| --- | --- | --- |
| [1. Setup](docs/guide/01-setup.md) | Create your repo, run the app, connect OpenCode | App runs, tests pass, the agent answers |
| [2. Idea and scope](docs/guide/02-idea-and-scope.md) | Pick a unique idea, a small MVP and the next feature (any AI chat) | Brief, `docs/glossary.md`, `docs/features/`, `data/seed.json` |
| [3. Data, key decisions and hosting](docs/guide/03-data-and-hosting.md) | Decide storage, key decisions and hosting; connect the database | Decisions in `docs/architecture/decisions.md`, `.env` |
| [4. Requirements](docs/guide/04-requirements.md) | Turn the brief into testable requirements with the agent | Complete `docs/requirements.md` |
| [5. Architecture](docs/guide/05-architecture.md) | Agree on data model, code structure and libraries with an architect agent | `docs/architecture/` folder, `architect` agent |
| [6. Build the MVP](docs/guide/06-build-the-mvp.md) | Plan → implement → verify, one story at a time | Working MVP, green checks, commits |
| [7. Add a feature](docs/guide/07-add-a-feature.md) | Add one substantial feature with the full AI workflow | Feature merged with tests and updated docs |
| [8. Deploy](docs/guide/08-deploy.md) *(optional)* | Put the app online for free | Public URL |
| [9. Portfolio](docs/guide/09-portfolio.md) | Make the repo describe **your** app, remove this guide | Portfolio-ready repository |

## Quick start

Requires Git and Node.js 22.18+.

```bash
npm install
npm run dev        # API on http://localhost:3001, frontend on http://localhost:5173
npm run test       # unit tests
npm run verify     # lint + typecheck + tests + build
```

Details: [docs/development.md](docs/development.md).

## What is inside

```
apps/web/        React 19 + Vite + Tailwind CSS frontend
apps/api/        Express 5 backend (one endpoint: GET /api/version)
e2e/             Playwright end-to-end tests
docs/            requirements.md, architecture/, glossary.md, development.md, guide/
.opencode/       AI agents (analyst; subagents frontend, backend, reviewer) and commands (/plan-feature, /implement, /verify)
AGENTS.md        rules for AI agents
opencode.jsonc   OpenCode configuration and permissions
```

React · Vite · Tailwind CSS · TypeScript · Node.js · Express · Vitest · React Testing Library · Supertest · Playwright · ESLint
