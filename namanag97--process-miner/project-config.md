---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A full-stack **Process Mining SaaS Platform** built with:
- **Backend**: FastAPI (Python 3.10+) with PM4Py for process mining algorithms
- **Frontend**: React 19 with Rspack bundling
- **Database**: SQLite (async via aiosqlite) with Alembic migrations
- **Data Processing**: DuckDB for high-performance data ingestion and analytics
- **Storage**: S3/MinIO for event log files (local filesystem in dev)
- **Workflow Orchestration**: Temporal for async long-running operations
- **Authentication**: JWT with access/refresh tokens

## Essential Commands

### Backend Development

```bash
# Start backend dev server (auto-reload on port 8001)
cd backend/src && ../.venv/bin/python -m uvicorn api.main:app --reload --port 8001

# Alternative using Makefile
cd backend && make dev

# Run all quality checks (lint + typecheck + security)
make check

# Run tests
make test
cd backend && .venv/bin/python -m pytest tests/ -v

# Run single test file
cd backend && .venv/bin/python -m pytest tests/api/test_datasets.py -v

# Database migrations
cd backend && .venv/bin/alembic upgrade head        # Apply migrations
cd backend && .venv/bin/alembic revision --autogenerate -m "description"  # Create migration
cd backend && .venv/bin/alembic current             # Check current version
cd backend && .venv/bin/alembic history             # View migration history
```

### Frontend Development

```bash
# Start frontend dev server (port 4200)
cd frontend-new && npm run start

# Build for production
cd frontend-new && npm run build

# Run tests
cd frontend-new && npm run test

# Regenerate API client from OpenAPI spec
cd frontend-new && npm run generate:sdk
```

### Full Stack Development

```bash
# Install all dependencies
make install

# Run full CI suite (lint + typecheck + security + tests)
make all

# Run both backend and frontend
make dev-full
```

## Architecture

### Backend: Layered Architecture with Feature Modules

The backend follows a **layered architecture** with strict dependency rules enforced by `import-linter`:

```
┌─────────────────────────────────────────────────────┐
│  API Layer (src/api/)                               │
│  - FastAPI app factory, middleware, exception handlers│
│  - Imports routers from features and infra          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Features Layer (src/features/)                     │
│  - Process mining algorithms and business logic     │
│  - Each feature: router.py, service.py, schemas.py  │
│  - Depends on: infra, shared                        │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Infrastructure Layer (src/infra/)                  │
│  - Platform services: auth, organizations, projects │
│  - Core: config, database, logging, middleware      │
│  - Temporal workflows, jobs, health checks          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Shared Layer (src/shared/)                         │
│  - Utilities, types, constants                      │
│  - No dependencies on other layers                  │
└─────────────────────────────────────────────────────┘
```

**Key Principles:**
- **Features Independence**: Feature modules cannot depend on API layer
- **Infra Independence**: Infrastructure should not import from features
- **Shared Isolation**: Shared layer has no internal dependencies

### Multi-Tenant Authorization Hierarchy

```
Organization (owner)
  └── Workspace (owner/admin/editor/viewer)
        └── Project (inherits workspace permissions)
              └── Dataset (inherits project permissions)
```

### Dataset Lifecycle

```
PENDING → UPLOADING → UPLOADED → MAPPED → INGESTING → READY
                                                 ↓
                                            FAILED/ERROR
```

### Process Mining Capabilities

- **Discovery**: Alpha, Inductive, Heuristics, Split miners
- **Conformance**: Token replay, alignments, footprint comparison
- **Analytics**: Bottlenecks, cycle times, throughput, rework detection
- **Visualization**: DFG (Directly-Follows Graph), Petri nets
- **Predictions**: ML-based next activity and remaining time
- **Organizational**: Social network analysis, resource handovers
- **Simulation**: What-if analysis
- **OCPM**: Object-Centric Process Mining (OCEL 2.0)

### Backend Code Organization

```
backend/src/
├── api/                      # FastAPI application entry point
│   ├── main.py              # App factory, middleware, exception handlers
│   └── routers/             # Router imports and registration
├── application/              # CQRS application layer
│   ├── commands/            # Command handlers
│   ├── queries/             # Query handlers
│   └── projections/         # Read model projections
├── features/                 # Process mining feature modules
│   └── process_mining/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/namanag97) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
