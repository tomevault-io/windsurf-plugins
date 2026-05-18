---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Architecture

This project is split into two parts:
- `/src` — OxiCloud Backend server in **Rust**
- `/static` — Oxicloud Frontend in **vanilla CSS & vanilla JavaScript**

# Backend part

## Backend Build & Dev Commands

```bash
cargo build                          # Dev build
cargo build --release                # Optimized release build
cargo run                            # Run server (port 8086)
cargo test --workspace               # Run all tests (~208)
cargo test <test_name>               # Run a single test by name
cargo test --features test_utils     # Run tests that use mockall mocks
cargo clippy -- -D warnings          # Lint (zero warnings policy)
cargo fmt --all --check              # Format check
cargo fmt --all                      # Auto-format
RUST_LOG=debug cargo run             # Run with debug logging
cargo run --bin generate-openapi     # Regenerate resources/gen/openapi.json
```

A `justfile` is available for common tasks (`just --list` to see all). Key recipes: `just check` (fmt + clippy), `just test`, `just openapi`.

Requires **Rust 1.93+** (edition 2024) and **PostgreSQL 13+** (with `pg_trgm` and `ltree` extensions).

Database setup: `docker compose up -d postgres` — schema is applied automatically via sqlx migrations on app startup. Migration files live in `migrations/`. For local dev, set `DATABASE_URL` in `.env` (see `example.env`).

## Backend Pre-commit checks

Always run these before committing, in this order:

```bash
cargo fmt --all                                              # Auto-format
cargo clippy --all-features --all-targets -- -D warnings     # Lint (must pass with zero warnings)
```

CI enforces both — commits that fail either check will not merge.

## Backend Architecture

Hexagonal / Clean Architecture with four layers. Dependencies point inward only.

### Layer structure (`src/`)

- **`domain/`** — Core business entities (`entities/`) and repository trait definitions (`repositories/`). Pure Rust, no framework dependencies. Entity types: `File`, `Folder`, `User`, `Calendar`, `CalendarEvent`, `Contact`, `Share`, `TrashedItem`, `Session`, `DeviceCode`, `AppPassword`.

- **`application/`** — Use cases and orchestration.
  - `ports/` — Trait definitions (inbound/outbound) for storage, auth, caching, compression, dedup, thumbnails, chunked uploads, CalDAV/CardDAV, etc. This is the hexagonal "ports" layer.
  - `services/` — Use case implementations (`FileManagementService`, `FolderService`, `ShareService`, `TrashService`, `CalendarService`, `ContactService`, `SearchService`, `BatchOperations`, etc.).
  - `adapters/` — CalDAV/CardDAV protocol adapters (iCalendar/vCard parsing).
  - `dtos/` — Data transfer objects for API boundaries.

- **`infrastructure/`** — Concrete implementations of ports.
  - `repositories/pg/` — All PostgreSQL repository implementations (via `sqlx`). Uses `auth` schema for users/sessions, `storage` schema for files/folders/blobs (content-addressable dedup with ltree paths).
  - `services/` — JWT, password hashing (Argon2), OIDC, compression, thumbnails, chunked uploads, WOPI discovery, WebDAV locking, file content caching (moka).
  - `adapters/` — CalDAV/CardDAV storage adapters bridging domain traits to PG.
  - `db.rs` — Dual connection pool setup (user pool + maintenance pool).

- **`interfaces/`** — HTTP layer (Axum).
  - `api/handlers/` — REST API handlers for files, folders, auth, admin, search, shares, WebDAV, CalDAV, CardDAV, WOPI, chunked uploads, batch operations.
  - `api/routes.rs` — Route registration, splits protected vs public routes.
  - `nextcloud/` — NextCloud-compatible API (WebDAV, OCS, login flow v2, trashbin) with Basic Auth middleware.
  - `middleware/` — Auth (JWT validation), CSRF, rate limiting.
  - `web/` — Static file serving.

- **`common/`** — Cross-cutting concerns.
  - `di.rs` — `AppServiceFactory` builds all services and produces `AppState` (the central DI container passed to Axum). This is the composition root.
  - `config.rs` — `AppConfig::from_env()` loads all `OXICLOUD_*` env vars.

### Key patterns

- **DI via `AppState`**: All services are `Arc`-wrapped and assembled in `common/di.rs`. `AppState` is wrapped in `Arc` and passed as Axum state. Many services are `Option<Arc<T>>` because they depend on features being enabled (auth, WOPI, trash, etc.).

- **Content-addressable storage**: Files use BLAKE3 blob dedup. `storage.file_blobs` stores content; `storage.file_metadata` references blobs with ref-counting. See `file_blob_write_repository.rs` and `file_blob_read_repository.rs`.

- **ltree paths**: Folder hierarchy uses PostgreSQL `ltree` for efficient subtree queries (recursive copies, moves, searches).

- **Dual DB pools**: `DbPools` in `infrastructure/db.rs` separates user-facing queries from maintenance/background tasks to prevent starvation.

- **Feature flags**: Major features (auth, trash, search, sharing, quotas) are toggled via `OXICLOUD_ENABLE_*` env vars in `FeaturesConfig`.

- **UUID columns**: All ID columns use native PostgreSQL `UUID` type. SQL queries must use `::uuid` casts when passing string parameters to UUID columns.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AtalayaLabs/OxiCloud](https://github.com/AtalayaLabs/OxiCloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
