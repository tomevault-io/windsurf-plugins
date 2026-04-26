---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Flask web application that generates ETA (Estimated Time of Arrival) reports by fetching order data from external OData APIs (BuzManager). The app supports multiple instances (DD and CBR), handles authentication via Google OAuth, manages customer configurations, and exports reports to CSV/XLSX formats.

## Development Commands

### Setup and Installation
```bash
# Install dependencies
pip install -r requirements-dev.txt

# Initialize database
export DATABASE=/path/to/your/database.db
flask init-db

# Database backup
flask db-backup [--dir /path/to/backup]
```

### Running the Application
```bash
# Development mode (set APP_ENV=development in .env)
python app.py

# Production mode
gunicorn app:app
```

### Testing
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_routes.py

# Run with verbose output
pytest -v

# Run specific test
pytest tests/test_routes.py::test_admin_page
```

### API
The `/api/v1/` REST API is implemented as a Flask Blueprint in the `api/` package.

**Authentication**: API key via `X-API-Key` header. Set `BUZ_API_KEY` environment variable.

**Endpoints**:
| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/customers` | List all customers |
| GET | `/api/v1/customers/<obfuscated_id>` | Get single customer |
| POST | `/api/v1/customers` | Create customer |
| PUT | `/api/v1/customers/<obfuscated_id>` | Update customer |
| DELETE | `/api/v1/customers/<obfuscated_id>` | Delete customer |
| POST | `/api/v1/reports/<obfuscated_id>/generate` | Start async report (returns job_id) |
| GET | `/api/v1/reports/<obfuscated_id>/download?format=csv\|xlsx` | Download report |
| GET | `/api/v1/jobs/<job_id>` | Job status/progress |
| GET | `/api/v1/statuses` | List status mappings |
| POST | `/api/v1/statuses/refresh` | Sync statuses from OData |
| GET | `/api/v1/health` | Health check (no auth required) |

**Response format**: `{"data": ..., "meta": {...}}` on success, `{"error": "message", "code": "ERROR_CODE"}` on error.

**Bot workflow**: Create customer -> POST generate -> Poll GET jobs/<id> -> GET download

### Cache Prewarming
```bash
# Prewarm cache for all customers (before blackout periods)
flask prewarm-cache

# Prewarm specific instances
flask prewarm-cache --instance DD --instance CBR
```

### Maintenance
```bash
# Purge completed/failed jobs older than 30 days (dry-run by default)
flask purge-jobs
flask purge-jobs --days 7 --confirm
```

## Architecture

### Multi-Instance OData Architecture
The application connects to two separate OData instances:
- **DD (DESDR)**: Uses `BUZ_DD_USERNAME` and `BUZ_DD_PASSWORD`
- **CBR (WATSO)**: Uses `BUZ_CBR_USERNAME` and `BUZ_CBR_PASSWORD`

Each customer record in the database can have a `dd_name` and/or `cbr_name`, and the system fetches data from both instances if configured.

### Async Report Generation
Reports are generated asynchronously to handle long-running OData queries:
1. **Route** (`/<obfuscated_id>`): Creates a job in the `jobs` table and spawns a background thread
2. **Worker** (`services/eta_worker.py` or `_run_eta_report_job()`): Executes in ThreadPoolExecutor
3. **Job Service** (`services/job_service.py`): Tracks progress, logs, and completion status
4. **Polling UI** (`templates/report_loading.html`): Frontend polls `/jobs/<job_id>` endpoint
5. **Render** (`/report/<job_id>`): Serves completed report from job result

The job tracking includes:
- Progress percentage
- Log messages
- Error handling with Sentry integration
- Stall detection (STALL_TTL = 300 seconds / 5 minutes - only for truly hung workers)

### Cache Strategy (services/cache.py & services/fetcher.py)
The application uses a SQLite-backed cache to handle:
- **Blackout periods**: Falls back to cached data when live OData is unavailable (503s)
- **Timeouts**: Returns cached data on connection errors or slow responses
- **Cooldown**: After detecting a 503, avoids hammering the API for N minutes
- **Force refresh**: Can bypass cache for real-time data when OData is available

Key function: `fetch_or_cached()` in `services/fetcher.py` orchestrates this logic.

### Database Schema
The database uses SQLite with Flask's `g` object for per-request connections:
- **customers**: Maps customer names to obfuscated IDs; supports `field_type` of "Customer Name" or "Customer Group"
- **users**: OAuth user management with roles (admin/user) and active status
- **status_mapping**: Maps OData production statuses to custom display names; can be synced from live data
- **jobs**: Tracks async report generation with status, progress, logs, and results
- **cache**: Generic key/value store with timestamps for cache invalidation

### Database Migrations
Migrations use `PRAGMA user_version` for versioning (see `services/migrations.py`):
- Migrations are idempotent and check for existence before altering
- Auto-backup before migrations (uses `VACUUM INTO`)
- Runs automatically on app startup via `run_migrations()`

To add a new migration:
1. Create `_migration_N_description()` function in `services/migrations.py`
2. Add to `MIGRATIONS` list at bottom of file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/derekgwatson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
