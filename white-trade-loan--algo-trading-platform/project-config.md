---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

OpenAlgo is a production-ready algorithmic trading platform built with Flask (backend) and React 19 (frontend). It is **four products in one self-hosted instance**, all sharing a single broker session and WebSocket feed:

| Surface | Route | Purpose |
| --- | --- | --- |
| **Unified Broker API** | `/api/v1/` | External platforms (TradingView, Amibroker, ChartInk, Excel, Python, MCP) |
| **Python Strategy Host** | `/python` | In-browser CodeMirror editor — paste scripts, schedule on IST times, run parallel strategies with process isolation and live logs |
| **Flow (No-Code Builder)** | `/flow` | Drag-and-drop nodes: market data → indicators → conditions → order execution; JSON import/export |
| **Options Trading Suite** | `/tools` | 12 analytical tools: Strategy Builder, Option Chain, IV Smile, Max Pain, Vol Surface, GEX, OI Tracker, Straddle Chart, etc. |

All surfaces share the Sandbox engine (₹1 Crore sandbox capital, exchange-aligned auto square-off) and support Telegram alerts.

**Repository**: https://github.com/marketcalls/openalgo
**Documentation**: https://docs.openalgo.in

## Security and Deployment Model

- **Single user per deployment** — no multi-user, no privilege escalation. One user, one broker session per instance.
- **Self-hosted on user's own server** — server access = full control. No SaaS component.
- All official install scripts (`install.sh`, `install-docker.sh`, `install-multi.sh`, `docker-run.sh`, `docker-run.bat`, `start.sh`) auto-generate unique `APP_KEY` and `API_KEY_PEPPER` via `secrets.token_hex(32)`.
- **SEBI static IP mandate** (effective April 1, 2026): All transactional API orders require broker-side static IP whitelisting. Delta Exchange (crypto) also enforces this. Stolen broker credentials CANNOT be used from an attacker's machine — the broker rejects requests from non-registered IPs. However, attacks routed THROUGH the OpenAlgo server (which has the registered IP) are still viable.
- External platforms (TradingView, GoCharting, Chartink) send API keys in JSON body or URL query params — they cannot set custom HTTP headers. This is an accepted architectural trade-off.
- The MCP server (`mcp/mcpserver.py`) is local-only, communicates via stdio with Claude Desktop/Cursor/Windsurf. It is NOT remotely exposed.
- Indian broker tokens expire daily at ~3:00 AM IST. Session management is aligned to this schedule.

## Development Environment Setup

### Prerequisites
- Python 3.12+ (required per pyproject.toml)
- Node.js 20.20+, 22.22+, or 24.13+ for React frontend development (per `frontend/package.json`)
- **uv package manager (required)** - Never use global Python

### Initial Setup

```bash
# Install uv package manager (required)
pip install uv

# Configure environment
cp .sample.env .env

# Generate new APP_KEY and API_KEY_PEPPER:
uv run python -c "import secrets; print(secrets.token_hex(32))"

# Run application (uv automatically handles virtual env and dependencies).
# The React frontend dist is force-committed to `main` by CI, so a fresh
# clone of main already has frontend/dist/ ready to serve. You only need
# to install Node and build locally if you are actively editing React code.
uv run app.py
```

### Important: Always Use UV

**Never use global Python or manually manage virtual environments.** Always prefix Python commands with `uv run`:

```bash
# Running the app
uv run app.py

# Running any Python script
uv run python script.py

# Installing a new package (adds to pyproject.toml)
uv add package_name

# Syncing dependencies after pulling changes
uv sync
```

### React Frontend Development

```bash
cd frontend

# Install dependencies
npm install

# Development server (hot reload)
npm run dev

# Production build
npm run build

# Run tests
npm test

# Run end-to-end tests
npm run e2e

# Linting and formatting
npm run lint
npm run format
```

## Application Architecture

### Frontend

**React 19 Frontend** (`/frontend/`): Modern SPA with TypeScript, Vite, shadcn/ui, TanStack Query. Built and served from `/frontend/dist/` by Flask via `blueprints/react_app.py`.

### Backend Structure

- `app.py` - Main Flask application entry point
- `blueprints/` - Flask route handlers (UI and webhooks)
- `restx_api/` - REST API endpoints (`/api/v1/`)
- `broker/` - Broker integrations (30+ brokers), each with `api/`, `database/`, `mapping/`, `streaming/`, `plugin.json`
- `services/` - Business logic layer
- `database/` - SQLAlchemy models and database utilities
- `utils/` - Shared utilities and helpers
- `websocket_proxy/` - Unified WebSocket server (port 8765)

### Database Architecture

OpenAlgo uses **6 separate databases** for isolation:

- `db/openalgo.db` - Main database (users, orders, positions, settings)
- `db/logs.db` - Traffic and API logs
- `db/latency.db` - Latency monitoring data
- `db/health.db` - Health monitoring data
- `db/sandbox.db` - Sandbox trading mode (isolated from live trading)
- `db/historify.duckdb` - Historical market data (DuckDB)

Each database has its own initialization function in `/database/`.

#### SQLite Connection Pooling (NullPool)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [white-trade-loan/algo-trading-platform](https://github.com/white-trade-loan/algo-trading-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
