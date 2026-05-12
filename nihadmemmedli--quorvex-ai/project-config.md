---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## System Overview

**Natural Language to Test Script Converter** - AI-powered pipeline with dual-interface architecture (CLI + Web Dashboard) that converts plain English test specifications into production-ready Playwright tests.

**Key capability**: Write tests in markdown → Get validated, passing Playwright code automatically.

**Architecture**: Python backend (`orchestrator/`) + Next.js frontend (`web/`) + PostgreSQL/SQLite database + Playwright test runner + Memory system (vector embeddings + graph store)

## Essential Commands

```bash
# One-time setup (Python venv, Node deps, Playwright browsers, Database)
make setup

# Start web dashboard (Backend API on :8001 + Frontend on :3000)
make dev

# Run a specific test spec via CLI (uses Native Pipeline by default)
make run SPEC=specs/your-test.md

# Direct CLI execution with options
python orchestrator/cli.py specs/your-test.md              # Native Pipeline (default, recommended)
python orchestrator/cli.py specs/your-test.md --hybrid     # Hybrid healing (Native + Ralph escalation)
python orchestrator/cli.py specs/your-test.md --standard-pipeline  # Legacy standard pipeline

# Process PDF PRD to tests
python orchestrator/cli.py your-prd.pdf --prd
python orchestrator/cli.py your-prd.pdf --prd --feature "User Login"

# AI-powered app exploration
python orchestrator/workflows/app_explorer.py --url https://example.com --max-interactions 50

# Run generated Playwright tests
npx playwright test                                    # All tests
npx playwright test tests/generated/your-test.spec.ts # Specific test

# Skill mode (network interception, multi-tab, etc.) - see .claude/skills/playwright/SKILL.md
make setup-skills
python orchestrator/cli.py --run-skill /tmp/script.js
python orchestrator/cli.py specs/test.md --skill-mode

# Load testing with K6
make k6-workers-up                    # Start distributed K6 worker containers
make k6-workers-scale N=3             # Scale K6 workers
make k6-workers-down                  # Stop K6 workers
make k6-workers-status                # Check worker health

# Backup & Storage (production)
make backup             # Database-only backup
make backup-full        # Full backup (DB + specs + tests + PRDs + ChromaDB)
make restore TS=...     # Restore from timestamp
make storage-health     # Check DB, MinIO, local storage health
make archival           # Run artifact archival (30-day retention)

# Database migrations (PostgreSQL only)
make db-migrate M="description"   # Generate new Alembic migration
make db-upgrade                   # Run pending migrations
make db-downgrade                 # Roll back one migration

# Production maintenance
make upgrade            # Full upgrade (backup, pull, migrate, restart)
make health-check       # Hit all health endpoints
make docker-prune       # Remove dangling images and build cache

# Linting & formatting (Python: ruff, config in orchestrator/pyproject.toml)
make lint       # Ruff check (Python) + next lint (frontend)
make format     # Ruff format (Python)
make test       # Run all Python tests (cd orchestrator && pytest tests/ -v)

# Production development (Docker: backend :8001 + frontend :3000 + PostgreSQL)
make prod-dev   # Start prod with local code mounted (no rebuild needed)
make prod-restart # Restart backend (picks up code changes)
make prod-logs  # Tail production logs
make prod-status # Show status of all services

# Utilities
make clean      # Remove run artifacts
make check-env  # Validate configuration
make logs       # Tail backend and frontend logs
make stop       # Stop all running services
```

## Code Style

**Python** (enforced by ruff, config in `orchestrator/pyproject.toml`): Line length 120, double quotes, isort for imports. `B008` ignored (FastAPI `Depends` in defaults is fine). Target: Python 3.10.

**Frontend**: Next.js default linting via `next lint`.

## Pipeline Architecture

### Execution Modes

**CLI Mode** (`orchestrator/cli.py`): Direct execution, no database required, artifacts stored in `runs/TIMESTAMP/`

**Web Dashboard Mode** (`orchestrator/api/` + `web/`): FastAPI backend (port 8001), Next.js frontend (port 3000), PostgreSQL/SQLite for persistence

### Pipeline Types

1. **Native Pipeline** (default): Browser exploration at every stage → Most reliable
   - `NativePlanner` → `NativeGenerator` → `NativeHealer`
   - Optional: `--hybrid` for Native + Ralph healing escalation (up to 20 iterations)
2. **Standard Pipeline** (`--standard-pipeline`): Legacy text-only planning
   - `planner.py` → `plan_executor.py` → `exporter.py` → `validator.py`

**PRD Pipeline** (`--prd`): PDF → Feature extraction → Native spec generation → Test generation

**AI Exploration Pipeline**: Autonomous web app discovery using Playwright MCP tools. Discovers pages, user flows, API endpoints, form behaviors, error states. Stores findings for requirements generation and RTM creation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NihadMemmedli/quorvex_ai](https://github.com/NihadMemmedli/quorvex_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
