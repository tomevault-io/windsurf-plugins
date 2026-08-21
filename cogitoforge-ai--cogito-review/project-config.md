---
trigger: always_on
description: Instructions for AI coding agents working on this repository.
---

# AGENTS.md

Instructions for AI coding agents working on this repository.

## Naming

- Use `Cogito Review` as the product name in prose.
- Use `review`, `finding`, `repository integration`, `LLM provider`, `agent callback`, and `runtime provider` consistently with the existing docs.
- Keep literal code tokens exactly as they appear in source code, CLI commands, environment variables, API paths, and generated artifacts.
- Do not translate source-code identifiers, API names, or file names.

## Communication

- All source code, code comments, commit messages, and technical artifacts must use English.
- All chat messages to the user must use Vietnamese.
- Keep communication concise, direct, and action-oriented.
- When presenting plans, assumptions, risks, or implementation status, make them explicit so the user can review quickly.
- Do not use emoji in source code. Avoid overusing emoji in chat.

## Workflow

- Follow the repository workflow and AI contribution policy in [AI_POLICY.md](AI_POLICY.md).
- Do not duplicate or replace that workflow with ad hoc steps.
- If this file conflicts with `AI_POLICY.md`, surface the conflict to the user instead of guessing.

## Environment Setup

- Use the repository tooling directly. The standard local setup is `cp .env.example .env` followed by `make dev`.
- Install developer dependencies with `make install`.
- Use `uv` for Python package management and Node.js 22+ for frontend tooling.
- Prefer repository commands such as `make lint`, `make test`, and `make openapi` over custom one-off command chains.
- Use Docker Compose for stack lifecycle and integration-style local validation.
- Do not commit `.env`, tokens, generated secrets, or local machine credentials.

## Commands

- **Start the development stack:** `make dev`
- **Start the local prod-like stack from a repo checkout:** `make prod`
- **Stop the local prod-like stack:** `make prod-down`
- **Run database migrations:** `make migrate`
- **Roll back the latest database migration:** `make migrate-down`
- **Build the local agent image:** `make build-agent`
- **Render backend OpenCode config on the host:** `make render-opencode-config`
- **Regenerate backend OpenAPI and frontend API types:** `make openapi`
- **Run repository lint and frontend type checks:** `make lint`
- **Run the main test suite:** `make test`
- **Install dependencies and pre-commit hooks:** `make install`
- **Run backend integration tests:** `cd backend && uv run pytest -m integration`
- **Run backend unit tests only:** `cd backend && uv run pytest -m "not integration"`
- **Run shared tests only:** `cd shared && uv run pytest`
- **Run agent tests only:** `cd agent && uv run pytest`
- **Run frontend checks only:** `cd frontend && yarn test`

Run the smallest reliable validation scope first, then expand when the change crosses boundaries.

## Repository Structure

This repository is a monorepo with multiple runtime boundaries.

- `shared/` - cross-cutting Python package used by backend and agent
- `backend/` - FastAPI API, services, repositories, schemas, auth, worker entrypoints, and migrations
- `agent/` - isolated review runner, MCP server, toolbase integrations, and container entrypoints
- `frontend/` - React SPA with TanStack Router, TanStack Query, and generated OpenAPI types
- `operator/` - Kubernetes operator implementation
- `deploy/` - Docker, Helm, and raw Kubernetes deployment assets
- `docs/` - architecture, security, deployment, and product behavior documentation
- `website/` - documentation website sources
- `.github/workflows/` - CI, publish, operator, and website automation

Important generated or workflow-managed artifacts:

- `frontend/src/api/generated/schema.ts` - generated from `openapi.json`
- `openapi.json` - generated from the backend application
- Do not hand-edit generated artifacts when a generation workflow already exists.

## Architecture Boundaries

1. The backend is the source of truth for application state and persists data in PostgreSQL.
2. The worker consumes queued jobs and launches isolated review execution through runtime providers.
3. The agent performs review execution in isolation and reports results back through callback APIs.
4. The agent must remain database-independent. It must not read from or write to PostgreSQL directly.
5. Review execution boundaries matter: backend prepares work, worker launches work, agent executes work, backend persists results.
6. Shared packages define contracts and reusable logic, but should not blur the runtime boundary between backend and agent.
7. Frontend code should consume API contracts through generated types rather than handwritten response shapes.
8. Provider-specific logic should stay behind provider abstractions instead of leaking across services or routes.

For architecture details, read [docs/architecture-overview.md](docs/architecture-overview.md), [docs/worker.md](docs/worker.md), [docs/agent-executor.md](docs/agent-executor.md), and [docs/review-architecture.md](docs/review-architecture.md).

## Security Model

When reviewing code, changing authentication or authorization flows, or evaluating a possible vulnerability, keep the documented security model in mind.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CogitoForge-AI/cogito-review](https://github.com/CogitoForge-AI/cogito-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
