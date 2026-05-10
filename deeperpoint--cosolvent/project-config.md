---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

Monorepo with separate backend and frontend directories:

```
Cosolvent/
├── backend/                   # Python API + backend compiler
│   ├── app/                   # FastAPI application
│   ├── cli/                   # Backend CLI (compile, validate, wizard)
│   ├── alembic/               # DB migrations
│   ├── tests/                 # Backend tests
│   ├── scripts/               # Utility scripts
│   ├── Dockerfile
│   └── pyproject.toml
├── frontend/                  # Frontend compiler + generated Next.js app
│   ├── compiler/              # Python compiler (generates Next.js from OpenAPI + YAML)
│   │   ├── parsers/           # OpenAPI + marketplace.yaml parsers
│   │   ├── transforms/        # IR merge, page conventions, navigation
│   │   ├── generators/        # Code generators (types, schemas, hooks, routes, components)
│   │   ├── tests/
│   │   └── pyproject.toml
│   ├── Dockerfile
│   └── (generated Next.js files: package.json, src/, etc.)
├── docker-compose.yml         # Shared compose (orchestrates all services)
├── marketplace.yaml           # Shared config (read by both compilers)
├── openapi/                   # Shared OpenAPI spec (backend generates, frontend reads)
├── Makefile                   # Root orchestration
└── .env                       # Shared env vars
```

## Commands

```bash
# Install dependencies
make install                    # Backend: creates backend/.venv and installs deps
make install-frontend           # Frontend compiler: creates frontend/.venv and installs deps

# Linting & formatting (backend, Ruff + mypy)
make lint                       # Check for lint errors
make lint-fix                   # Auto-fix lint errors
make format                     # Format code
make type-check                 # Run mypy

# Testing
make unit                       # Backend unit tests
make unit-frontend              # Frontend compiler unit tests
make integration                # Integration tests (requires Docker stack)
make test-all                   # lint + unit + unit-frontend + integration + e2e

# Single backend test
cd backend && .venv/bin/python -m pytest tests/unit/path/to/test.py::ClassName::test_method -v

# Single frontend compiler test
cd frontend && .venv/bin/python -m pytest compiler/tests/test_compiler.py::ClassName::test_method -v

# Dev server (Docker, recommended)
make up                         # Start full stack: Postgres + Redis + API + worker
make setup-up                   # Start onboarding UI only (no DB required)
make down                       # Stop stack
make reset                      # Stop and remove volumes
make logs-api                   # Tail API logs

# Dev server (local, needs Postgres + Redis running externally)
make api                        # uvicorn app.main:app --reload
make worker                     # ARQ task worker

# CLI pipeline
make validate-config            # Validate marketplace.yaml
make compile                    # Generate backend artifacts from marketplace.yaml
make compile-check              # Verify generated artifacts match config
make generate-frontend          # Generate Next.js frontend from OpenAPI + marketplace.yaml
```

**URLs when running:**
- Onboarding UI: `http://localhost:18080/onboarding`
- API: `http://localhost:18000/api/`
- Swagger docs: `http://localhost:18000/docs`

## Architecture

### Overview
Config-driven marketplace builder. A single `marketplace.yaml` defines participant types, profile schemas, discovery rules, and onboarding workflows. Two compilers generate artifacts:
- **Backend compiler** (`backend/app/compiler/`): Generates Python runtime artifacts (role routers, enums, policy matrix)
- **Frontend compiler** (`frontend/compiler/`): Generates a complete Next.js app from OpenAPI spec + marketplace.yaml

### Backend Module Layout (`backend/app/modules/`)
Each module follows the same pattern: `router.py` (FastAPI routes) → `service.py` (business logic) → `repository.py` (DB access) → `schemas.py` (Pydantic models).

| Module | Responsibility |
|--------|---------------|
| `auth` | Signup/login/sessions via HttpOnly cookies |
| `profiles` | User profiles driven by marketplace.yaml field schemas |
| `discovery` | Vector search + keyword filtering for matching |
| `communication` | Conversations, messages, WebSocket real-time |
| `ai` | RAG queries, document management, LLM/embedding config |
| `files` | S3-based storage with presigned URL generation |
| `setup` | Onboarding wizard UI + config validation endpoints |
| `admin` | Admin-only management APIs |

### Database (`backend/app/core/`)
**Hybrid Postgres + pgvector** (no MongoDB despite document-style API):
- `db_schema.py`: All tables use `id` (UUID), `data` (JSONB), `created_at`, `updated_at`
- `database.py`: `DatabaseProxy` wraps SQLAlchemy to expose a Mongo-like interface
- `ai_document_chunks` and `profile_vectors` tables store 1536-dim pgvector embeddings

```python
# Mongo-style queries throughout the codebase
db = get_db()
await db.users.find_one({"email": "..."})
await db.profiles.insert_one({...})
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeeperPoint/Cosolvent](https://github.com/DeeperPoint/Cosolvent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
