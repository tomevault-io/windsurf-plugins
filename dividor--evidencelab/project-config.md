---
trigger: always_on
description: AI-powered document analysis and search platform. Ingests PDFs/Word docs through a processing pipeline (parse, chunk, summarize, tag, index) and exposes hybrid semantic search, a research assistant, and AI protocol integrations (MCP, A2A).
---

# Evidence Lab

AI-powered document analysis and search platform. Ingests PDFs/Word docs through a processing pipeline (parse, chunk, summarize, tag, index) and exposes hybrid semantic search, a research assistant, and AI protocol integrations (MCP, A2A).

## Architecture

Monorepo with five subsystems sharing `config.json` and databases:

```
pipeline/          # Document processing (Celery workers, Docling parsing, LLM tagging)
ui/backend/        # FastAPI REST API (port 8000) — search, assistant, auth, admin
ui/frontend/       # React 18 + TypeScript SPA (port 3000)
mcp_server/        # Model Context Protocol server (port 8001) — Claude/ChatGPT integration
a2a_server/        # Agent-to-Agent protocol server (port 8001) — Google ADK, CrewAI
```

**Stack:** Python 3.11, FastAPI, Celery + Redis, SQLAlchemy 2.0, Alembic, Qdrant (vector DB), PostgreSQL 16 (pgvector), React 18, TypeScript, LangChain/LangGraph.

**Key config files:**
- `config.json` — central source of truth for datasources, models, pipeline settings, field mappings
- `.env` / `.env.example` — API keys, DB connections, feature flags, auth mode
- `docker-compose.yml` — full local dev stack (qdrant, postgres, redis, embedding-server, api, pipeline, mcp, ui)

### Production Deployment

Traffic flow: **Caddy** (HTTPS/Let's Encrypt, ports 80/443) → **nginx** (static React assets + internal proxy) → **FastAPI** (port 8000, localhost-only).

```
Internet → Caddy (:443) → nginx in ui (:80) ─┬─ static assets (1yr cache, gzip)
                                               ├─ /api/*    → api:8000 (strips /api)
                                               ├─ /mcp/*    → mcp:8001 (300s timeout, no buffering)
                                               ├─ /a2a/*    → mcp:8001 (300s timeout, no buffering)
                                               └─ /* (SPA)  → index.html
```

Key files: `docker-compose.prod.yml`, `Caddyfile`, `ui/frontend/Dockerfile.prod`, `ui/frontend/nginx.conf`.

Notes:
- Frontend is a multi-stage build: `node:18-alpine` builds React → `nginx:alpine` serves static files. `REACT_APP_*` env vars are **baked in at build time** — Docker rebuild required to change.
- Qdrant runs on the host via systemd (not in Docker) — containers connect via `host.docker.internal:6333`.
- Pipeline service is commented out in prod compose by default.
- API binds to `127.0.0.1:8000` — never publicly exposed.

## Commands

```bash
# Run locally (full stack)
docker compose up -d --build

# Production
docker compose -f docker-compose.prod.yml up -d --build

# Unit tests
pytest tests/unit/ -v
docker compose exec -T pipeline pytest tests/unit/ -v

# Frontend tests
cd ui/frontend && npm test -- --watchAll=false
docker compose exec -e CI=true ui npm test -- --watchAll=false

# Integration tests (requires full Docker stack)
docker compose exec -T pipeline pytest tests/integration -v -s

# Lint / format (all hooks)
pre-commit run --all-files

# Individual linters
black .
isort --profile black .
flake8 --max-line-length=100 --extend-ignore=E203,W503 .
mypy --ignore-missing-imports .  # scripts/ and alembic/ excluded (see .pre-commit-config.yaml)

# Code complexity check
python scripts/quality/code_metrics.py --fail-on-bad --skip-js-cognitive

# Database migrations
docker compose exec -T pipeline alembic upgrade head

# Security scans
bandit -r pipeline/ ui/backend/ -lll --exclude tests,node_modules,.venv
pip-audit -r requirements.txt --desc on
cd ui/frontend && npm audit --audit-level=high

# Run pipeline on host (Mac/Linux, no Docker)
./scripts/pipeline/run_pipeline_host.sh [orchestrator-args]
# Requires: LibreOffice, Qdrant + Postgres running, .env with API keys
# Default: --data-source uneg --workers 7 --skip-download --skip-scan --recent-first

# Reset failed pipeline documents for reprocessing
python scripts/pipeline/reset_failed_statuses.py --data-source <name> [--dry-run]

# Database backup & restore
python scripts/sync/db/dump_postgres.py --data-source <name> [--prod]
python scripts/sync/db/dump_qdrant.py --data-source <name>
python scripts/sync/db/restore_postgres.py --source <path> [--clean]
python scripts/sync/db/restore_qdrant.py --source <path>

# Upload backups to remote (Azure, GCP, SCP)
python scripts/sync/db/sync_backup_to_remote.py \
  --source-qdrant <path> --source-postgres <path> --mode azure_storage
```

## Project Rules

### Git Commits
- **NEVER use `--no-verify` when committing.** If a pre-commit hook fails, ALWAYS fix the underlying issue (lint errors, formatting, complexity, etc.) before committing again. No exceptions.
- **NEVER commit directly to `main`, `rc/v*`, or any release branch.** All changes go on a feature branch and in via PR. No exceptions.
- Use Conventional Commits format: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`, `perf:`, `ci:`, `build:`.

### Documentation
- **All docs MUST go in `docs/` at the repo root.** The directory `ui/frontend/public/docs/` is wiped and regenerated from `docs/` at every build by `copy-docs.js`. Anything written there will be lost on the next build.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dividor/evidencelab](https://github.com/dividor/evidencelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
