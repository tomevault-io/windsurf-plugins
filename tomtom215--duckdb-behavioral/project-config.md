---
trigger: always_on
description: enables ClickHouse-compatible mode combinations (e.g., `strict | strict_increase`).
---

# CLAUDE.md — DuckDB Behavioral Analytics Extension

Engineering reference for the `duckdb-behavioral` codebase.

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Build & Test](#build--test)
- [Functions](#functions)
- [Dependencies](#dependencies)
- [Code Quality Standards](#code-quality-standards)
- [Performance](#performance)
- [ClickHouse Parity Status](#clickhouse-parity-status)
- [Testing](#testing)
- [CI/CD](#cicd)
- [Common Tasks](#common-tasks)

## Project Overview

`duckdb-behavioral` is a DuckDB loadable extension written in Rust that provides
behavioral analytics functions inspired by ClickHouse. It is listed in the
[DuckDB Community Extensions](https://github.com/duckdb/community-extensions/tree/main/extensions/behavioral)
repository and can be installed with `INSTALL behavioral FROM community; LOAD behavioral;`.
It can also be built from source, producing a `.so`/`.dylib` that DuckDB loads at runtime.

## Architecture

```
src/
├── lib.rs                  # Entry point via quack_rs::entry_point_v2! macro
├── common/
│   ├── mod.rs
│   ├── event.rs            # Event type (u32 bitmask conditions, Copy) shared by window_funnel, sequence_*
│   └── timestamp.rs        # Interval-to-microseconds conversion
├── pattern/
│   ├── mod.rs
│   ├── parser.rs           # Recursive descent parser for sequence patterns
│   └── executor.rs         # NFA-based pattern matcher
├── sessionize.rs           # Sessionize state (boundary-tracking for segment trees)
├── retention.rs            # Retention state (bitmask-based)
├── window_funnel.rs        # Window funnel state (greedy forward scan, bitflag modes)
├── sequence.rs             # Sequence match/count/events state (wraps pattern engine)
├── sequence_next_node.rs   # Sequence next node state (sequential matching, Arc<str> values)
└── ffi/
    ├── mod.rs              # register_all() — dispatches to all FFI modules via Registrar trait
    ├── sessionize.rs       # FFI callbacks for sessionize (raw libduckdb-sys — window function)
    ├── retention.rs        # FFI via quack-rs builder + returns_logical(LIST(BOOLEAN)) + ListVector
    ├── window_funnel.rs    # FFI via quack-rs builder + FfiState + VectorReader/VectorWriter
    ├── sequence.rs         # FFI via quack-rs builder for sequence_match + sequence_count
    ├── sequence_match_events.rs  # FFI via quack-rs builder + returns_logical(LIST(TIMESTAMP)) + ListVector
    └── sequence_next_node.rs     # FFI via quack-rs builder + VectorWriter::write_varchar
```

### Key Design Decisions

1. **Pure Rust core + FFI bridge**: Business logic lives in top-level modules
   (`sessionize.rs`, `retention.rs`, etc.) with zero FFI dependencies. The `ffi/`
   submodules handle DuckDB C API registration only.

2. **Aggregate functions via quack-rs SDK**: DuckDB's Rust crate does not yet
   provide high-level aggregate function registration. We use `quack-rs` v0.7.1
   ([crates.io](https://crates.io/crates/quack-rs)) which wraps the raw C API with safe builders
   (`AggregateFunctionSetBuilder`), state management (`FfiState<T>`), vector I/O
   (`VectorReader`/`VectorWriter` including `write_varchar`), complex type helpers
   (`ListVector`, `LogicalType::list()`), parameterized return type support
   (`returns_logical(LogicalType)`), and `AggregateTestHarness` for combine
   testing. All 6 aggregate functions use the builder for registration --
   including `retention` and `sequence_match_events` which use
   `returns_logical(LogicalType::list(...))` for their `LIST(T)` return types.
   `sessionize` remains fully hand-rolled (window function limitation in quack-rs).

3. **Function sets for variadic signatures**: Since `duckdb_aggregate_function_set_varargs`
   doesn't exist, we register function sets with 31 overloads (2-32 boolean parameters)
   via `AggregateFunctionSetBuilder::overloads(2..=32, ...)` which automatically calls
   `duckdb_aggregate_function_set_name` on each overload.

4. **Combinable `FunnelMode` bitflags**: `window_funnel` modes are represented as a
   `u8` bitflag struct (`FunnelMode(u8)`) rather than a mutually exclusive enum. This
   enables ClickHouse-compatible mode combinations (e.g., `strict | strict_increase`).
   Five ClickHouse modes are defined: `STRICT` (accepts both `'strict'` and
   `'strict_deduplication'` SQL strings, matching ClickHouse aliases), `STRICT_ORDER`,
   `STRICT_INCREASE`, `STRICT_ONCE`, `ALLOW_REENTRY`. One extension mode is defined:
   `STRICT_DEDUPLICATION` (SQL: `'timestamp_dedup'`), providing timestamp-based
   deduplication not present in ClickHouse.

5. **O(1) combine for sessionize**: The `SessionizeBoundaryState` tracks `first_ts`,
   `last_ts`, and `boundaries` count, enabling O(1) combine for DuckDB's segment
   tree windowing machinery.

6. **Entry point via `quack_rs::entry_point_v2!` macro**: The `behavioral_init_c_api`
   symbol is generated by the macro, which handles API initialization, connection
   management via `duckdb_connect`/`duckdb_disconnect`, and error reporting. The
   closure receives a `&Connection` implementing the `Registrar` trait, providing
   a version-agnostic API for registering extension components. Aggregate functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomtom215/duckdb-behavioral](https://github.com/tomtom215/duckdb-behavioral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
