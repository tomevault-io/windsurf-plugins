---
trigger: always_on
description: Full-stack Rust framework. Single binary, PostgreSQL-only. Axum/Tokio/SQLx backend with SvelteKit or Dioxus frontends. Proc macros generate handler traits, codegen produces type-safe frontend bindings.
---

# Forge Framework Development

Full-stack Rust framework. Single binary, PostgreSQL-only. Axum/Tokio/SQLx backend with SvelteKit or Dioxus frontends. Proc macros generate handler traits, codegen produces type-safe frontend bindings.

## Pre-1.0 Policy

Zero tech debt. Breaking changes are encouraged without a migration path if they produce a cleaner API or simpler internals. No backward compatibility shims, deprecated fields, legacy aliases, or "old way / new way" coexistence until stable release. If removing something makes the system better, remove it.

## Workspace

```
crates/forge           CLI + public lib (publishes as `forgex`). Builder pattern, auto-registration via inventory crate, template scaffolding.
crates/forge-core      Types, traits, contexts, errors, config, testing infra. All handler traits (ForgeQuery, ForgeMutation, ForgeJob, etc.) defined here.
crates/forge-macros    Proc macros. Transforms #[query]/#[mutation]/#[job]/etc into trait impls + inventory::submit! registration. SQL table extraction via sqlparser.
crates/forge-runtime   Gateway (Axum router, SSE, RPC dispatch), job worker (SKIP LOCKED polling), workflow executor, cron scheduler, daemon runner, reactivity (LISTEN/NOTIFY -> invalidation -> re-execute -> SSE fan-out), cluster coordination (advisory lock leader election), rate limiting, observability, signals (product analytics + diagnostics).
crates/forge-codegen   AST parser (syn) -> SchemaRegistry -> BindingSet IR -> emitters. TypeScript (SvelteKit) and Rust (Dioxus) targets. All type mapping in emit.rs (single source of truth).
packages/forge-svelte  @forge-rs/svelte npm package. Runtime: ForgeClient, stores, SSE transport, auth, ForgeSignals (analytics tracker).
packages/forge-dioxus  forge-dioxus crate. Dioxus WASM client runtime, hooks, signals, ForgeSignals (analytics tracker).
examples/with-svelte/  minimal, demo, realtime-todo-list (each a workspace member)
examples/with-dioxus/  minimal, demo, realtime-todo-list
benchmarks/app         Performance testing
docs/                  Docusaurus site
```

## Stack

Rust 1.92, edition 2024. Axum 0.8, Tokio 1.48, SQLx 0.8 (compile-time checked). PostgreSQL 18. Svelte 5 (runes, SvelteKit 2) / Dioxus 0.7. Bun for JS. OpenTelemetry via OTLP HTTP (port 4318).

## Commands

```bash
cargo build --workspace                  # all crates
cargo build -p forgex                    # CLI only
cargo test --workspace                   # unit tests (SQLX_OFFLINE=true, no DB)
cargo test -p forge-svelte-demo-template --features testcontainers  # integration (needs Docker)
cargo clippy --all-targets --all-features --workspace -- -D warnings
cargo fmt --all --check

# full CI-style template test (scaffolds project, starts PG, builds, runs playwright)
scripts/ci/test-template.sh with-svelte/demo target/debug/forge .
```

### Regenerating .sqlx cache

The `.sqlx/` directory holds compile-time query metadata for `SQLX_OFFLINE=true` builds. Regenerate it when queries change.

The workspace has a shared schema problem: `benchmarks/app` and the demo examples both define a `users` table with different shapes. The solution is a single "super" database with all tables merged, plus a combined `forge_migrations` table that satisfies both `migrations/executor.rs` (uses `version` + `checksum`) and `migrations/runner.rs` (uses `name` + `down_sql`).

```bash
# 1. Start a temporary PG instance
docker run -d --name forge-sqlx-pg -e POSTGRES_PASSWORD=forge -e POSTGRES_DB=forge \
  -p 5433:5432 postgres:18
until docker exec forge-sqlx-pg pg_isready -U postgres -d forge 2>/dev/null; do sleep 1; done

# 2. Apply system schema
docker exec -i forge-sqlx-pg psql -U postgres -d forge \
  < crates/forge-runtime/migrations/system/v001_initial.sql

# 3. Apply example migrations (demo covers users/trades/etc; todos adds todos table)
sed '/^-- @down/,$d' examples/with-svelte/demo/migrations/0001_initial.sql \
  | docker exec -i forge-sqlx-pg psql -U postgres -d forge
sed '/^-- @down/,$d' examples/with-svelte/realtime-todo-list/migrations/0001_todos.sql \
  | docker exec -i forge-sqlx-pg psql -U postgres -d forge

# 4. Add benchmark-only tables (counters; skip its users — conflicts with demo)
docker exec forge-sqlx-pg psql -U postgres -d forge -c "
  CREATE TABLE counters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL UNIQUE,
    value BIGINT NOT NULL DEFAULT 0,
    updated_by UUID REFERENCES users(id),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
  );"

# 5. Create combined forge_migrations table (merges both internal schemas)
docker exec forge-sqlx-pg psql -U postgres -d forge -c "
  CREATE TABLE forge_migrations (
    id SERIAL PRIMARY KEY,
    version VARCHAR(255) NOT NULL DEFAULT '',
    name VARCHAR(255) NOT NULL DEFAULT '',
    applied_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    checksum VARCHAR(64),
    execution_time_ms INTEGER,
    down_sql TEXT
  );"

# 6. Generate
DATABASE_URL="postgres://postgres:forge@localhost:5433/forge" \
  cargo sqlx prepare --workspace

# 7. Cleanup
docker stop forge-sqlx-pg && docker rm forge-sqlx-pg
```

## Crate Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isala404/forge](https://github.com/isala404/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
