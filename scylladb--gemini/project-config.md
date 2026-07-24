---
trigger: always_on
description: Generates random CQL statements (INSERT, UPDATE, DELETE, SELECT) for a given table with
---

# Gemini Project

Gemini is an automated randomized testing suite for ScyllaDB and Apache Cassandra.
It generates random CQL mutations (INSERT, UPDATE, DELETE) and applies them to two clusters
simultaneously: an **oracle** (source of truth) and a **system under test** (SUT).
It then runs SELECT queries on both clusters and compares results row-by-row.
Any difference indicates a bug in the SUT.

## How Gemini Works

1. **Schema**: Generated from a random seed or loaded from JSON. Defines keyspace, tables,
   partition/clustering keys, columns, indexes, materialized views, and UDTs.
2. **Partitions**: A pool of partition key values is initialized with a configurable statistical
   distribution (uniform, zipfian, sequential, lognormal) that controls access patterns.
3. **Warmup** (optional): Write-only phase with no deletes to populate initial data.
4. **Main phase**: Runs in one of four modes:
   - `mixed` (default) -- concurrent mutation + validation workers
   - `write` -- mutation workers only
   - `read` -- validation workers only
   - `warmup` -- writes without deletes
5. **Workers**: Per-table pools managed by `errgroup`:
   - `mutationConcurrency` Mutation workers generate random INSERT/UPDATE/DELETE statements
   - `readConcurrency` Validation workers run SELECT queries and compare results
6. **Mutations**: Applied to both clusters in parallel with retry and exponential backoff.
   Each cluster is retried independently.
7. **Validation**: Rows loaded from both clusters in parallel, sorted by PK+CK, compared via
   `CompareCollectedRows`. List values are deduplicated (LWT retry artifact workaround).
8. **Delete tracking**: Deleted partitions are tracked in time buckets via a min-heap.
   After configurable delays, they are re-validated (expecting zero rows on both clusters).
9. **Error budget**: When max errors reached, workers stop gracefully via `stop.Flag`.

## Project Structure

- `pkg/` -- all packages for the gemini project
- `docs/` -- project documentation (architecture, partitions, metrics, etc.)
- `scripts/` -- development and testing scripts
- `docker/` -- Dockerfiles and docker-compose files for dev environment and CI

### Key Packages

| Package | Purpose |
|---|---|
| `pkg/cmd` | CLI entry point, flag parsing, root command |
| `pkg/jobs` | Mutation/Validation worker orchestration |
| `pkg/store` | Dual-cluster CQL operations and row comparison |
| `pkg/partitions` | Thread-safe partition pool with distributions |
| `pkg/statements` | CQL statement generation with cached queries |
| `pkg/stmtlogger` | Async statement logging framework |
| `pkg/stmtlogger/scylla` | ScyllaDB + file statement writer |
| `pkg/typedef` | Core types: Schema, Table, Column, Type interface, Stmt |
| `pkg/builders` | Schema builder pattern |
| `pkg/distributions` | Statistical distributions for partition selection |
| `pkg/status` | Global ops/error counters |
| `pkg/stop` | Graceful shutdown flag (soft/hard) |
| `pkg/metrics` | Prometheus metric definitions |
| `pkg/joberror` | Structured error types with comparison results |
| `pkg/utils` | Backoff, timers, file helpers |
| `pkg/random` | Goroutine-safe random number generation |
| `pkg/replication` | Replication strategy configuration |
| `pkg/murmur` | Murmur3 hash for token-aware routing |
| `pkg/inflight` | In-flight request tracking |
| `pkg/tableopts` | Table options for CQL CREATE TABLE |
| `pkg/schema` | Schema generation from seeds |
| `pkg/services` | Supporting services (pprof, etc.) |
| `pkg/benchmarks` | Performance benchmarks |
| `pkg/testutils` | Test helpers (ScyllaDB containers, etc.) |

## Language & Framework

- **Language**: Go 1.25
- **Architecture**: CLI application with modular package structure
- **Database**: Scylla/Cassandra using CQL protocol
- **Testing**: Randomized testing with statistical distributions

## Go Instructions (1.24, 1.25)

1. Always use `for range` instead of `for` with `i++` when iterating, this is the new syntax from Go 1.25
   Example: `for i := 0; i < 10; i++` should be replaced with `for i := range 10`. If the `i` is not needed it can be
   omitted.
   This is also true when using some integer variable as a counter. `for i := 0; i < VARIABLE; i++` can be replaced with
   `for i := range VARIABLE`, also `i` can be omitted.
2. Always assume `go` **1.25**. Prefer `go` commands that work with Go 1.25.
3. Keep `go.mod` `go 1.25`. Do **not** add a `toolchain` line when updating the `go` line (Go 1.25 no longer auto-adds
   it).
4. Use the new `go.mod` **`ignore`** directive to exclude non-packages (e.g., examples, scratch) from `./...`
5. Prefer standard library first; avoid third-party deps unless asked.
6. Slice stack-allocation opportunities (new in 1.25) and avoid unsafe pointer aliasing.
7. Use testing/synctest for flaky/racy tests.
8. Consider the container-aware GOMAXPROCS defaults when benchmarking.
9. **DWARF 5** debug info by default
10. Add a `synctest` based test that removes `time.Sleep` and waits deterministically.
11. Use `go test -race` to detect data races.
12. Always use in tests for context `t.Context()` and for benchmarking `b.Context()`, there are new go 1.24 function,
    and they better and avoid linting errors.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scylladb/gemini](https://github.com/scylladb/gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
