---
trigger: always_on
description: - `/src/` - Backend Rust application code
---

# crates.io

## Repository Layout

- `/src/` - Backend Rust application code
  - `/src/bin/` - Binary entry points: `server.rs` (main API server), `background-worker.rs` (async job processor), `monitor.rs` (monitoring tool), `crates-admin/` (admin CLI tools)
  - `/src/controllers/` - API controllers organized by domain (`krate/`, `user/`, `version/`, `trustpub/`, `github/`, `admin/`)
  - `/src/worker/jobs/` - Background job implementations (crate analysis, README rendering, OG image generation, database dumps)
  - `/src/middleware/` - Request processing middleware (auth, rate limiting, logging, error handling)
  - `/src/tests/` - Backend integration tests with snapshot testing using `insta`
  - `/src/config/` - Configuration loading and validation
  - `/src/util/` - Shared utilities (errors, authentication, pagination)
- `/svelte/` - Frontend SvelteKit application
  - `/svelte/src/routes/` - File-based routes and page components
  - `/svelte/src/lib/` - Shared components, stores, services, and utilities
  - `/svelte/static/` - Static assets served at `/` (favicon, robots.txt, etc.)
- `/crates/` - Workspace crates providing specialized functionality
  - `crates_io_api_types/` - API response serialization types
  - `crates_io_database/` - Database models and schema (Diesel ORM)
  - `crates_io_worker/` - Background job queue system
  - `crates_io_index/` - Git index management for crate metadata
  - `crates_io_tarball/` - Package tarball processing and validation
  - `crates_io_trustpub/` - Trusted Publishing implementation
  - `crates_io_markdown/`, `crates_io_linecount/` - Content processing
- `/migrations/` - Database migrations (260+ historical migrations managed by Diesel)
- `/e2e/` - Playwright tests for the frontend with accessibility checks
- `/packages/` - MSW (Mock Service Worker) test utilities for API mocking
- `/script/` - Development utilities
- `/docs/` - Project documentation (`CONTRIBUTING.md`, `ARCHITECTURE.md`, PR review guidelines)

## General Guidance

- Read `/docs/CONTRIBUTING.md` for comprehensive setup instructions and contribution guidelines.
- Use `cargo insta accept` instead of `cargo insta review` when updating snapshot tests.
- Use `#[expect(...)]` instead of `#[allow(...)]` to silence warnings that should be resolved later.
- Match existing code style within each area (backend Rust follows workspace lints, frontend follows ESLint/Prettier config).
- Add documentation comments to new types and functions; existing code may lack comments, but new code should have them.

## Backend

### Building and Running

Run the API server:

```bash
cargo run
```

Run the background worker:

```bash
cargo run --bin background-worker
```

Database migrations:

```bash
diesel migration run              # Apply pending migrations
diesel migration revert           # Revert last migration
diesel migration redo             # Revert and reapply last migration
diesel migration generate <name>  # Create new migration
```

Admin CLI (check crate ownership, manage users, etc.):

```bash
cargo run --bin crates-admin -- <subcommand>
```

### Testing

Run backend tests (with separate test database):

```bash
cargo test
```

Run specific tests:

```bash
cargo test <test_name>
```

Accept updated snapshot tests:

```bash
cargo insta accept --workspace
```

Check code quality:

```bash
cargo fmt --all --check                                   # Formatting
cargo clippy --all-targets --all-features --workspace     # Linting
cargo doc --no-deps --document-private-items --workspace  # Doc lints
```

Test database setup: Set `TEST_DATABASE_URL` in `.env` to a separate database (e.g., `postgres://postgres@localhost/cargo_registry_test`). The test harness creates isolated databases and runs migrations automatically. Create the base test database once with `createdb --lc-collate=C --lc-ctype=C -T template0 cargo_registry_test`. The `C` collation is required by the `semver_ord()` function.

### Architecture and Conventions

- Use Axum web framework patterns (handlers, extractors, middleware).
- Follow Diesel ORM patterns for database queries; see models in `crates/crates_io_database/src/models/`.
- Use `anyhow::Result` for fallible operations; use `thiserror` for domain-specific error types.
- Background jobs go in `/src/worker/jobs/` and must implement the `Job` trait from `crates_io_worker`. Jobs must be idempotent.
- Controllers return responses via helper functions in `/src/util/` for consistent serialization.
- Use snapshot testing with `insta` for API responses.
- Never ignore deprecation warnings; fix them immediately or use `#[expect(deprecated)]`.
- Rate limiting configuration lives in `/src/rate_limiter.rs`; new endpoints should use appropriate limiters.
- CDN and caching behavior is configured in the [rust-lang/simpleinfra](https://github.com/rust-lang/simpleinfra) repository, not here. The backend only sets `Cache-Control`/`Vary` headers and whether the CDNs honor them depends on the Fastly/CloudFront config in that repo. See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for the CDN overview.
- Structured logging uses `tracing`; add spans for request context. See [`docs/LOGGING.md`](docs/LOGGING.md) for conventions on log messages, fields, and levels.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rust-lang/crates.io](https://github.com/rust-lang/crates.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
