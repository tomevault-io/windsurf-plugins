---
trigger: always_on
description: OpenDevOps Agent is an open-source AWS incident investigation tool powered by any LLM via LiteLLM. It runs a LangGraph ReAct loop (via DeepAgents) that calls 27 tools — 21 structured boto3 AWS tools plus bash, history analytics, skills, and a structured final-answer tool — then streams results to a React/Vite chat UI over SSE. Auth, multi-user RBAC, event-driven incident detection (EventBridge → SQS), and proactive anomaly polling are all built in and optional.
---

# CLAUDE.md — OpenDevOps Agent

## Project Overview

OpenDevOps Agent is an open-source AWS incident investigation tool powered by any LLM via LiteLLM. It runs a LangGraph ReAct loop (via DeepAgents) that calls 27 tools — 21 structured boto3 AWS tools plus bash, history analytics, skills, and a structured final-answer tool — then streams results to a React/Vite chat UI over SSE. Auth, multi-user RBAC, event-driven incident detection (EventBridge → SQS), and proactive anomaly polling are all built in and optional.

---

## Repo Structure

```
apps/
  core/           Installable package `opendevops-core` — the shared agent brain:
                  src/opendevops_core/{agent, tools, providers, models, skills,
                  integrations, migrations, config.py}. Has no web/CLI layer.
                  Consumed by the OSS backend (and, later, the SaaS product repo).
  backend/        OSS web app + CLI — src/{api, cli, config, mcp_server.py}, tests/,
                  scripts/, pyproject.toml. Depends on opendevops-core via a uv path source.
  frontend/       React/Vite UI — src/, package.json, vite.config.ts
  documentation/  Markdown docs (future hosted docs site)
deployment/
  docker-compose/ docker-compose.yml (PostgreSQL + backend + frontend)
  railway/        Dockerfile.railway + railway.toml (combined single-image deploy)
design-system/   Cross-cutting design reference (colors, typography, UI kits)
demos/           Reproducible AWS incident scripts for local testing
Makefile         Root convenience targets — wraps `cd apps/backend && uv run ...`
```

All Python commands run from `apps/backend/` (or via `make <target>` at repo root).
`uv sync` from `apps/backend/` installs `opendevops-core` editable from `../core`, so edits
to core are live without republishing.

### Core vs app boundary
- **Reusable agent logic lives in `apps/core` (`opendevops_core`)** — the DeepAgents loop,
  tools, providers, models, skills, integrations, DB backends, and baseline migrations.
- **Web/CLI-only code stays in `apps/backend`** — FastAPI routers, auth, the CLI, `mcp_server.py`.
- **Config injection:** core reads settings through the `settings` proxy in
  `opendevops_core/config.py`, which delegates to whatever instance the host app registers via
  `configure()`. The OSS app's `Settings(CoreSettings)` (in `apps/backend/src/config/appsettings.py`)
  adds web/auth-only fields (e.g. `jwt_*`) and calls `configure(settings)` at startup.

---

## Automatic Behavior Rules

**Always do these when making changes:**

- **New env var:** if core reads it, add the Pydantic field to `CoreSettings` in `apps/core/src/opendevops_core/config.py`; if it's web/auth-only, add it to `Settings(CoreSettings)` in `apps/backend/src/config/appsettings.py`. Either way mirror it in `.env.example` (with a comment). Never read env vars directly — always go through `settings`.
- **New DB column or table:** add a new numbered migration. Core-domain schema (tables core code reads/writes) goes in `apps/core/src/opendevops_core/migrations/` (e.g. `014_name.sql`); OSS-app-only schema goes in `apps/backend/migrations/`. The runner applies core-then-app, tracked in the `schema_migrations(source, version)` ledger. Never add columns inline in Python code.
- **New tool:** add it to `ALL_TOOLS` in `apps/core/src/opendevops_core/agent/core.py`. Tool functions must be plain synchronous Python functions — DeepAgents infers the JSON schema from type hints and docstrings.
- **New API route that matches a React Router path:** prefix it with `/api/` to avoid the SPA fallback conflict. The `/{full_path:path}` catch-all in `apps/backend/src/api/app.py` intercepts any GET that matches a registered FastAPI route first.
- **New skill:** drop a `SKILL.md` file into `apps/core/src/opendevops_core/skills/<name>/SKILL.md`. It is picked up automatically at startup (and bundled into the core wheel) — no code changes needed. Use the frontmatter format (`name`, `description`) from the existing `lambda-throttling` skill.
- **Docs sync:** if a feature has a corresponding file in `apps/documentation/`, update it when the feature changes. The `apps/documentation/` folder is the public documentation source.

---

## Common Commands

```bash
# Install / update dependencies
cd apps/backend && uv sync          # or: make install

# Development server — FastAPI with hot reload
cd apps/backend && uv run dev       # or: make dev

# Production web UI (FastAPI backend + serves built frontend, no reload)
cd apps/backend && uv run devops-agent ui   # or: make ui

# Apply SQL migrations to PostgreSQL (requires CHECKPOINT_BACKEND=postgres + DATABASE_URL)
cd apps/backend && uv run migrate   # or: make migrate

# CLI investigation
cd apps/backend && uv run devops-agent investigate "Lambda high error rate on payment service"
cd apps/backend && uv run devops-agent ask "Why would an ECS task OOM?"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AhmadHammad21/OpenDevOps](https://github.com/AhmadHammad21/OpenDevOps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
