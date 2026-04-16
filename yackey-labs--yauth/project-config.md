---
trigger: always_on
description: `yauth` is a modular, plugin-based authentication library for Rust (Axum) with TypeScript client + SolidJS UI packages. It provides email/password, passkey (WebAuthn), MFA (TOTP + backup codes), OAuth, bearer tokens (JWT), API keys, and admin endpoints — all behind feature flags.
---

# CLAUDE.md — yauth

## What This Is

`yauth` is a modular, plugin-based authentication library for Rust (Axum) with TypeScript client + SolidJS UI packages. It provides email/password, passkey (WebAuthn), MFA (TOTP + backup codes), OAuth, bearer tokens (JWT), API keys, and admin endpoints — all behind feature flags.

**Repo:** `github.com/yackey-labs/yauth`

## Workspace Structure

### Rust Crates (`crates/`)

| Crate | Purpose |
|---|---|
| `yauth` | Main library — plugins, middleware, builder, auth logic, backends, repository traits |
| `yauth-entity` | Domain types (User, Session, Password, etc.) — ORM-agnostic, no migration dependency |
| `yauth-migration` | Schema types, DDL generation, diff engine, migration file gen — **zero ORM deps**, build-time-only dep of `cargo-yauth` (NOT a dependency of `yauth` itself) |
| `cargo-yauth` | CLI binary — `cargo yauth init/add-plugin/remove-plugin/status/generate` |

Key internal modules in `yauth`:
- `backends/diesel_pg/` — PostgreSQL backend (`DieselPgBackend`)
- `backends/diesel_mysql/` — MySQL/MariaDB backend (`DieselMysqlBackend`)
- `backends/diesel_libsql/` — SQLite/Turso backend (`DieselLibsqlBackend`)
- `backends/diesel_sqlite/` — Vanilla SQLite backend (`DieselSqliteBackend`)
- `backends/sqlx_pg/` — PostgreSQL backend via sqlx (`SqlxPgBackend`)
- `backends/sqlx_mysql/` — MySQL backend via sqlx (`SqlxMysqlBackend`)
- `backends/sqlx_sqlite/` — SQLite backend via sqlx (`SqlxSqliteBackend`)
- `backends/memory/` — In-memory backend (`InMemoryBackend`)
- `backends/redis/` — Redis caching decorators
- `repo/` — `DatabaseBackend` trait (provides `repositories()`), repository traits, `Repositories` struct, `RepoError`
- `domain/` — ORM-agnostic domain types (always compiled, no backend deps)

Key modules in `yauth-migration`:
- `types` — `TableDef`, `ColumnDef`, `ColumnType`, `ForeignKey`, `Dialect`
- `core` — Core table definitions (users, sessions, audit_log)
- `plugin_schemas` — Schema definitions for each plugin
- `collector` — Schema collection + topological sort by FK deps
- `postgres/sqlite/mysql` — Dialect-specific DDL generators
- `diff` — Schema diff engine (CREATE TABLE, DROP TABLE, ALTER TABLE)
- `generate` — Migration file generators (diesel up.sql/down.sql, sqlx numbered .sql)
- `config` — `yauth.toml` config file support
- `tracking` — Schema hash computation

### TypeScript Packages (`packages/`)

| Package | Purpose |
|---|---|
| `@yackey-labs/yauth-shared` | Shared types (`AuthUser`, `AuthSession`, AAGUID map) |
| `@yackey-labs/yauth-client` | HTTP client for all auth endpoints |
| `@yackey-labs/yauth-ui-vue` | Vue 3 components + composables (LoginForm, useAuth, etc.) |
| `@yackey-labs/yauth-ui-solidjs` | SolidJS components (LoginForm, RegisterForm, etc.) |

## Key Commands

```bash
# Rust
cargo test --features full,all-backends          # Run all unit tests
cargo fmt --check                                 # Format check
cargo clippy --features full,all-backends -- -D warnings  # Lint

# TypeScript
bun install                          # Install dependencies
bun run build                        # Build all TS packages
bun run lint                         # Biome lint check
bun run lint:fix                     # Biome lint + fix
bun run typecheck                    # TypeScript type check
bun validate                         # lint:fix + typecheck + build
bun validate:ci                      # lint + typecheck + build + generate:check (strict)

# Client generation
bun generate                         # Regenerate TS client from Rust types + routes
bun generate:check                   # Fail if generated client is out of date (CI)

# Integration / Pentest (all parallel-safe — no --test-threads=1 needed)
docker compose up -d                 # Start PostgreSQL + MySQL + Redis + Mailpit
cargo test --features full,all-backends --test pentest                      # OWASP pentest (memory + diesel_pg + diesel_mysql)
cargo test --features full,all-backends --test diesel_integration           # Diesel PG integration tests
cargo test --features full,all-backends --test diesel_mysql_integration     # Diesel MySQL integration tests

# Conformance tests (cross-backend repository trait verification)
# All tests share a single tokio runtime via OnceLock, so connection pools survive across tests.
DATABASE_URL=postgres://yauth:yauth@127.0.0.1:5433/yauth_test \
MYSQL_DATABASE_URL=mysql://yauth:yauth@127.0.0.1:3307/yauth_test \
  cargo test --features full,all-backends --test repo_conformance

# Schema generation CLI (generates migration files for your ORM — no runtime migrations)
cargo yauth init --orm diesel --dialect postgres --plugins email-password,passkey
cargo yauth add-plugin mfa
cargo yauth remove-plugin passkey
cargo yauth status
cargo yauth generate --check -f yauth.toml
```

### Conformance Test Suite


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yackey-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
