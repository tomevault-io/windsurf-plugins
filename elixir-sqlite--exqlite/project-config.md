---
trigger: always_on
description: Exqlite is an Elixir SQLite3 library that uses Erlang NIFs (Native Implemented Functions) to interface with the SQLite C library. It provides both a low-level API (`Exqlite.Sqlite3`) and a high-level `DBConnection` implementation (`Exqlite.Connection`).
---

# Agent Development Guide

## Project Overview

Exqlite is an Elixir SQLite3 library that uses Erlang NIFs (Native Implemented Functions) to interface with the SQLite C library. It provides both a low-level API (`Exqlite.Sqlite3`) and a high-level `DBConnection` implementation (`Exqlite.Connection`).

**Key dependencies:**
- `db_connection` - Connection pooling and protocol
- `elixir_make` - Compiles C code via Makefile
- `cc_precompiler` - Cross-compilation support for precompiled NIFs

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Application Code / Ecto                                    │
├─────────────────────────────────────────────────────────────┤
│  Exqlite.Connection (DBConnection)                          │
├─────────────────────────────────────────────────────────────┤
│  Exqlite.Sqlite3 (Elixir API)     │  Exqlite.Query/Result   │
├─────────────────────────────────────────────────────────────┤
│  Exqlite.Sqlite3NIF (NIF bindings)                          │
├─────────────────────────────────────────────────────────────┤
│  c_src/sqlite3_nif.c (C NIF implementation)                 │
├─────────────────────────────────────────────────────────────┤
│  c_src/sqlite3.c (Vendored SQLite library)                  │
└─────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Files | Purpose |
|-------|-------|---------|
| **Connection** | `lib/exqlite/connection.ex` | DBConnection implementation, handles pooling, transactions, connection lifecycle |
| **Sqlite3 API** | `lib/exqlite/sqlite3.ex` | Elixir-friendly wrapper around NIF calls with type validation |
| **NIF Bindings** | `lib/exqlite/sqlite3_nif.ex` | Raw NIF function declarations (1:1 mapping to C) |
| **C NIF** | `c_src/sqlite3_nif.c` | NIF implementations, manages Erlang terms and SQLite objects |
| **SQLite** | `c_src/sqlite3.c` | Vendored SQLite library (or system SQLite via `EXQLITE_USE_SYSTEM`) |

## Key Files

### Elixir Source (`lib/`)

- `lib/exqlite.ex` - Public API (query, prepare, execute, transaction)
- `lib/exqlite/connection.ex` - DBConnection implementation (~780 lines)
- `lib/exqlite/sqlite3.ex` - Low-level SQLite3 interface
- `lib/exqlite/sqlite3_nif.ex` - NIF function stubs
- `lib/exqlite/query.ex` - Query struct and protocol implementation
- `lib/exqlite/result.ex` - Result struct
- `lib/exqlite/error.ex` - Error handling
- `lib/exqlite/stream.ex` - Stream protocol implementation
- `lib/exqlite/pragma.ex` - PRAGMA helpers
- `lib/exqlite/flags.ex` - SQLite open flags

### C Source (`c_src/`)

- `c_src/sqlite3_nif.c` - Main NIF implementation (~1700 lines)
- `c_src/sqlite3.c` - Vendored SQLite amalgamation
- `c_src/sqlite3.h` - SQLite header
- `c_src/sqlite3ext.h` - SQLite extension header

### Build Files

- `Makefile` - NIF compilation rules
- `mix.exs` - Elixir project configuration
- `.clang-format` - C code formatting

## Build System

### Compilation Flow

1. `mix compile` triggers `elixir_make` compiler
2. `elixir_make` invokes `make` with the `Makefile`
3. `Makefile` compiles `c_src/*.c` to `$(MIX_APP_PATH)/priv/sqlite3_nif.so`

### Build Commands

```bash
# Standard compilation
mix deps.get
mix compile

# Force native build (skip precompiled NIFs)
EXQLITE_FORCE_BUILD=1 mix compile

# Use system SQLite instead of vendored
EXQLITE_USE_SYSTEM=1 mix compile

# Verbose build output
V=1 mix compile

# Clean build artifacts
mix clean
make clean  # Also clean C objects
```

### Environment Variables

| Variable | Description |
|----------|-------------|
| `EXQLITE_USE_SYSTEM` | Use system SQLite instead of vendored `sqlite3.c` |
| `EXQLITE_FORCE_BUILD` | Force native compilation, skip precompiled NIFs |
| `EXQLITE_SYSTEM_CFLAGS` | Extra C compiler flags (e.g., `-I/usr/include`) |
| `EXQLITE_SYSTEM_LDFLAGS` | Extra linker flags (e.g., `-L/lib -lsqlite3`) |
| `V` | Set to `1` for verbose Make output |
| `DEBUG` | Set to enable debug build with `-g` flag |

### Precompiled NIFs

The project uses `cc_precompiler` to provide precompiled NIF binaries for common platforms. The precompiler configuration is in `mix.exs` and supports:
- Linux (x86_64, aarch64, riscv64 musl)
- macOS (universal)
- Windows
- Android (aarch64, armv7a)

## NIF Development Guidelines

### NIF Function Pattern

NIF functions in `c_src/sqlite3_nif.c` follow this pattern:

```c
// 1. Define the NIF function
static ERL_NIF_TERM exqlite_open(ErlNifEnv* env, int argc, const ERL_NIF_TERM argv[]) {
    // 2. Extract arguments from Erlang terms
    // 3. Call SQLite functions
    // 4. Return result as Erlang term (ok/error tuple)
}

// 5. Register in the NIF table
ERL_NIF_INIT(Elixir.Exqlite.Sqlite3NIF, nif_funcs, NULL, NULL, NULL, NULL)
```

### Important NIF Concepts

- **Dirty Schedulers**: All SQLite NIFs run on dirty schedulers to avoid blocking the BEAM scheduler
- **Resource Objects**: Connections and statements use `ErlNifResourceType` for safe garbage collection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elixir-sqlite/exqlite](https://github.com/elixir-sqlite/exqlite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
