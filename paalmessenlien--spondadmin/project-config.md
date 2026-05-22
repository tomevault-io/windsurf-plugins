---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spond Admin Interface - A full-stack web application for managing Spond events, groups, and members. The system syncs data from the Spond API and provides administrative capabilities with analytics dashboards.

## Commands

### Docker (Recommended)

```bash
# Development - starts db, backend (hot-reload), frontend (HMR)
docker compose up
docker compose up --build          # rebuild after dependency changes
docker compose down                # stop all services
docker compose logs -f backend     # tail backend logs

# Production
docker compose -f docker-compose.prod.yml up -d
./scripts/deploy.sh                # full deployment (backup, build, migrate, health check)

# Create admin user (Docker)
docker compose exec backend python3 create_admin.py

# Run migrations (Docker)
docker compose exec backend alembic upgrade head

# Database backup/restore (Docker)
./scripts/backup.sh
./scripts/restore.sh backups/<file>.dump
```

### Local Development (without Docker)

```bash
cd backend
source venv/bin/activate

# Start development server
uvicorn app.main:app --host 127.0.0.1 --port 8001 --reload

# Database migrations
alembic revision --autogenerate -m "Description"
alembic upgrade head

# Create admin user
python3 create_admin.py

# Reset admin password
python3 reset_admin_password.py
```

```bash
cd frontend

# Start development server
npm run dev

# Build for production
npm run build
```

### Kill Stale Processes

```bash
lsof -ti:8001 | xargs kill -9  # Backend
lsof -ti:3000 | xargs kill -9  # Frontend
```

## Architecture

### Backend (`backend/`)

- **Framework**: FastAPI with async SQLAlchemy ORM
- **Database**: PostgreSQL (production + Docker dev) / SQLite (local dev)
- **Auth**: JWT tokens with bcrypt password hashing, role-based access (admin/editor/viewer)
- **External API**: Spond API via `spond` and `spond-classes` libraries
- **Scraping**: bueskyting.no competition results, records, and upcoming events via crawl4ai/httpx+BeautifulSoup
- **AI Integration**: Configurable AI providers (OpenAI, Anthropic, DeepSeek) for event analysis

**Key Directories**:
- `app/api/v1/` - API route handlers (auth, events, groups, members, analytics, scheduler, scores, scraper, backups, migrations, ai_providers, external_events)
- `app/services/` - Business logic, Spond sync, backup, migration, and scraper services
- `app/models/` - SQLAlchemy ORM models
- `app/schemas/` - Pydantic validation schemas
- `app/core/` - Config (`config.py`), security (`security.py`), dependencies (`deps.py`)

**Sync Architecture**: Each entity (events, groups, members) has paired services:
- `*_service.py` - CRUD operations
- `*_sync_service.py` - Spond API synchronization

**Background Jobs**: APScheduler runs automatic syncs at configurable intervals (see `.env.example`).

### Frontend (`frontend/`)

- **Framework**: Nuxt 3 with Vue 3
- **UI**: Nuxt UI v3 with Tailwind CSS v4
- **State**: Pinia stores
- **Charts**: Chart.js with vue-chartjs

**Key Directories**:
- `pages/dashboard/` - Main app pages (events/, groups/, members/, analytics.vue, reports/, scores/, competitions/, settings/)
- `components/` - Reusable Vue components (charts, empty states, breadcrumbs)
  - `components/reports/` - Report-specific table components with sorting/filtering
- `composables/` - Composable functions for shared logic (`useApi.ts`, `usePermissions.ts`)
- `stores/` - Pinia state management
- `layouts/` - Page layouts (dashboard.vue)
- `middleware/` - Auth middleware

**Routing Pattern**: List pages use `pages/dashboard/{entity}/index.vue`, detail pages use `pages/dashboard/{entity}/[id].vue`.

### Data Flow

1. Spond API → Sync Services → PostgreSQL
2. bueskyting.no → Scraper Service → PostgreSQL
3. Frontend → Backend API → Database
4. Background scheduler refreshes data automatically

### Docker Infrastructure

- `docker-compose.yml` - Development (db + backend + frontend, hot-reload)
- `docker-compose.prod.yml` - Production (+ nginx reverse proxy, resource limits, logging)
- `nginx/` - Nginx config with SSL/TLS, rate limiting, security headers
- `scripts/` - Deployment, backup, restore, and SQLite-to-PostgreSQL migration scripts
- `monitoring/` - Service health check script

### Backup System

- **Backend**: `BackupService` uses `pg_dump`/`pg_restore` with Bunny CDN offsite storage
- **API**: `/backups/` CRUD endpoints (admin only)
- **Frontend**: Settings > Database Backups page
- **Scripts**: `scripts/backup.sh` (scheduled), `scripts/restore.sh` (CLI)
- CDN path: `spondadmin/backups/` in the shared `archery-trainer-storage` Bunny CDN zone

### Migration Management

- **Backend**: `MigrationService` wraps Alembic operations (status, history, run)
- **API**: `/migrations/status`, `/migrations/history`, `/migrations/run` (admin only)
- **Frontend**: Settings > Migrations page

## Configuration

### Docker Development
Uses `backend/.env.docker` with PostgreSQL defaults. Just run `docker compose up`.

### Local Development
Backend config via `backend/.env` (copy from `.env.example`):
- `SPOND_USERNAME` / `SPOND_PASSWORD` - Spond API credentials

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Paalmessenlien/spondadmin](https://github.com/Paalmessenlien/spondadmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
