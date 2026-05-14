---
trigger: always_on
description: > **Purpose**: Guide for AI agents working **ON** the ecto_libsql codebase itself
---

# EctoLibSql - AI Agent Development Guide

> **Purpose**: Guide for AI agents working **ON** the ecto_libsql codebase itself
>
> **⚠️ IMPORTANT**: This guide is for **developing and maintaining** the ecto_libsql library.
> **⚠️ IMPORTANT**: For **USING** ecto_libsql in applications, see [USAGE.md](USAGE.md) instead.

---

## Quick Rules

- **British/Australian English** for all code, comments, and documentation (except SQL keywords and compatibility requirements)
- **⚠️ CRITICAL: ALWAYS check formatting BEFORE committing**:
  1. Run formatters: `mix format && cd native/ecto_libsql && cargo fmt`
  2. Verify checks pass: `mix format --check-formatted && cargo fmt --check`
  3. **Only then** commit: `git commit -m "..."`
- **NEVER use `.unwrap()` in production Rust code** — use `safe_lock` helpers (see [Error Handling](#error-handling-patterns))
- **Tests MAY use `.unwrap()`** for simplicity

---

## Landing the Plane (Session Completion)

**MANDATORY WORKFLOW — work is NOT complete until `git commit` succeeds:**

1. **File issues for remaining work** — create Beads issues for anything needing follow-up
2. **Run quality gates** (if code changed) — tests, linters, builds
3. **Update issue status** — close finished work, update in-progress items
4. **COMMIT**:
   ```bash
   git commit -m "Your commit message"
   bd sync
   ```
5. **Clean up** — clear stashes, prune remote branches
6. **Verify** — all changes committed
7. **Hand off** - provide context for next session

**If commit fails, resolve and retry until it succeeds.**

---

## Project Overview

**EctoLibSql** is a production-ready Ecto adapter for LibSQL, implemented as a Rust NIF for high performance.

**Connection modes:**
- **Local**: `database: "local.db"`
- **Remote**: `uri: "libsql://..." + auth_token: "..."`
- **Replica**: `database + uri + auth_token + sync: true`

---

## Architecture

### Layer Stack

```
Phoenix / Application
  ↓
Ecto.Adapters.LibSql (storage, type loaders/dumpers)
  ↓
Ecto.Adapters.LibSql.Connection (SQL generation, DDL)
  ↓
EctoLibSql (DBConnection protocol)
  ↓
EctoLibSql.Native (Rust NIF wrappers)
  ↓
Rust NIF (libsql-rs, connection registry, async runtime)
```

### Key Files

**Elixir:**
- `lib/ecto_libsql.ex` — DBConnection protocol
- `lib/ecto_libsql/native.ex` — NIF wrappers
- `lib/ecto_libsql/state.ex` — Connection state
- `lib/ecto/adapters/libsql.ex` — Main adapter
- `lib/ecto/adapters/libsql/connection.ex` — SQL generation

**Rust** (`native/ecto_libsql/src/`):

| Module | Purpose |
|--------|---------|
| `lib.rs` | Root module, NIF registration |
| `models.rs` | Core structs (`LibSQLConn`, `CursorData`, `TransactionEntry`) |
| `constants.rs` | Global registries (connections, transactions, statements, cursors) |
| `utils.rs` | Safe locking, error handling, row collection, type conversions |
| `connection.rs` | Connection establishment, health checks, encryption |
| `query.rs` | Query execution, auto-routing, replica sync |
| `statement.rs` | Prepared statement caching, parameter/column introspection |
| `transaction.rs` | Transaction management, ownership tracking, isolation levels |
| `savepoint.rs` | Nested transactions (create, release, rollback) |
| `batch.rs` | Batch operations (transactional/non-transactional) |
| `cursor.rs` | Cursor streaming, pagination for large result sets |
| `replication.rs` | Replica frame tracking, synchronisation control |
| `metadata.rs` | Insert rowid, changes, autocommit status |
| `decode.rs` | Value type conversions |
| `tests/` | Test modules |

**Tests:**
- `test/*.exs` — Elixir tests (adapter, integration, migrations, error handling, Turso)
- `native/ecto_libsql/src/tests/` — Rust tests (constants, utils, integration)

### Key Data Structures

```rust
pub struct LibSQLConn {
    pub db: libsql::Database,
    pub client: Arc<Mutex<libsql::Connection>>,
}

pub struct TransactionEntry {
    pub conn_id: String,        // Which connection owns this transaction.
    pub transaction: Transaction,
}

pub struct CursorData {
    pub columns: Vec<String>,
    pub rows: Vec<Vec<Value>>,
    pub position: usize,
}
```

---

## Development Workflow

### Branch Strategy

**ALWAYS branch from `main`** for new work:

```bash
git checkout main && git pull origin main
git checkout -b feature-descriptive-name   # or bugfix-descriptive-name
```

### ⚠️ CRITICAL: Preserving Untracked Files

- **NEVER run `git clean`** without explicit user approval
- **NEVER run `git checkout .`** or `git restore .` on the whole repo
- **NEVER run `git reset --hard`** without explicit user approval
- Untracked files stay in place across branch switches — this is expected

### PR Workflow

All changes go through PRs to `main`:

```bash
git push -u origin feature-descriptive-name
gh pr create --base main --title "feat: description" --body "..."
```

After merge:
```bash
git checkout main && git pull origin main
git branch -d feature-descriptive-name
```

### Pre-Commit Checklist

**STRICT ORDER — do NOT skip steps or reorder:**

```bash
# 1. Format code (must come FIRST)
mix format && cd native/ecto_libsql && cargo fmt

# 2. Run tests (catch logic errors)
mix test && cd native/ecto_libsql && cargo test

# 3. Verify formatting checks (MUST PASS before commit)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ocean/ecto_libsql](https://github.com/ocean/ecto_libsql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
