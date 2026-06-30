---
trigger: always_on
description: MediaFusion is a Rust-based Stremio and Kodi addon that aggregates streaming content from multiple sources (torrents, HTTP streams, YouTube, Telegram, debrid services, etc.). Version 6 is a full Rust rewrite of a previously Python-based backend.
---

## Project Overview

MediaFusion is a Rust-based Stremio and Kodi addon that aggregates streaming content from multiple sources (torrents, HTTP streams, YouTube, Telegram, debrid services, etc.). Version 6 is a full Rust rewrite of a previously Python-based backend.

## Commands

### Backend (Rust)

```bash
# Development server with auto-reload
make rust-dev
# OR: cargo run --manifest-path backend/Cargo.toml --bin mediafusion-api

# Background worker (scraping jobs, separate process)
cargo run --manifest-path backend/Cargo.toml --bin mediafusion-worker

# Release build
make rust-build
# OR: cd backend && cargo build --release --bin mediafusion-api --bin mediafusion-worker

# Tests
make rust-test
# OR: cd backend && cargo test

# Run a single test file
cd backend && cargo test --test jobs_validate_tv

# Backfill PTT metadata on existing torrent/usenet streams (see docs/deployment/worker-cli.md)
make worker-backfill-stream-metadata
# OR: make worker-backfill-stream-metadata BATCH_SIZE=2000

# Lint
make rust-lint
# OR: cd backend && cargo clippy --all-targets -- -D warnings

# Format check
make rust-fmt
# OR: cd backend && cargo fmt --check
```

### Frontend (React + TypeScript)

```bash
make frontend-install     # cd clients/frontend && npm ci
make frontend-dev         # cd clients/frontend && npm run dev
make frontend-build       # cd clients/frontend && npm run build
make frontend-lint        # cd clients/frontend && npm run lint
make frontend-fmt         # cd clients/frontend && npm run format:check
```

### Local Development Setup

```bash
# Start only PostgreSQL and Redis (minimal Docker)
cd deployment/docker-compose && docker compose -f docker-compose-minimal.yml up -d

# Migrations run automatically on API startup — no manual step needed
make rust-dev
```

### Docker Builds

```bash
make build VERSION=6.0.0          # Single-platform
make build-multi VERSION=6.0.0    # Multi-platform (amd64 + arm64)
```

### Run All Checks

```bash
make lint    # All linting
make fmt     # All format checks
make test    # All tests
```

## Deprecated Python Server & Workers (`python-deprecated/`)

Everything in this directory has been fully replaced by Rust equivalents. It is kept **for parity checking only** — do not run or deploy it. The FastAPI server, taskiq workers, APScheduler jobs, and Scrapy scrapers all have corresponding implementations in `backend/`. The Alembic migrations in `python-deprecated/migrations/` represent the v5 schema history; v6 uses sqlx migrations in `backend/migrations/`.

## Architecture

### Services

- **`mediafusion-api`** (Rust, Axum): HTTP server on port 8001. Handles Stremio addon manifest, catalog, stream, and user profile endpoints.
- **`mediafusion-worker`** (Rust): Long-running background job processor. Scrapes Prowlarr, RSS feeds, Telegram, YouTube, etc. Runs scheduled jobs.
- **PostgreSQL**: Primary data store. Migrations live in `backend/migrations/` as `NNNN_description.{up|down}.sql` and are applied automatically on startup via sqlx.
- **Redis**: Session cache, rate limiting, stream cache, job queue coordination.
- **Frontend** (`clients/frontend`): React 19 config UI. Built with Vite, TailwindCSS 4, Radix UI, React Query.
- **Kodi addon** (`clients/kodi`): Python-based Kodi client.

### Key Architectural Decisions

- **Unified stream table**: A base `Stream` table with type-specific sub-tables (`TorrentStream`, `HTTPStream`, `YouTubeStream`, etc.) linked by ID.
- **`StreamMediaLink`**: Many-to-many table that links streams to media, supporting specific file granularity within multi-file torrents.
- **sqlx over ORMs**: Queries in `src/db/` use compile-time-validated `sqlx::query!`/`query_as!` macros. Dynamic/route-level queries use runtime sqlx but must use the typed structs from `src/db/types.rs`. Query cache lives in `backend/.sqlx/`.
- **No ORM magic**: Schema changes require explicit migration files; `sqlx::query!` macros validate against the live DB at compile time via `cargo sqlx prepare`.
- **Multi-provider metadata**: Movies/series can have IDs from TMDB, TVDB, IMDb, MAL, Kitsu simultaneously.
- **Explicit quality columns**: Stream quality (resolution, codec, audio, HDR) stored as typed columns, not JSONB.

### Directory Layout

```
backend/          # Rust workspace (api + worker binaries, shared lib)
  src/            # Library code shared between binaries
  migrations/     # sqlx SQL migrations (run automatically on startup)
  tests/          # Integration tests
clients/
  frontend/       # React TypeScript web UI
  kodi/           # Kodi addon (Python)
deployment/
  docker-compose/ # Compose files for full-stack, minimal dev, HA, perf testing
  k8s/            # Kubernetes manifests
  Dockerfile
docs/             # env-reference.md, database-erd.md, performance.md
scripts/          # Build and release utilities
```

## Database Migrations

Migrations are in `backend/migrations/` with the format `NNNN_description.{up|down}.sql`. They run automatically when the API or worker starts. To check migration status or roll back:

```bash
MEDIAFUSION_MIGRATE=status ./mediafusion-api

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mhdzumair/MediaFusion](https://github.com/mhdzumair/MediaFusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
