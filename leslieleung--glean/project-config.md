---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Glean (拾灵) is a personal knowledge management tool and RSS reader built with a Python backend and TypeScript frontend. The project uses a monorepo structure with workspaces for both backend and frontend.

For backend-specific development guidance, see [backend/CLAUDE.md](backend/CLAUDE.md).
For frontend-specific development guidance, see [frontend/CLAUDE.md](frontend/CLAUDE.md).

## Quick Start

```bash
# Start infrastructure (PostgreSQL + Redis + Milvus)
make up

# Start all services (API + Worker + Web)
make dev-all

# Or run services individually
make api             # FastAPI server (http://localhost:8000)
make worker          # arq background worker
make web             # React web app (http://localhost:3000)
make admin           # Admin dashboard (http://localhost:3001)
make electron        # Electron desktop app
```

For detailed deployment instructions, see [DEPLOY.md](DEPLOY.md).

## Docker Compose Configuration

The project includes multiple Docker Compose configurations for different use cases:

### Production Deployment

```bash
# Basic deployment (without admin dashboard)
docker compose up -d

# Full deployment with admin dashboard
docker compose --profile admin up -d

# Stop services
docker compose down

# Test pre-release versions (alpha/beta/rc)
IMAGE_TAG=v0.3.0-alpha.1 docker compose up -d
# Or set in .env: IMAGE_TAG=v0.3.0-alpha.1
```

### Development Environment

```bash
# Start development infrastructure (PostgreSQL, Redis, Milvus)
docker compose -f docker-compose.dev.yml up -d

# View logs
docker compose -f docker-compose.dev.yml logs -f

# Stop services
docker compose -f docker-compose.dev.yml down
```

### Local Development with Override

```bash
# Use local builds instead of Docker images
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

### Test Environment

```bash
# Start test database (port 5433, isolated from dev)
docker compose -f docker-compose.test.yml up -d

# Or use Makefile shortcut
make test-db-up
```

### Environment Variables

Key environment variables for Docker deployments:

- `WEB_PORT`: Web interface port (default: 80)
- `ADMIN_PORT`: Admin dashboard port (default: 3001)
- `POSTGRES_DB/USER/PASSWORD`: Database credentials
- `SECRET_KEY`: JWT signing key
- `CREATE_ADMIN`: Create admin account on startup (default: false)
- `ADMIN_USERNAME/PASSWORD`: Admin credentials
- `DEBUG`: Enable debug mode (default: false)

For a complete list of environment variables, see `.env.example` in the project root.

## Development Commands

### Database Migrations
```bash
make db-upgrade                    # Apply migrations
make db-migrate MSG="description"  # Create new migration (autogenerate)
make db-downgrade                  # Revert last migration
make db-reset                      # Drop DB, recreate, and apply migrations (REQUIRES USER CONSENT)
```

Working directory: `backend/packages/database` | Tool: Alembic (SQLAlchemy 2.0)

### Testing & Code Quality
```bash
make test            # Run pytest for all backend packages/apps
make test-cov        # Run tests with coverage report
make lint            # Run ruff + pyright (backend), eslint (frontend)
make format          # Format code with ruff (backend), prettier (frontend)

# Frontend-specific (from frontend/ directory)
pnpm typecheck                          # Type check all packages
pnpm --filter=@glean/web typecheck      # Type check specific package
pnpm --filter=@glean/web build          # Build specific package
```

### Package Management
```bash
# Root: npm (for concurrently tool)
npm install

# Backend: uv (Python 3.11+)
cd backend && uv sync --all-packages

# Frontend: pnpm + Turborepo
cd frontend && pnpm install
```

## Architecture

### Technology Stack

| Layer       | Backend                                | Frontend                 |
| ----------- | -------------------------------------- | ------------------------ |
| Language    | Python 3.11+ (strict pyright)          | TypeScript (strict)      |
| Framework   | FastAPI                                | React 18 + Vite          |
| Database    | SQLAlchemy 2.0 (async) + PostgreSQL 16 | -                        |
| State/Cache | Redis 7 + arq                          | Zustand + TanStack Query |
| Styling     | -                                      | Tailwind CSS             |
| Package Mgr | uv                                     | pnpm + Turborepo         |
| Linting     | ruff + pyright                         | ESLint + Prettier        |

**Infrastructure**: PostgreSQL 16 (5432), Redis 7 (6379), Milvus (optional), Docker Compose

### Backend Structure

```
backend/
├── apps/
│   ├── api/           # FastAPI REST API (port 8000)
│   │   └── routers/   # auth, feeds, entries, bookmarks, folders, tags, admin, preference
│   └── worker/        # arq background worker (Redis queue)
│       └── tasks/     # feed_fetcher, bookmark_metadata, cleanup, embedding_worker, preference_worker
├── packages/
│   ├── database/      # SQLAlchemy models + Alembic migrations
│   ├── core/          # Business logic and domain services

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeslieLeung/glean](https://github.com/LeslieLeung/glean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
