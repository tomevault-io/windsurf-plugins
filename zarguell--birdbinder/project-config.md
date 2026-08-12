---
trigger: always_on
description: > Read this before making ANY changes to this codebase. It will save you from repeating our mistakes.
---

# BirdBinder — Agent Development Guide

> Read this before making ANY changes to this codebase. It will save you from repeating our mistakes.

## Project Overview

BirdBinder is a self-hosted PWA that turns bird sighting photos into collectible digital cards. Users submit photos → AI identifies the species → cards are generated with AI art → users collect, trade, and organize cards in binders.

## Branching Strategy

| Branch | Purpose | CI Build |
|--------|---------|----------|
| `dev` | Day-to-day development. All PRs target here. | `ghcr.io/zarguell/birdbinder:dev` + SHA |
| `main` | Stable. Merge from dev to release. | `ghcr.io/zarguell/birdbinder:latest` + `vYYMMDD` + SHA |
| `v*` tags | Versioned releases after merging to main. | `ghcr.io/zarguell/birdbinder:X.Y.Z` + `X.Y` + SHA |

**Workflow:**
1. Develop on `dev` (or create PRs targeting `dev`)
2. When ready, merge to main: `git checkout main && git merge dev && git push`
3. Tag a release: `git tag vX.Y.Z && git push --tags` (or create a GitHub Release)
4. Renovate creates dependency PRs against `dev`

**Local dev:** Use `docker compose build` / `docker compose up` for local development. CI builds are for deployed environments.

## Repository & Architecture

```
birdbinder/
├── backend/                  # FastAPI application
│   ├── app/
│   │   ├── main.py           # App entry, CORS, router mounting, SPA serving
│   │   ├── config.py         # pydantic-settings (Settings class)
│   │   ├── db.py             # Async engine, Base, get_db()
│   │   ├── auth.py           # CF JWT decode + API key validation
│   │   ├── dependencies.py   # get_current_user() dependency
│   │   ├── ensure_schema.py  # Safety net for missing columns
│   │   ├── huey_instance.py  # SqliteHuey (data/huey.db)
│   │   ├── image.py          # HEIF→JPEG, EXIF extraction, thumbnails
│   │   ├── storage.py        # File save/retrieve helpers
│   │   ├── data/             # Static JSON (birds.json, regions.json)
│   │   ├── models/           # SQLAlchemy models (14 files)
│   │   ├── schemas/          # Pydantic request/response models
│   │   ├── routers/          # 11 router files
│   │   └── services/         # 10 service files
│   ├── migrations/           # Alembic migrations
│   └── tests/                # 25 test files
├── frontend/                 # SvelteKit SPA (adapter-static → backend/app/static/)
│   ├── src/lib/api.ts        # Typed API client
│   └── src/routes/           # 13 page routes
├── Dockerfile                # Multi-stage: Node 20 → Python 3.13
├── docker-compose.yml
└── entrypoint.sh             # alembic → ensure_schema → uvicorn + huey
```

## Tech Stack

| Layer | Tech |
|-------|------|
| Backend | Python 3.13, FastAPI, SQLAlchemy 2.0 (async), Alembic, Pydantic v2 |
| Database | SQLite (aiosqlite async) |
| Tasks | Huey (SqliteHuey, 2 process workers) |
| Frontend | Svelte 5 (runes), SvelteKit 2, Tailwind CSS 4, TypeScript |
| AI | OpenAI-compatible API (vision + image gen) |
| Auth | CF_Authorization JWT + Bearer API key |
| Build | Docker multi-stage, uv for Python deps, npm for frontend |

## Development Workflow

### Running Locally

```bash
# Backend (from backend/)
uv run uvicorn app.main:app --reload

# Frontend (from frontend/)
npm run dev

# Tests
uv run pytest tests/ -v

# Migrations
uv run alembic upgrade head
```

### TDD — ALWAYS Write Tests First

1. **RED** — Write a failing test that reproduces the bug or specifies the new behavior
2. **GREEN** — Write the minimal code to pass
3. **REFACTOR** — Clean up while tests stay green

Test command: `cd backend && .venv/bin/python -m pytest tests/test_FILE.py -v`

### Making Changes

1. Write the test first
2. Implement
3. Run the full test suite: `.venv/bin/python -m pytest tests/ -v`
4. Create migration if schema changed: `alembic revision --autogenerate -m "description"`
5. Commit and push

## Database Schema

### Tables & Relationships

```
users (email-based profiles)
  │
sightings ──1:N──→ cards ──1:N──→ binder_cards ←──N:1── binders
    │                  │                                      │
    └──1:N──→ jobs    └── referenced by ──→ activities ←──1:N── likes
                                                                           └──1:N── comments
card_sets (card_targets as JSON array of card IDs)
trades (offered_card_ids, requested_card_ids as JSON)
species_cache (static eBird taxonomy lookup)
app_settings (key-value store)
```

### FK Cascade Summary — CRITICAL

| FK | ondelete CASCADE? | ORM cascade? | Notes |
|----|-------------------|--------------|-------|
| Card.sighting_id → Sightings.id | ✅ | ✅ | Working |
| Job.sighting_id → Sightings.id | ✅ | ✅ | Working |
| BinderCard.binder_id → Binders.id | ✅ | ✅ | Working |
| BinderCard.card_id → Cards.id | ✅ | ❌ | **Manual cleanup needed** |
| Comment.activity_id → Activities.id | ✅ | ✅ | Working |
| Like.activity_id → Activities.id | ✅ | ✅ | Working |
| Activity.reference_id | **NO FK** | ❌ | **Loose string, always manual** |
| Trade card ID arrays | **NO FK** | ❌ | JSON arrays |
| CardSet.card_targets | **NO FK** | ❌ | JSON array |
| Binder.cover_card_id | **NO FK** | ❌ | Loose string |

### Enums

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zarguell/birdbinder](https://github.com/zarguell/birdbinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
