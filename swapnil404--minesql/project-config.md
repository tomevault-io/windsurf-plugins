---
trigger: always_on
description: This file helps coding agents (Claude, Cursor, Codex, etc.) understand the mineSQL codebase and work effectively within it.
---

# AGENTS.md

This file helps coding agents (Claude, Cursor, Codex, etc.) understand the mineSQL codebase and work effectively within it.

---

## Project Overview

mineSQL is a Postgres-wire-compatible relational database engine written in Go. Its storage backend is a live Minecraft world — every row is a strip of banner blocks and signs standing on grass, the transaction log is a row of lecterns to the west of spawn, and internal metadata lives underground. It implements real database internals: WAL, MVCC, a query executor, and a SQL parser.

Full technical spec (authoritative): [`docs/spec.md`](./docs/spec.md)

**When in doubt about any decision — build order, wire protocol messages, WAL invariants, row encoding format, plugin protocol opcodes — the spec wins.**

The Paper plugin (Java, separate repo) lives at: `github.com/swapnil404/minesql-hal`

---

## Repository Layout

```
minesql/
├── cmd/minesql/         # main entrypoint — starts wire server, initializes HAL
├── internal/
│   ├── wire/            # Postgres wire protocol (jackc/pgproto3)
│   ├── parser/          # SQL parsing (pganalyze/pg_query_go)
│   ├── planner/         # query planner — produces execution plans
│   ├── executor/        # pull-based query executor
│   ├── storage/         # row encoding, strip layout, table metadata, codec
│   ├── wal/             # write-ahead log, crash recovery
│   ├── mvcc/            # transaction manager, xmin/xmax visibility
│   └── hal/             # Minecraft HAL — TCP client to Paper plugin
├── docker/
│   ├── Dockerfile       # Paper server image with plugin pre-installed
│   └── world/           # pre-seeded superflat world
├── docker-compose.yml
├── docker-compose.dev.yml
└── docs/
    ├── spec.md
    └── architecture.md
```

---

## Build & Run

```bash
# Build
go build ./cmd/minesql

# Run (requires Minecraft server running, see docker-compose.dev.yml)
MINESQL_MINECRAFT_ADDR=localhost:25576 MINESQL_PORT=5455 go run ./cmd/minesql

# Run all tests
go test ./...

# Run tests for a specific package
go test ./internal/storage/...

# Lint
golangci-lint run
```

### Dev Environment

Start Minecraft only (engine runs locally):
```bash
docker compose -f docker-compose.dev.yml up
```

Full stack:
```bash
docker compose up
```

---

## Code Style

- **Formatter**: `gofmt` — all code must be gofmt clean, no exceptions
- **Linter**: `golangci-lint` — run before any commit
- **Error handling**: always handle errors explicitly, never `_` an error unless there is a clear documented reason
- **No global state**: pass dependencies explicitly, do not use package-level variables for mutable state
- **Interfaces over concrete types**: especially at package boundaries (e.g. `hal.Storage`, not `hal.MinecraftHAL` directly)
- **Context propagation**: all blocking operations (HAL calls, TCP reads) must accept and respect `context.Context`
- **Package naming**: short, lowercase, no underscores — `wal`, `mvcc`, `hal`, `wire`

---

## World Layout

```
     west  ←  Z=0 (spawn)  →  east
              │
Z < 0         │         Z > 0
WAL lecterns  │         user table data
(transaction  │         (banner+sign strips
 log books)   │          standing on grass)
              │
         underground (Y < 60)
         catalog barrels + control block
```

- **Z > 0, Y=64** — user table data. Table N (N >= 1) starts at Z = (N-1) × 10000. Each row is a strip of banners and signs along X.
- **Z < 0, Y=64** — WAL lecterns. Each lectern is one transaction log entry. Walk west = older transactions.
- **Underground (Y=10)** — catalog (table metadata as barrels) and control block at (0, 10, 0). Never visible to users.

---

## Layer Responsibilities

Understanding which layer owns what prevents putting logic in the wrong place:

| Layer | Owns | Does NOT own |
|---|---|---|
| `wire` | Postgres protocol framing, message types | SQL parsing, query logic |
| `parser` | AST construction from SQL string | planning, execution |
| `planner` | Choosing scan strategy, plan tree | executing plans |
| `executor` | Iterating over plan, producing rows | storage format, Minecraft |
| `storage` | Row encoding, strip layout, table metadata | WAL, MVCC, Minecraft |
| `wal` | Log entry write/read, crash recovery | storage format details |
| `mvcc` | Transaction IDs, xmin/xmax visibility | row encoding |
| `hal` | TCP connection to plugin, block I/O | everything above |

**Critical rule**: the `hal` package must never import any other internal package. It is the bottom of the dependency graph.

---

## Storage Format

Rows use a hybrid banner+sign layout. Each row occupies a fixed-width strip of blocks along X at Y=64, Z=(tableZStart + rowIndex):

- **Banners 0–1**: xmin (int64, 8 bytes across 2 banners)
- **Banners 2–3**: xmax (int64, null = 0xFFFFFFFFFFFFFFFF)
- **Banners 4..N**: INT/BIGINT/BOOLEAN columns in ordinal order. INT = 1 banner (4 bytes, 2 wasted). BIGINT = 2 banners. BOOLEAN = 1 banner.
- **Signs 0..M**: TEXT columns. One standing OAK_SIGN per TEXT column (4 lines × 16 chars = 64 chars max).

Banner byte encoding — each of the 6 pattern layers encodes 1 byte:
- High nibble (bits 7–4) = pattern type index 0–15

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swapnil404/mineSQL](https://github.com/swapnil404/mineSQL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
