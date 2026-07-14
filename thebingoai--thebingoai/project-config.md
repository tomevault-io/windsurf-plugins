---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A BI platform with AI-powered dashboards, real-time chat, and multi-database connectivity. Built with FastAPI backend and Nuxt 4 frontend. Features RAG via LangGraph, multi-provider LLM support (OpenAI, Anthropic, Ollama), SSO authentication (via Bingo SSO), and Celery + Redis for async background processing.

## Project Structure
This is the community edition. It can run standalone or as a git submodule inside the enterprise repo.
Enterprise extends community via plugins/overlays. Never assume enterprise features live on a branch.
When used as a submodule, this repo lives at `bingo-enterprise/bingo/`.

## Phase 0 Primitives

These two cross-cutting helpers shipped in Phase 0 (data-platform-v1 branch). Every later phase relies on them.

### `backend/auth/system_context.py` — system actor marker

Background tasks (Pipeline runner, dbt subprocess, profiling) wrap code that the enterprise governance plugin (Phase G) would otherwise gate on per-user RBAC:

```python
from backend.auth.system_context import system_context, current_system_context

with system_context(reason="pipeline.run", scope=owner_scope) as ctx:
    ...  # current_system_context() returns ctx inside this block
```

Phase G reads `current_system_context()` → when set, skips RBAC and writes an `audit_events` row with `actor_user_id = NULL`. Phase 0 ships only the marker; the audit write is in Phase G.

### `backend/config/feature_flags.py` — per-Org feature gates

Flags live in `organizations.feature_flags` (JSONB), cached in Redis (`org:{id}:feature_flags`, TTL 60s).

```python
from backend.config.feature_flags import enabled, set_flag, requires_flag, FLAG_DISABLED

enabled("org-uuid", "new_data_plane")             # bool, default False
set_flag("org-uuid", "new_data_plane", True)       # write Postgres + bust Redis

@requires_flag("new_data_plane")
def materialize_new(org_id: str, ...):             # returns FLAG_DISABLED when flag is off
    ...
```

Flag registry lives as `KNOWN_FLAGS` in `feature_flags.py`. Guarded by a canary test — add to the set when introducing a new flag. `org_id` is a UUID string (matches `organizations.id`).

## Docker
- Use `docker compose` (v2), not `docker-compose` (v1).
- Redis URLs inside Docker network use service names (e.g., `redis://redis:6379`), not `localhost`.
- Always verify compose file paths before running commands.

## Git Operations
- SSH remote URL: `git@github.com:thebingoai/thebingoai.git`
- Always complete the full commit-and-push cycle in one go; don't stop after staging.
- Check `git remote -v` before pushing if unsure of remote configuration.

## Working Style section 

### Planning vs Implementation
When asked to implement something, proceed directly to implementation after a brief plan outline. Do not spend the entire session in planning mode unless explicitly asked for a plan only. Avoid over-engineering — prefer simple, pragmatic solutions.

### File Reading
Do not re-read files you have already read in this session. Track what you've explored and avoid redundant exploration. If you need to reference something you already read, use your memory of it.

## Development Setup

### Local Development (Recommended)

If you also have the enterprise overlay checked out, see the enterprise `CLAUDE.md` for combined-repo docker commands.

**Community standalone:**
```bash
# Start community only
./start.sh

# Access points:
# - Frontend: http://localhost:3000
# - API: http://localhost:8000
# - API Docs: http://localhost:8000/docs
# - Health: http://localhost:8000/health
```

Requirements:
- Docker and Docker Compose must be installed and running
- `.env` configured with required API keys and database URL

`start.sh` (community only) auto-detects the database mode from `DATABASE_URL` in `.env`:
- **Supabase (default)**: External DB URL → skips Docker PostgreSQL
- **Local PostgreSQL**: `localhost` or `postgres:` URL → includes Docker PostgreSQL via override compose file



### Database Setup

**Option 1: Supabase (recommended)**
1. Create a Supabase project at https://supabase.com
2. Go to Settings > Database > Connection string > URI
3. Set `DATABASE_URL` in `.env` to the connection pooler URI (port 6543)
4. Optionally set `DATABASE_URL_DIRECT` to the direct connection URI (port 5432) for migrations

**Option 2: Local PostgreSQL**
1. Set `DATABASE_URL=postgresql://thebingo_user:thebingo_password@localhost:5432/thebingo` in `.env`
2. `start.sh` will automatically include Docker PostgreSQL

Migrations run automatically on startup via `alembic upgrade head` in the Dockerfile CMD. When using Supabase, set `DATABASE_URL_DIRECT` to bypass the connection pooler for migrations.

### Backend Only (Docker)

```bash
# Supabase mode (no local PostgreSQL)
docker compose -f docker/local/docker-compose.yml up -d

# Local PostgreSQL mode
docker compose -f docker/local/docker-compose.yml -f docker/local/docker-compose.postgres.yml up -d

# View logs
docker compose -f docker/local/docker-compose.yml logs -f backend

# Rebuild after code changes
docker compose -f docker/local/docker-compose.yml up --build -d
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thebingoai/thebingoai](https://github.com/thebingoai/thebingoai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
