---
trigger: always_on
description: Guidance for coding agents working in `yoink`.
---

# AGENTS.md
Guidance for coding agents working in `yoink`.

## Project Overview
- `yoink` is a self-hosted music library manager.
- Backend: Rust 2024, Axum, Tokio, SeaORM, utoipa/OpenAPI, tracing.
- Frontend: React 19 SPA in `frontend/` using TanStack Start, TanStack Router, TanStack Query, TanStack DB, shadcn/ui v4, Tailwind CSS v4, Bun, and Vite.
- The frontend is built separately and embedded into the server binary via `rust-embed`.
- Workspace crates:
  - `crates/yoink-server`: API server, auth, providers, services, DB entities, background workers.

## Repo-Specific Rule Files
- Checked for Cursor rules in `.cursor/rules/` and `.cursorrules`: none found.
- Checked for Copilot rules in `.github/copilot-instructions.md`: none found.
- The only repository-local agent instructions currently live in this `AGENTS.md`.

## Setup
- Install toolchain helpers with `mise install`.
- Install frontend dependencies with `bun install` in `frontend/`.
- Copy env defaults with `cp .env.example .env` if needed.
- Backend dev server runs on `http://127.0.0.1:3000`.
- Frontend dev server runs on `http://localhost:5173` and proxies `/api/**` and `/auth/**` to the backend.

## Build, Lint, and Test Commands
### Root / Combined
- Lint everything wired through mise: `mise run lint`
- Run both dev servers through mise: `mise run dev`
- Start Docker services: `docker compose up -d`
- Start dev Docker services: `docker compose -f compose.dev.yaml up -d`

### Backend (Rust)
- Run server once: `cargo run -p yoink-server`
- Run server through mise: `mise run run-server`
- Watch mode: `mise run dev-server`
- Build debug: `cargo build -p yoink-server`
- Build release: `cargo build -p yoink-server --release`
- Format: `cargo fmt`
- Format check: `cargo fmt --check`
- Lint: `cargo clippy --package yoink-server -- -D warnings`
- Server tests: `cargo test -p yoink-server`
- Whole workspace tests: `cargo test --workspace`
- List backend tests: `cargo test -p yoink-server -- --list`
- Run tests matching a name substring: `cargo test -p yoink-server list_jobs_returns_album_and_track_jobs`
- Run one exact backend test: `cargo test -p yoink-server list_jobs_returns_album_and_track_jobs -- --exact`
- Run one exact fully-qualified backend test: `cargo test -p yoink-server services::downloads::tests::list_jobs_returns_album_and_track_jobs -- --exact`

### Frontend
- Dev server: `bun run dev`
- Production build: `bun run build`
- Preview production build: `bun run preview`
- Lint: `bun run lint`
- Auto-fix lint issues: `bun run lint:fix`
- Format: `bun run fmt`
- Format check: `bun run fmt:check`
- Run frontend tests: `bun run test`
- Run one frontend test file: `bun x vitest run src/lib/router/breadcrumbs.test.ts`
- Run one named frontend test: `bun x vitest run src/lib/router/breadcrumbs.test.ts -t "returns static breadcrumb labels"`

### API Type Generation
- Regenerate frontend API types from a running backend: `mise run gen-frontend-types`
- This writes `frontend/src/lib/api/types.gen.ts`.

## Validation Guidance
- Small Rust change: `cargo fmt --check && cargo test -p yoink-server`
- Frontend change: run `bun run lint && bun run fmt:check && bun run test` in `frontend/`
- API contract change: also run `mise run gen-frontend-types`
- Release-oriented change: run `bun run build` in `frontend/` before `cargo build -p yoink-server --release`

## Architecture Notes
- The server bootstraps from `crates/yoink-server/src/main.rs` and builds the provider registry there.
- Routing is assembled in `crates/yoink-server/src/routes/mod.rs` using `OpenApiRouter`.
- App state lives in `crates/yoink-server/src/state.rs` and holds the DB connection, provider registry, SSE broadcaster, auth service, and shared settings.
- The database is initialized through SeaORM schema sync in `AppState::new`; do not assume SQLx query macros or committed `.sqlx` metadata are in use.
- Frontend routes are file-based under `frontend/src/routes/`.
- `frontend/src/routeTree.gen.ts` and `frontend/src/lib/api/types.gen.ts` are generated; do not hand-edit them.

## Rust Style Guidelines
### Imports and Layout
- Group imports as `std` first, external crates second, `crate::` imports last.
- Prefer nested imports when it improves readability, for example `use std::{path::PathBuf, sync::Arc};`.
- Prefer explicit imports over globs.
- Keep top-level module declarations in `main.rs` or `lib.rs`.
- Keep `mod tests` adjacent to the code it tests; this repo strongly favors inline test modules.

### Formatting
- Follow `rustfmt` defaults; do not hand-format against the formatter.
- Use trailing commas in multiline enums, structs, function calls, and macro arguments.
- Split long method chains and builder chains one call per line.
- Preserve section-divider comments when present; banner comments are used in larger files.
- Add doc comments for public or non-obvious behavior, not for obvious plumbing.

### Types and Data Modeling
- Server-owned API/OpenAPI DTOs live in `crates/yoink-server/src/api/`.
- Shared models usually derive `Serialize`, `Deserialize`, and `ToSchema`.
- Use `Uuid::now_v7()` for new persistent IDs; SeaORM active models commonly set this in `ActiveModelBehavior::new()`.
- Prefer enums over raw strings for domain state such as quality, wanted status, or download status.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlyinPancake/yoink](https://github.com/FlyinPancake/yoink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
