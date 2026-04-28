---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What is OptionLedger?

A web application for options traders using Tastytrade. It imports, organizes, and analyzes trading data — grouping transactions into position groups, detecting strategies, tracking rolls, and providing real-time P&L.

**Stack**: FastAPI backend, Vue 3 SPA (Vite + Tailwind), SQLAlchemy ORM (SQLite default, PostgreSQL via Docker), WebSocket for live quotes.

**Data flow**:
```
Tastytrade API → Import → Pipeline (7 stages) → Database → Web Interface
Live Market Data → WebSocket → Real-time Position Updates → Dashboard
```

## Running the Application

```bash
./start.sh          # Activates venv, builds frontend (vite build), runs python3 app.py
python app.py       # Direct start (http://localhost:8000)
docker compose up -d  # PostgreSQL + Redis (app, admin, postgres, redis services)
```

Default route `/` redirects to `/positions/options`.

### Alembic Migrations
```bash
venv/bin/alembic upgrade head                    # SQLite
DATABASE_URL=postgresql://... venv/bin/alembic upgrade head  # PostgreSQL
# For fresh DB created by create_all(), stamp baseline first:
venv/bin/alembic stamp 880552b12e57 && venv/bin/alembic upgrade head
```

## Architecture

### Backend Structure

```
src/
├── dependencies.py          # FastAPI deps (get_current_user_id, get_tastytrade_client)
├── schemas.py               # Pydantic schemas
├── sync_trades.py           # Legacy sync entrypoint
├── api/
│   └── tastytrade_client.py # OAuth2 auth, async methods, quote caching (30s TTL)
├── auth/
│   ├── jwt_validator.py     # Supabase JWT validation (HS256)
│   └── user_provisioning.py # Auto-create User rows on first login
├── database/
│   ├── engine.py            # init_engine(), dialect_insert(), dual-dialect support
│   ├── models.py            # 22 SQLAlchemy models
│   ├── db_manager.py        # DatabaseManager, get_session()
│   └── tenant.py            # Multi-tenant filtering (user_id scoping, ContextVar)
├── models/
│   ├── lot_manager.py       # LotManager — lot creation and closing logic
│   ├── order_models.py      # Enums (OrderType, OrderStatus) and domain types
│   ├── order_processor.py   # OrderProcessor — transaction grouping engine
│   ├── pnl_calculator.py    # PnLCalculator — P&L with roll chain handling
│   └── strategy_detector.py # StrategyDetector — option strategy identification
├── pipeline/
│   ├── orchestrator.py      # 7-stage pipeline orchestration
│   ├── order_assembler.py   # Stage: assemble orders from transactions
│   ├── position_ledger.py   # Stage: build position lots
│   ├── pnl_events.py        # Stage: populate pnl_events fact table
│   ├── group_manager.py     # Stage: create/update position groups
│   └── strategy_engine/     # Pattern-based strategy recognition
│       ├── recognizer.py    # Main recognizer entry point
│       ├── adapters.py      # Data adapters
│       ├── constants.py     # Strategy constants
│       ├── types.py         # Type definitions
│       └── patterns_*.py    # Pattern modules (single, vertical, combo, multi, calendar)
├── routers/                 # 13 FastAPI routers
│   ├── accounts.py          # /api/accounts, /api/account-balances
│   ├── auth.py              # /api/auth/config, /api/waitlist
│   ├── health.py            # /api/health, /api/connection/*
│   ├── ledger.py            # /api/ledger, /api/ledger/groups/*, /api/ledger/move-lots
│   ├── notes.py             # /api/order-comments, /api/position-notes
│   ├── pages.py             # /login, /beta-full, catch-all SPA route
│   ├── positions.py         # /api/positions/cached, /api/positions, /api/open-chains
│   ├── quotes.py            # /api/quotes
│   ├── reports.py           # /api/dashboard, /api/performance/*, /api/reports/*
│   ├── settings.py          # /api/settings/targets, /api/settings/credentials
│   ├── sync.py              # /api/sync, /api/reprocess, /api/sync/initial, /api/reconcile
│   ├── tags.py              # /api/tags, /api/tags/{id}
│   └── tastytrade_oauth.py  # /api/auth/tastytrade/authorize, /auth/tastytrade/callback
├── services/
│   ├── ledger_service.py    # Ledger business logic
│   ├── report_service.py    # Report/dashboard queries
│   └── sync_service.py      # Sync orchestration
└── utils/
    ├── auth_manager.py      # ConnectionManager singleton (per-user pool, LRU, 60-min TTL)
    └── credential_encryption.py  # Fernet encrypt/decrypt
```

### Pipeline (7 stages)

The sync pipeline (`src/pipeline/orchestrator.py`) processes data in order:
1. **Fetch** — pull transactions from Tastytrade API
2. **Save** — persist raw transactions to database
3. **Detect strategies** — identify multi-leg strategies via `strategy_engine/`
4. **Process orders** — group transactions into order chains
5. **Create lots** — build position lots from orders (`LotManager`)
6. **Process groups** — create/update position groups (`GroupManager`)
7. **Populate pnl_events** — denormalized fact table for time-based P&L reporting

### Database Models (22 tables)

**Core data**: Account, AccountBalance, Position, RawTransaction, SyncMetadata, QuoteCache (no user_id)
**Order system**: OrderChain, OrderChainCache, OrderComment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sbj175) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
