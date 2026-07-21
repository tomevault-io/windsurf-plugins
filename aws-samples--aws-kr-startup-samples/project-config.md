---
trigger: always_on
description: Project context document for AI coding agents.
---

# AGENTS.md

Project context document for AI coding agents.

## Project Overview

**Claude Code Proxy** is an enterprise LLM gateway service that manages Claude Code access with automatic failover, usage tracking, and multi-tenant access control. It acts as a centralized proxy between Claude Code clients and AI model providers (Anthropic Plan API and Amazon Bedrock).

### Core Features
- **Dual-provider routing**: Primary Anthropic Plan API with automatic Bedrock fallback on rate limits
- **Multi-tenant access**: Unique access keys per user with per-key budget limits
- **Cost visibility**: Real-time token usage tracking with pricing snapshots and detailed breakdowns
- **Admin dashboard**: Web UI for user management, key provisioning, Bedrock credential registration, and analytics
- **Enterprise security**: Access keys stored as hashed values, Bedrock credentials encrypted with KMS

## Detailed Documentation

For detailed project information, refer to the documents in `.kiro/steering/`:

| File | Contents |
|------|----------|
| `.kiro/steering/product.md` | Product overview, business context, API contract, data model |
| `.kiro/steering/structure.md` | Project structure, architecture patterns, data flow |
| `.kiro/steering/tech.md` | Tech stack, commands, environment variables, debugging guide |

## Quick Reference

### Local Bootstrap

```bash
# Full stack (docker compose)
docker compose up --build

# DB only (when running backend/frontend locally)
docker compose up db
```

### Build & Run

```bash
# Backend
cd backend
pip install -e ".[dev]"
alembic upgrade head
uvicorn src.main:app --reload --port 8000

# Frontend
cd frontend
npm ci
npm run dev

# Infrastructure
cd infra
pip install -r requirements.txt
cdk synth
```

### Environment Variables

Backend (e.g., `.env` or runtime environment):
- Copy `backend/.env.example` to `backend/.env` and update values.
- `PROXY_DATABASE_URL` e.g., `postgresql+asyncpg://postgres:postgres@localhost:5432/proxy`
- `PROXY_PLAN_API_KEY` (optional)
- `PROXY_KEY_HASHER_SECRET`
- `PROXY_ADMIN_USERNAME`
- `PROXY_ADMIN_PASSWORD_HASH`
- `PROXY_CLOUDWATCH_METRICS_ENABLED` (optional, default `true`): env on/off for CloudWatch metrics. `false` = no PutMetricData.
- `PROXY_CLOUDWATCH_NAMESPACE` (optional, default `ClaudeCodeProxy`): CloudWatch namespace for `proxy.*` metrics. If changed, update ECS task IAM `cloudwatch:namespace` in `infra/stacks/compute_stack.py`.
- OTEL (optional, default off): `PROXY_OTEL_METRICS_ENABLED`, `PROXY_OTEL_ENDPOINT` (e.g. `http://localhost:4317`), `PROXY_OTEL_SERVICE_NAME`, `PROXY_OTEL_INSECURE`, `PROXY_OTEL_EXPORT_INTERVAL_MS`, `PROXY_OTEL_EXPORT_TIMEOUT_MS`. `PROXY_OTEL_USER_METRICS_ENABLED` enables per-user `proxy.user.*` (OTEL only). See `.kiro/steering/tech.md`.

Frontend:
- Copy `frontend/.env.example` to `frontend/.env.local` (or `frontend/.env`) and update values.
- `VITE_BACKEND_API_URL` e.g., `http://localhost:8000`

Note: See `docker-compose.yml` for default values/examples

### Infrastructure Prerequisites

- AWS credentials/region configuration required, see `.kiro/steering/tech.md` for details

### Testing

```bash
# Backend tests
cd backend
pytest

# Frontend type check
cd frontend
npx tsc --noEmit
```

### Linting & Formatting

```bash
# Backend
cd backend
ruff check .
ruff format .
mypy src

# Frontend
cd frontend
npm run lint
```

## Code Style

### Backend (Python)
- Python 3.11+, type hints required
- Use async/await pattern (all DB/HTTP operations)
- Environment variables use `PROXY_` prefix
- Repository pattern for DB access separation

### Frontend (TypeScript)
- React 18 + Vite
- Path alias: `@/` → `src/`
- Tailwind CSS
- No Node version pinning file (`.nvmrc`/`.node-version`)

## Main Endpoint

```
POST /ak/{access_key}/v1/messages  # Main proxy endpoint
```

## Database

- PostgreSQL 15 + asyncpg
- Migrations: Alembic
- Soft delete pattern (`deleted_at` column)

```bash
# Create migration
alembic revision --autogenerate -m "description"

# Apply migration
alembic upgrade head
```

## Key File Locations

| Area | File |
|------|------|
| FastAPI app | `backend/src/main.py` |
| Proxy router | `backend/src/api/proxy_router.py` |
| Routing logic | `backend/src/proxy/router.py` |
| DB models | `backend/src/db/models.py` |
| Config | `backend/src/config.py` |
| Logging | `backend/src/logging.py` |
| API client | `frontend/src/lib/api.ts` |
| CDK app | `infra/app.py` |

## Architecture Overview

### Routing Strategies

Users can configure one of two routing strategies:

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| `plan_first` (default) | Try Anthropic Plan API first, fallback to Bedrock on rate limit/failure | Organizations with existing Plan subscription |
| `bedrock_only` | Skip Plan API entirely, use only Bedrock | Pure pay-per-use model |

Configure via: Admin dashboard or `PUT /admin/users/{user_id}/routing-strategy`

### Request Flow Architecture

```
POST /ak/{access_key}/v1/messages
    ├─ AuthService (validate & cache access key)
    ├─ BudgetService (KST-based monthly budget check)
    ├─ ProxyRouter (circuit breaker + routing strategy)
    │   ├─ PlanAdapter (primary: Anthropic Plan API)
    │   │   ├─ stream response to client
    │   │   └─ UsageRecorder (track tokens & calculate cost)
    │   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/aws-kr-startup-samples](https://github.com/aws-samples/aws-kr-startup-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
