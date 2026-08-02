---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
CobaltDB is a production-oriented pure-Go SQL database engine (zero CGO at runtime). It ships in two shapes from the same codebase:
- **Embedded library** — `github.com/cobaltdb/cobaltdb/pkg/engine` opened via `engine.Open(path, *engine.Options)` for in-process use (`:memory:` or disk-backed).
- **Standalone server** — `cmd/cobaltdb-server` speaks the MySQL wire protocol (default port 4200/3307) so any MySQL client or ORM works unchanged.

Supports standard SQL plus JSON, full-text search, window functions, CTEs, row-level security, temporal `AS OF` queries, HNSW vector search, replication, and AES-256-GCM encryption at rest.

**Note:** `AGENTS.md` was deleted as part of a cleanup pass (the root-level copy was a stale duplicate of this guide). Prefer editing `CLAUDE.md` for agent-facing instructions.

## Build & Verify

Use the Makefile — it pins the right flags. Outputs go to `bin/`.

```bash
make build              # builds bin/cobaltdb-server and bin/cobaltdb-cli
make verify             # go build + go vet + go test ./... (core gate)
make verify-security    # verify + race + vuln + gosec + lint (needs CGO for -race)
make race               # CGO_ENABLED=1 go test -race ./...
make lint               # golangci-lint runs globally across all packages (6 linters)
make test-coverage      # writes coverage.out + coverage.html
make release            # cross-compiles server+CLI for linux/darwin/windows × amd64/arm64 into dist/
```

The `lint` and `gosec` targets run golangci-lint and gosec globally across all packages — there is no per-package security restriction. The `SECURITY_PKGS` note in older docs is stale.

Runtime: `make run-server` / `make run-cli` use `go run`. Binaries already built at repo root (`cobaltdb-server`, `cobaltdb-cli`) are artifacts from prior builds, not canonical — always rebuild via `make build` into `bin/`.

## Architecture

### Core Packages
- `pkg/catalog` - SQL execution engine (SELECT, INSERT, UPDATE, DELETE, DDL)
- `pkg/query` - SQL parser, AST definitions, and query optimizer
- `pkg/btree` - B-tree storage engine (in-memory and disk-based)
- `pkg/storage` - Storage layer (buffer pool, WAL, encryption)
- `pkg/engine` - Database orchestration (the public embedded entrypoint); `circuit_breaker.go` and `retry.go` are wired into the MySQL wire protocol query path via `pkg/server` (see `refactor.md`)
- `pkg/txn` - Transaction manager, lock manager, MVCC, deadlock detection
- `pkg/wasm` - WebAssembly compiler and runtime for SQL execution
- `pkg/server` - MySQL protocol server implementation
- `pkg/protocol` / `pkg/wire` - MySQL wire protocol codec primitives
- `pkg/security` - Row-level security (RLS) policies
- `pkg/auth` - Authentication and authorization (Argon2id)
- `pkg/audit` - Audit logging (encrypted)
- `pkg/logger` - Structured logging used across packages
- `pkg/metrics` - Metrics collection, monitoring, slow query logging, AlertManager
- `pkg/optimizer` - Cost-based query optimizer with join reordering and index selection
- `pkg/cache` - Query result cache with TTL support
- `pkg/pool` - Connection pooling with health checks and dynamic sizing
- `pkg/replication` - Master-slave replication (async, sync, full_sync modes)
- `pkg/backup` - Backup and restore with compression support

### Key Components

#### Catalog
The Catalog is the central execution engine. It manages tables, indexes, and executes queries.

**File Organization:**
- `catalog_core.go` - Catalog struct, `selectLocked` dispatch, `scanTableRows`, table utilities
- `catalog_select.go` - JOIN execution, outer-query projection, view aggregate processing
- `catalog_select_helpers.go` - Column resolution, CTE handling, post-processing
- `catalog_eval.go` - Expression evaluation (`evaluateExpression`, `evaluateWhere`, `evaluateLike`, `evaluateIn`, `evaluateBetween`, function dispatch)
- `catalog_eval_json.go` - JSON function evaluation
- `catalog_eval_string.go` - String function evaluation (UPPER, LOWER, TRIM, SUBSTR, etc.)
- `catalog_aggregate.go` - GROUP BY, aggregates, HAVING, hidden column management
- `catalog_window.go` - Window functions (ROW_NUMBER, RANK, LAG, LEAD, aggregates OVER)
- `catalog_insert.go` - INSERT logic with constraint validation
- `catalog_update.go` - UPDATE/DELETE with JOIN support
- `catalog_delete.go` - DELETE with soft-delete and FK enforcement
- `catalog_ddl.go` - DDL operations (CREATE TABLE, indexes, triggers, policies)
- `catalog_fastpath.go` - COUNT(*) and SUM/AVG streaming fast paths
- `catalog_rls.go` - Row-level security helpers
- `catalog_txn.go` - Transaction management, rollback, undo replay
- `catalog_maintenance.go` - Save/Load, vacuum, analyze
- `catalog_cte.go` - CTE execution (recursive and non-recursive)
- `catalog_view.go` - Materialized view management
- `catalog_returning.go` - RETURNING clause evaluation

**Thread Safety:**
- Uses `sync.RWMutex` for concurrency control
- Public methods acquire locks, internal `*Locked` methods are lock-free
- The mutex is NOT reentrant - never call locked methods from locked methods

**Main Execution Paths:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cobaltdb/cobaltdb](https://github.com/cobaltdb/cobaltdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
