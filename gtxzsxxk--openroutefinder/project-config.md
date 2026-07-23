---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Important:** This file and `docs/claude/*.md` are living documentation. Any code change that affects architecture, APIs, data formats, SID/STAR behavior, or testing must also update the relevant documentation here. Do not leave docs out of sync with code.

## Project Overview

OpenRouteFinder is a flight route finder for flight simulation. It finds the shortest airway between two airports using Dijkstra/A* and is the only online service that allows users to select SID/STAR procedures.

- **Backend**: Python 3.10+, FastAPI, A* route engine
- **Frontend**: Vue 3 + TypeScript + Vite + Tailwind CSS + MapLibre GL JS + Pinia
- **Data format**: FlatBuffers (`.fb.zst`) for navigation data; multi-cycle support with hot reload
- **PWA**: Offline-capable, installable

## Development Commands

All commands run from the repository root unless noted.

### Setup
```bash
pip install -r requirements.txt
cd webFinder && npm install
```

### Development (both servers)
```bash
npm run dev              # concurrently starts frontend (:5173) and backend (:9807)
npm run dev:frontend     # Vite dev server only
cd openRouterFinder && uvicorn api:app --reload --port 9807   # backend only
```

### Build & Production
```bash
npm run build            # build frontend (runs vue-tsc + vite build)
cd openRouterFinder && uvicorn api:app --host 0.0.0.0 --port 9807   # production
```

### Testing

Two-layer suite: `tests/unit/` (direct object/function tests) and `tests/e2e/`
(boots a real uvicorn server and drives it over HTTP). Config is in
`pyproject.toml`; the e2e layer has `tests/e2e/conftest.py`.

```bash
PYTHONPATH=. DISABLE_CAPTCHA=true pytest      # all tests
pytest tests/unit -v                          # unit layer only
pytest tests/e2e -v                           # e2e layer (boots a server)
pytest tests/unit/test_dijkstra.py -v         # single file
pytest tests/unit/test_airport_unit.py::test_build_sid_with_filter -v   # single test
```

### Linting
```bash
ruff check               # lint Python backend
ruff check --fix         # auto-fix Python issues
npm run lint             # lint frontend (cd webFinder && npm run lint)
```

Pre-commit hooks are configured. Run before committing:
```bash
pre-commit run --all-files
```

## Documentation Index

Detailed documentation lives in `docs/claude/`. Read the relevant file before modifying that area:

| Document | Covers | Read When Modifying... |
|----------|--------|----------------------|
| [Backend Architecture](docs/claude/backend.md) | FastAPI endpoints, A* engine, data structures, storage subsystem, utilities | Any Python backend code |
| [SID/STAR Processing](docs/claude/sid-star.md) | `FlatbuffersAirportConnector`, procedure parsing, transition splitting, approach bridges, synthetic marker filtering | `core/airport.py`, any procedure-related logic |
| [Frontend Architecture](docs/claude/frontend.md) | Vue 3 components, composables, Pinia state, MapLibre integration, i18n | Any frontend code |
| [Testing](docs/claude/testing.md) | Test files, running tests, procedure integrity checks, integration tests | Adding or modifying tests |
| [Data Formats](docs/claude/data-formats.md) | Three navdata backends (legacy pickle, FlatBuffers, Fenix), schema, preprocessing | Navdata loading, storage, conversion |
| [API Endpoints](docs/claude/api-endpoints.md) | All REST endpoints, request/response shapes | `api.py`, frontend API calls |

## Architecture

### Backend (`openRouterFinder/`)

**Core data flow**: nav data is stored as zstd-compressed FlatBuffers files in `data/`. At startup, `api.py` loads all `navdata_*.fb.zst` files via `NavDataRegistry`, builds an airport prefix index, and starts a METAR updater thread.

Key modules:

- `api.py` — FastAPI app with all endpoints (`/api/route`, `/api/airports`, `/api/airports/{icao}/procedures`, `/api/admin/*`, etc.). Route calculation runs in `_dijkstra_pool` (4 workers) guarded by an `asyncio.Semaphore(4)`. Fenix builds run serially in `_build_pool` (1 worker). Admin endpoints share a `verify_admin_key` dependency. The airport prefix index is rebuilt atomically after a navdata cycle is uploaded or deleted.
- `core/dijkstra.py` — `RouteEngine` uses a **hybrid A\* search**: it first tries a mixed-graph A\* over airway + SID/STAR nodes, then falls back to a phase-separated search when constraints cannot be satisfied. Uses admissible great-circle heuristic, precomputed `edge.dist`, candidate pruning (top 50), and cycle prevention.
- `core/graph.py` — Immutable `Node`/`Edge` dataclasses and great-circle distance utilities. `Edge` carries a precomputed `dist` field.
- `core/airport.py` — `FlatbuffersAirportConnector` builds temporary nodes and edges for SID/STAR procedures from FlatBuffers navdata. Also contains the legacy `AirportConnector` for pickle-based data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gtxzsxxk/openRouteFinder](https://github.com/gtxzsxxk/openRouteFinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
