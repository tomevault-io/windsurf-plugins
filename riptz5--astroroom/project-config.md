---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

AstroRoom is a provenance-aware, evidence-first astronomy workspace for Sagittarius A* analysis. FastAPI backend (26 routers, 39 services) + React 19/Vite frontend. The mounted UI follows a calm stage-based investigation flow: Session Entry, Astronomical Asset Gallery, Build Sets, then downstream placeholders for Process, Analyze, Evidence, and Export. Licensed PolyForm Noncommercial 1.0.0.

## Governing Taglines

> **"No construyan más amplitud. Construyan cierre, trazabilidad y convergencia."**

> **"Toda imagen importante debe poder explicarse hacia atrás, paso por paso, hasta sus FITS."**

Before writing any code, verify:
- [ ] Does this serve the evidence question for Sgr A* activity across epochs?
- [ ] Does it have a canonical entity or does it create a duplicate?
- [ ] What artifact does it produce to disk?
- [ ] What intermediate can be inspected for debugging?
- [ ] Does it follow the input → intermediate → output → provenance chain?
- [ ] Is it processing, visual, or evidence product?
- [ ] Is the claim language calibrated to evidence, not physics?
- [ ] Does the timeline drive this, or is it decorative?
- [ ] Does it read from session context, not maintain shadow state?
- [ ] Is this new, or does similar logic already exist that should be adapted?

## Audit Infrastructure

Use `backend/app/services/audit/` for all scientific computations:
- `AuditLogger`: timestamps, SHA-256 hashing, parameter logging
- `sha256_array()`, `sha256_file()`: integrity verification
- `log_feature_matrix_csv()`: export features to CSV for verification

Reference implementation: `flares_service.py` (FDR, permutation tests, bootstrap CI with audit).

## Commands

```bash
# Backend
pip install -r requirements.txt
uvicorn backend.main:app --reload --port 8000

# Frontend
cd frontend && bun install
VITE_API_BASE_URL=http://localhost:8000 bun run dev

# Docker (full stack with PostgreSQL)
docker compose up --build

# Docker (SQLite-only, lightweight)
docker compose -f docker-compose.yml -f docker-compose.sqlite.yml up --build

# Docker (with Fermi science toolchain — heavy)
docker compose --profile fermi up --build

# Backend tests
pytest tests_backend -q                        # all
pytest tests_backend/test_pipeline.py -q       # single file
pytest tests_backend/test_fits.py::test_name -q  # single test
pytest tests_backend --cov=backend/app --cov-report=term-missing  # with coverage

# Frontend lint & build
cd frontend && bun run lint && bun run build

# Playwright E2E (auto-launches backend+frontend)
cd tests && bunx playwright install chromium   # required first-time setup
cd tests && bun install && bunx playwright test
ASTROROOM_NO_WEBSERVER=1 bunx playwright test  # skip auto-launch

# Scientist CLI (autonomous 6-phase analysis pipeline)
python astro_scientist.py --help

# Database migrations
alembic upgrade head
alembic revision --autogenerate -m "description"
alembic current
alembic history
```

## Architecture

### Backend Layers

```
backend/main.py                    ← App entrypoint, registers all 26 routers
backend/app/config.py              ← Settings (pydantic-settings), storage paths
backend/app/core/database.py       ← SQLAlchemy engine, Base, SessionLocal, get_db()
backend/app/core/seed.py           ← Auto-seeds default CompatibilityProfile + 7 InstrumentProfiles
backend/app/models/db_models.py    ← 12+ SQLAlchemy models (the schema of record)
backend/app/models/*.py            ← Pydantic request/response models per domain
backend/app/api/*.py               ← 26 FastAPI routers (thin: validate → call service → return)
backend/app/services/*.py          ← 39 service modules (all business logic lives here)
backend/app/evidence/*.py          ← Unified evidence engines and schema exports
backend/app/scientist/             ← Autonomous analysis agent (see below)
backend/app/agents/                ← Fermi/GCE specialized agent components
```

Request flow: **Router** (api/) validates input via Pydantic → calls **Service** (services/) → reads/writes **DB Models** (models/db_models.py) via `get_db()` dependency → returns Pydantic response.

The pipeline router (`api/pipeline.py`) is the largest — it implements all 7 phases and uses **SSE streaming** (`StreamingResponse` with `text/event-stream`) for long-running operations like epoch frame building and full pipeline runs.

### Frontend Layers

```
App.tsx → SessionProvider            ← Sovereign session store
  → AstroWorkspace                   ← Active orchestration surface (no shadow session truth)
    → SessionEntryStage              ← Investigation entry, recent sessions, target selection
    → AssetGalleryStage              ← SessionAsset inventory, filters, inspector, working tray
    → BuildSetsStage                 ← WorkingSet suggestions, compatibility reasoning, warnings
    → WorkflowStagePlaceholder       ← Process / Analyze / Evidence / Export placeholders
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/riptz5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
