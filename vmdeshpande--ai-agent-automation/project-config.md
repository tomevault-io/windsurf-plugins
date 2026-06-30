---
trigger: always_on
description: Monorepo with three packages that are NOT linked via workspaces — each has its own `node_modules`:
---

# AGENTS.md

## Repo Structure

Monorepo with three packages that are NOT linked via workspaces — each has its own `node_modules`:

- `backend/` — Node.js/Express, **CommonJS** (`"type": "commonjs"`), entry point `server.js`
- `frontend/` — Next.js 16 + React 19 + TypeScript, entry point `src/app/`
- `infra/` — Docker Compose stack (MongoDB, backend, worker, frontend, nginx)

## Dev Commands

**Backend** (two processes required):

```bash
cd backend
cp .env.example .env   # fill in JWT_SECRET, MONGO_URI, LLM keys
npm install
npm run dev             # API server on :5000
npm run worker          # agent runner (separate terminal)
```

**Frontend**:

```bash
cd frontend
npm install
npm run dev             # Next.js on :3000
```

**Lint & format** (run from repo root):

```bash
npm run lint            # eslint frontend/src + backend/src
npm run format          # prettier --write frontend/src backend/src
```

**Backend tests**:

```bash
cd backend
npm test                # runs jest --testMatch '**/*.handler.test.js'
```

Only handler test files are included. There is no frontend test suite.

## Git Hooks

Husky runs on commit:

- **pre-commit**: `lint-staged` → eslint --fix + prettier --write on staged files
- **commit-msg**: commitlint enforces [Conventional Commits](https://www.conventionalcommits.org/)

Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
Header max 100 chars, subject lowercase, no trailing period.

## Code Style

- **Backend JS**: `prefer-const`, `eqeqeq`, no `var`, no `throw literal`, no `return await`
- **Frontend TS**: `@typescript-eslint/consistent-type-imports` enforced, `no-explicit-any` as warning, React hooks rules enforced
- **Prettier**: semi, singleQuote, tabWidth 2, trailingComma es5, printWidth 100, bracketSpacing, arrowParens always

## Architecture Notes

- Backend and frontend are separate processes communicating over REST + MongoDB
- MongoDB must run with **replica set** enabled (Docker compose handles this via `mongo-init-replica`)
- Worker (`npm run worker` / `node src/agents/runner.js`) polls for tasks — must run alongside the API server
- `backend/src/` layout: `agents/`, `config/`, `controllers/`, `models/`, `routes/`, `services/`, `tools/`, `workflow/`
- Frontend API URL in Docker is derived from `BACKEND_PORT` — do NOT set `NEXT_PUBLIC_API_URL` manually for Docker deploys
- Env vars are never committed; `backend/.env.example` is the source of truth for local dev, `infra/.env.example` for Docker

## Architecture & Topic Guide

Deep docs live in `docs/`. By topic:

- **Workflow engine architecture** — [docs/architecture.md](docs/architecture.md), [docs/workflow-engine.md](docs/workflow-engine.md)
- **Runner & executor flow** — [docs/workflow-engine.md](docs/workflow-engine.md) (execution lifecycle + step flow); worker entry `backend/src/agents/runner.js`
- **Workflow graph / edge routing** — [docs/workflow-builder.md](docs/workflow-builder.md), [docs/architecture.md](docs/architecture.md) (branch routing)
- **Agent memory system** — [docs/semantic-memory.md](docs/semantic-memory.md), [docs/agent-system.md](docs/agent-system.md)
- **Document RAG** — [docs/document-intelligence-rag.md](docs/document-intelligence-rag.md)
- **Workflow builder conventions** — [docs/workflow-builder.md](docs/workflow-builder.md)
- **Tools & MCP** — [docs/automation-tools.md](docs/automation-tools.md), [docs/how-to-add-custom-tool.md](docs/how-to-add-custom-tool.md), [docs/mcp-integration.md](docs/mcp-integration.md)
- **Contribution workflow & commit style** — [CONTRIBUTING.md](CONTRIBUTING.md)
- **Ops** — [docs/docker-deployment.md](docs/docker-deployment.md), [docs/telemetry.md](docs/telemetry.md), [docs/privacy.md](docs/privacy.md), [docs/POSTMAN.md](docs/POSTMAN.md)

## Coding Conventions

Repository-specific patterns to follow (lint/prettier rules are under [Code Style](#code-style); contribution process is in [CONTRIBUTING.md](CONTRIBUTING.md)):

- **Backend layering**: `controllers/` → `routes/` → `services/` → `models/`; step execution under `agents/` (`handlers/`, `executor.js`, `runner.js`); tools in `tools/`; node catalog in `workflow/`
- **Controllers** return via `sendOK(res, payload)` / `sendError(res, code, key)`; wrap async in try/catch logging `"<fn> error"`; enforce ownership with `resource.userId.toString() !== req.user._id.toString()` → 403. Error keys are lowercase snake_case (`name_required`, `not_found`, `forbidden`)
- **Routes**: `router.use(authMiddleware)` then declare specific paths before `/:id` params
- **Step handlers** export `execute(step, context, agent, validatedStepId, timeoutMs)` and are registered in the `backend/src/agents/executor.js` handler map (key = lowercased step type). New tools alternatively follow the `meta` + `run(step, context)` registry contract — see [docs/how-to-add-custom-tool.md](docs/how-to-add-custom-tool.md)
- **Frontend**: `'use client'` components, App Router under `app/`, shared fetch client `frontend/src/lib/api.ts` (`api` / `apiGet` / `apiPost` / `apiPut` / `apiDelete`, Bearer token, 401 → `/login`), types in `frontend/src/types/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vmDeshpande/ai-agent-automation](https://github.com/vmDeshpande/ai-agent-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
