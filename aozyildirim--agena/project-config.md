---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Package Structure

Backend is split into **6 pip-installable packages** + **1 npm package** under `packages/`:

```
packages/
  core/        → agena-core      (settings, database, auth, rbac, security)
  models/      → agena-models    (27 SQLAlchemy ORM models + 9 Pydantic schemas)
  services/    → agena-services  (31 business logic services + integrations)
  agents/      → agena-agents    (CrewAI/LangGraph pipeline + vector memory)
  api/         → agena-api       (FastAPI routes, middleware, dependencies)
  worker/      → agena-worker    (Redis background task consumer)
  sdk/         → @agena/sdk      (TypeScript API client for npm)
```

**Import paths** use package prefixes:
- `from agena_core.settings import get_settings`
- `from agena_core.database import get_db_session`
- `from agena_core.security.jwt import create_token`
- `from agena_models.models.task_record import TaskRecord`
- `from agena_models.schemas.agent import AgentRunRequest`
- `from agena_services.services.orchestration_service import OrchestrationService`
- `from agena_services.integrations.azure_client import AzureClient`
- `from agena_agents.agents.crewai_agents import CrewAIAgentRunner`
- `from agena_agents.memory.qdrant import QdrantMemoryStore`
- `from agena_api.api.dependencies import get_current_tenant`
- `from agena_worker.workers.redis_worker import process_queue`

**NEVER** use old flat imports like `from core.settings` or `from models.user`.

## Development Environment

Use `start.sh` to launch the full stack. It starts Docker services and the CLI bridge on the host:

```bash
./start.sh                         # Start everything (Docker + host bridge)
./stop.sh                          # Stop everything
docker-compose restart <service>   # Restart after code changes
docker-compose up -d --build <service>  # Rebuild a specific service
```

Services: `backend` (FastAPI, :8010), `worker` (Redis consumer), `frontend_blue` (:3011), `frontend_green` (:3012), `mysql` (:3307), `redis` (:6380), `qdrant` (:6333)

### CLI Bridge (Host-Native)

The CLI bridge runs **on the host** (not in Docker) so it can use the host's Claude/Codex CLI authentication (macOS Keychain). Docker containers reach it via `http://host.docker.internal:9876`.

```bash
# start.sh handles this automatically. Manual start:
node docker/bridge-server.mjs &    # Start bridge on host (:9876)
curl localhost:9876/health          # Check status
```

- Bridge authenticates via the host's `claude auth login` session — no API key needed
- Bridge is **host-only** now — not defined in docker-compose. Started by `./start.sh`.

Backend code is volume-mounted (`./packages:/app/packages`) — hot-reloads via `pip install -e` on container start. Worker needs manual restart.

### Database Migrations

```bash
docker-compose exec backend alembic revision -m "description"  # Create migration
docker-compose exec backend alembic upgrade head               # Apply migrations
```

Alembic imports use `agena_core` and `agena_models`:
- `from agena_core.settings import get_settings`
- `import agena_models.models  # registers all ORM models`

### Frontend Build/Lint

```bash
docker-compose exec frontend_blue npm run build
docker-compose exec frontend_blue npm run lint
```

If stale webpack errors occur: `docker exec ai_agent_frontend_blue rm -rf /app/.next` then restart.

### Frontend Production Deploy (Zero-Downtime)

Frontend runs as **blue/green** production containers. Nginx load-balances between them. Code is NOT volume-mounted — changes require a rebuild.

```bash
./scripts/deploy-frontend.sh       # Zero-downtime (rebuilds one at a time)
# NEVER use docker-compose up --build for both at once — causes 502
```

### Local Development (without Docker)

```bash
pip install -e packages/core -e packages/models -e packages/services \
    -e packages/agents -e packages/api -e packages/worker
uvicorn agena_api.api.main:app --reload --port 8010
python -m agena_worker.workers.redis_worker
```

## Architecture

**Multi-tenant AI agent SaaS** — orchestrates LLM-powered code generation and creates GitHub/Azure DevOps PRs.

### Request Flow

```
API Request → FastAPI (agena_api) → Redis Queue → Worker (agena_worker)
  → OrchestrationService (agena_services)
  → LangGraph pipeline (5 nodes: fetch_context → analyze → generate_code → review_code → finalize)
  → CrewAI agents (PM, Developer, Reviewer, Finalizer)
  → PR creation (GitHub/Azure DevOps)
```

### Key Entry Points

- **Backend app**: `packages/api/src/agena_api/api/main.py`
- **Worker**: `packages/worker/src/agena_worker/workers/redis_worker.py`
- **Core orchestration**: `packages/services/src/agena_services/services/orchestration_service.py`
- **LLM routing**: `packages/services/src/agena_services/services/llm/provider.py`
- **Agent pipeline**: `packages/agents/src/agena_agents/agents/langgraph_flow.py` + `crewai_agents.py`
- **Flow executor**: `packages/services/src/agena_services/services/flow_executor.py`
- **Prompt service**: `packages/services/src/agena_services/services/prompt_service.py` (DB-backed, cached)

### Package Dependencies

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aozyildirim/Agena](https://github.com/aozyildirim/Agena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
