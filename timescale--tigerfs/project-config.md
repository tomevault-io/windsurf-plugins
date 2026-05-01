---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Project Overview

TigerFS is a FUSE-based filesystem that exposes PostgreSQL databases as mountable directories. Users interact with tables, rows, and columns using Unix tools (`ls`, `cat`, `grep`, `rm`) instead of SQL.

**Primary Goal:** Enable Claude Code to explore and manipulate database data using Read/Glob/Grep operations.

**Full specification:** See `docs/spec.md` for complete technical details including filesystem structure, data formats, SQL patterns, and error codes.

## Development Commands

```bash
# Build
go build -o bin/tigerfs ./cmd/tigerfs

# Test
go test ./...                    # All tests
go test -race ./...              # With race detection
go test -run TestName ./path     # Specific test

# Synth app tests
go test ./internal/tigerfs/fs/synth/...                                      # Unit (synth package)
go test -run "^TestSynth_" ./internal/tigerfs/fs/... ./test/integration/...  # Unit (synth_ops) + integration

# Before committing (required)
go fmt ./... && go vet ./... && go test ./... && go mod tidy
```

## Architecture

### Code Style

This is primarily a Go codebase. Follow Go idioms: error handling with explicit returns, table-driven tests, and gofmt formatting.

### Key Packages

| Package | Purpose |
|---------|---------|
| `cmd/tigerfs/` | Entry point with signal handling |
| `internal/tigerfs/cmd/` | Cobra commands (pure functional builders) |
| `internal/tigerfs/fs/` | Shared filesystem logic (path parsing, operations, stat cache) |
| `internal/tigerfs/fuse/` | Linux FUSE adapter |
| `internal/tigerfs/nfs/` | macOS NFS adapter |
| `internal/tigerfs/db/` | PostgreSQL client and queries |
| `internal/tigerfs/backend/` | Cloud backend resolution (Tiger, Ghost, postgres://) |
| `internal/tigerfs/mount/` | Mount registry for tracking active mounts |
| `internal/tigerfs/config/` | Viper-based configuration |
| `internal/tigerfs/logging/` | Zap structured logging |
| `internal/tigerfs/format/` | Data serialization (TSV, CSV, JSON) |

### Mount Backends

macOS uses an in-process NFS v3 server (`go-nfs`); Linux uses FUSE (`go-fuse`). Both delegate to `fs.Operations` for all filesystem logic -- adapters are thin translation layers. New filesystem behavior goes in `fs/`, not in the adapters. Note: `fuse/` still contains legacy direct-to-DB code paths; the active default path routes through `fs/`.

**go-nfs write model:** NFS v3 is stateless -- `go-nfs` fabricates Open/Write/Close per WRITE RPC. Each Close commits the buffer to the DB. Multi-chunk writes cause repeated commits until the final Close. Stat must return accurate file sizes for NFS reads (GETATTR before READ).

### Pipeline Paths

Capability directories (`.filter`, `.order`, `.first`, `.last`, `.export`, `.columns`) accumulate query state as users navigate deeper. The path `products/.filter/in_stock/true/.order/price/.first/10/.export/json` builds a single SQL query. State is tracked in `FSContext` and parsed by `fs/path.go`.

### Synth Apps

Tables whose names start with `_` (e.g., `_blog`, `_docs`) can be exposed as synthesized views -- markdown files, task lists, or plain text snippets. The synth layer (`fs/synth/`) maps filesystem operations to table rows using format-specific conventions. See `docs/markdown-app.md` and `docs/tasks-app.md`.

### SQL Identifier Quoting

**Always use `db.QuoteIdent()` and `db.QuoteTable()` when interpolating identifiers into SQL strings.** Never use `fmt.Sprintf` with `"%s"` to quote identifiers -- it does not escape embedded double quotes, which can cause SQL syntax errors or injection. Within the `db` package, use the short aliases `qi()` and `qt()`.

| Helper | Purpose | Example |
|--------|---------|---------|
| `db.QuoteIdent(name)` / `qi(name)` | Quote a single identifier (column, table, or schema) | `QuoteIdent("email")` -> `"email"` |
| `db.QuoteTable(schema, table)` / `qt(schema, table)` | Quote a schema-qualified table reference | `QuoteTable("public", "users")` -> `"public"."users"` |

**When to use:** Any column, table, or schema name interpolated into SQL via `fmt.Sprintf`. From outside the `db` package, use the exported names. Do not use for SQL keywords, parameterized values (`$1`), or pre-built clause strings.

### Command Architecture: Pure Functional Builder Pattern

**Critical pattern:** All commands use functional builders with zero global state. See any existing `buildXxxCmd()` function for the pattern.

**Rules:**
- No global variables for commands, flags, or state
- Every command built by a `buildXxxCmd()` function
- Flag variables declared locally within builder functions
- Bind flags to viper in `PersistentPreRunE`/`PreRunE`, not at build time

## Configuration

**Precedence (low to high):** Defaults → Config file (`~/.config/tigerfs/config.yaml`) → Environment (`TIGERFS_*`) → Flags

**TLS enforcement:** TigerFS enforces `sslmode=require` for non-localhost connections. The `--insecure-no-ssl` flag or `InsecureNoSSL` config disables this. Localhost (127.0.0.1, ::1, Unix sockets) defaults to `sslmode=disable` when no sslmode is specified; users can override with an explicit `?sslmode=require`.

**Rules:**
1. Always use the `Config` struct - never read from viper directly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timescale/tigerfs](https://github.com/timescale/tigerfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
