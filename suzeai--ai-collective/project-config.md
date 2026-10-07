---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

AI – Collective: a platform for creating and managing AI-powered companies — virtual organizations of any type (software, marketing, research, or general) built from Departments staffed by AI, each run as a LangGraph-based multi-agent topology. React 18 + TypeScript frontend (`/ui`) talking to a FastAPI backend (`/server`). Source-available, non-commercial license (see `LICENSE`/`NOTICE`).

**Note:** `README.md` describes an earlier state of the project (pre terminology-rename: mentions `server/domain/agent/`, `AgentBuilder.tsx`/`TeamBuilder.tsx`/`Workspaces.tsx`, only 4 topologies). Trust this file and the codebase over `README.md` for current names and structure.

## Commands

### Setup
```bash
make install          # uv sync --all-extras + npm --prefix ui ci
cp .env.template .env # then fill in at least one provider key (e.g. GOOGLE_API_KEY) referenced by config.yml's models: list
```

### Run locally (no Docker)
```bash
make backend                        # uvicorn --reload on :8000 (needs `make infra` for redis/rabbitmq if used)
make frontend                       # vite dev server on :8080
make infra / make infra-down        # redis + rabbitmq only, for local backend dev
```
Code defaults need no infra: `storage.backend=json`, `task_queue.backend=memory`, `lock.backend=threading` — but the committed `config.yml` may already point at `mongo`/`rabbitmq`/`redis`; check it before assuming which mode is active.

### Run with Docker
```bash
make dev            # full dev stack, hot-reload, all services -> http://localhost:2026
make dev-down       # stop + remove (incl. profile services)
make dev-logs / make dev-logs-backend / make dev-logs-frontend
make up / make down # production stack
```
Optional compose profiles via `PROFILES=` (e.g. `make dev PROFILES=router`): `sandbox`, `provisioner`, `router`, `mongo-express`, `tools`, `minio`.

### Tests
```bash
# Backend — tests live in test/ at repo root, NOT server/ (server/ has no test files despite the Makefile target name)
# test/api/ — endpoint tests via the shared `client` TestClient fixture; test/unit/ — everything else;
# test/manual/ — ad-hoc debug scripts, not collected by pytest.
PYTHONPATH=. uv run pytest test/ -v
PYTHONPATH=. uv run pytest test/unit/test_foo.py -v            # single file
PYTHONPATH=. uv run pytest test/api/test_foo.py -k name -vv    # single test, verbose

# Frontend (from ui/, or npm --prefix ui ...)
cd ui && npm run test          # vitest run (jsdom, tests under src/**/*.{test,spec}.{ts,tsx})
cd ui && npm run test:watch
cd ui && npx vitest run src/path/to/file.test.ts   # single file
```
`test/conftest.py` provides a session-scoped `client` (FastAPI `TestClient`) plus `admin_headers`/`user_headers` fixtures and a `unique()` helper for collision-free test names; it wipes the test storage dir once per session. CI (`.github/workflows/ci.yml`) runs `pytest test/ -v` and (in `ui/`) `npm run test` + `npm run build` on every push to `main`; frontend lint is non-blocking there.

### Lint / build
```bash
make lint            # ruff (backend) + eslint (frontend)
cd ui && npm run build         # vite build
```

## Architecture

### Backend: ports-and-adapters (hexagonal), dependencies point inward

```
server/
├── api/            # FastAPI app, routers, Pydantic schemas, DI (deps/), auth, settings
├── app/            # Use-case services; ports/ holds Protocols (repositories, llm, agent_graph)
├── domain/         # Framework-free business logic + models.py (frozen dataclasses)
├── infra/          # Adapters implementing app.ports (repos, LLM providers, queues, locks, sandbox)
└── share/          # Logging setup (log.py) + shared text helpers
```
- `domain` imports nothing from `app`/`api`/`infra` — with one accepted pragmatic exception: `server/domain/tools/*.py`, `server/domain/staff/langgraph_*.py` + `_graph_runtime.py`, and `server/domain/memory/working_memory.py` import `server.api.settings.settings` (and a couple of concrete `server.infra.*` adapters) directly, since threading these through `app.ports` would mean re-deriving config plumbing for ~20 files with no behavioral payoff. Don't treat this as a bug to fix opportunistically; new domain code should still prefer not adding fresh direct-to-infra imports.
- `app` depends only on `app.ports.*` Protocols, never concrete adapters — except a handful of lazy, function-local imports used to dodge circular imports: `graph_context_service.py` (`rag_retrieval_store`), `document_library_service.py` (`file_store`, `sandbox_session`, `thread_files`, `sandbox_middleware`), `long_term_memory_service.py` (`working_memory_store`), `staff_graph_service.py` (`sandbox_middleware`). Same rule: accepted as-is, not a template for new code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuZeAI/ai-collective](https://github.com/SuZeAI/ai-collective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
