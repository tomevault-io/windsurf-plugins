---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pg_deltax is a PostgreSQL extension written in Rust (using pgrx 0.17) that provides time-series data management on top of native PostgreSQL declarative partitioning. It supports PostgreSQL 17 and 18 (default: 17).

## Build & Test Commands

All builds run inside Docker containers via the Makefile.

```bash
make dev-image                        # Build development Docker image (required first)
make build                            # Compile the extension
make test                             # Run pgrx unit tests (PG 17)
make test PG_MAJOR=18                 # Run pgrx unit tests against specific PG version
make clippy                           # Run Rust linter (includes test code)
make coverage                         # Unit test coverage report → coverage/html/
make coverage-all                     # Unit + integration test coverage → coverage/html/
make integration-test                 # Run Python integration tests (PG 17 & 18)
make integration-test PG_VERSIONS=17  # Integration tests for specific PG version(s)
make run                              # Start PostgreSQL with extension on port 5432
make psql                             # Connect to running instance
make image                            # Build production runtime Docker image
make image-fresh                      # Rebuild runtime image with --no-cache (use after source changes)
make run-sql SQL="SELECT 1"           # Build, run SQL, show output + server logs, teardown
make run-sql-file FILE="test.sql"     # Same as run-sql but reads SQL from a file
make logs                             # Show pg_deltax log lines from running container
make logs-all                         # Show all logs from running container
make logs-follow                      # Follow logs in real-time
make cargo CMD="<cmd>"                # Run arbitrary cargo command in dev container
make clean                            # Clean Docker volumes
make release VERSION=0.2.1            # Bump version, commit, tag v0.2.1, push (triggers Release workflow)
make bench-clickbench                 # Run Clickbench benchmark
make bench-clickbench-keep            # Run Clickbench benchmark, keep container running
make bench-rtabench                   # Run RTABench (plain PG vs pg_deltax) on a 250K-order subset
make bench-rtabench-keep              # Wipe DB volume (not CSV cache), reload + recompress, keep container running
make bench-rtabench-full              # RTABench with full 10M orders locally (slow, for parity with EC2)
make bench-rtabench-clean             # Remove RTABench container + PG volume (preserves CSV cache)
make bench-rtabench-distclean         # Full wipe including the ~7 GB CSV cache (forces redownload)
make bench-clean                      # Remove benchmark data volume
make bench-all                        # Compare benchmarks with timescale
```

### Benchmark Workflow

There are two benchmark environments: **local** (Docker, small data subset, more checks) and **full** (EC2, complete ClickBench dataset). On most changes, run both.

#### Local Benchmark (Docker)

1. `make image-fresh` — rebuild the production image with your code changes
2. `make bench-clickbench-keep` — run the benchmark (~2 min), keeps container running
3. Use the connection string printed at the end to run EXPLAIN ANALYZE or ad-hoc queries against the benchmark DB

The benchmark prints a `psql postgres://...` connection string at the end. Use it to investigate specific queries with EXPLAIN ANALYZE, verify plan choices, etc.

#### Full Benchmark (EC2)

Runs from `clickbench/Makefile` against a remote EC2 instance with the complete
ClickBench dataset (~100M rows). The EC2 IP is saved in the `.env` file in the
clickbench folder.

```bash
# First-time setup (installs PG18, Rust, pgrx, builds extension, loads data, compresses)
make -C clickbench setup EC2=<ip>

# Iterating on code changes
make -C clickbench deploy EC2=<ip>          # rsync source, recompile, restart PG
make -C clickbench bench EC2=<ip>           # run all 43 queries (3 runs each), download results

# Investigating specific queries
make -C clickbench query EC2=<ip> Q=33      # EXPLAIN ANALYZE a single query
make -C clickbench query-cold EC2=<ip> Q=7  # same but with cold caches (restarts PG, drops OS caches)
make -C clickbench query EC2=<ip> Q=33 SET="SET pg_deltax.parallel_workers=4"  # with GUC overrides

# Ad-hoc
make -C clickbench sql EC2=<ip> SQL="SHOW work_mem"
make -C clickbench psql EC2=<ip>
make -C clickbench ssh EC2=<ip>
```

Results are saved to `clickbench/results/pg_deltax.json` and archived by timestamp+commit in `clickbench/results/history/`.

### RTABench Workflow

[RTABench](https://rtabench.com) is a real-time-analytics benchmark built on a normalized 5-table schema (customers, products, orders, order_items, order_events) — a counterpoint to ClickBench's single wide denormalized table. 31 raw queries; pg_deltax only targets those (the 10 pre-aggregated `1000_*` queries rely on TimescaleDB continuous aggregates and are intentionally skipped).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xataio/deltax](https://github.com/xataio/deltax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
