---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`dft` (datafusion-dft) is a batteries-included suite of DataFusion applications providing four interfaces: TUI, CLI, FlightSQL Server, and HTTP Server. All interfaces share a common execution engine built on Apache DataFusion and Apache Arrow.

## Building and Running

### Development Commands

```bash
# Build the project (default features: functions-parquet, s3)
cargo build

# Build with TUI support
cargo build --features=tui

# Build with all features
cargo build --all-features

# Run the TUI (requires tui feature)
cargo run --features=tui

# Run CLI with a query
cargo run -- -c "SELECT 1 + 2"

# Start HTTP server (requires http feature)
cargo run --features=http -- serve-http

# Start FlightSQL server (requires flightsql feature)
cargo run --features=flightsql -- serve-flightsql

# Generate TPC-H data
cargo run -- generate-tpch
```

### Benchmarking

Benchmarks measure query performance with detailed timing breakdowns:

```bash
# Serial benchmark (default, 10 iterations)
cargo run -- -c "SELECT 1" --bench

# Custom iteration count
cargo run -- -c "SELECT 1" --bench -n 100

# Concurrent benchmark (measures throughput under load)
cargo run -- -c "SELECT 1" --bench --concurrent

# With custom iterations and concurrency
cargo run -- -c "SELECT 1" --bench -n 100 --concurrent

# Save results to CSV
cargo run -- -c "SELECT 1" --bench --save results.csv

# Append to existing results
cargo run -- -c "SELECT 2" --bench --concurrent --save results.csv --append

# Warm up cache before benchmarking
cargo run -- -c "SELECT * FROM t" --bench --run-before "CREATE TABLE t AS VALUES (1)"
```

**Benchmark Modes:**
- **Serial** (default): Measures query performance in isolation
  - Shows pure query execution time without contention
  - Ideal for understanding baseline performance

- **Concurrent** (`--concurrent`): Measures performance under load
  - Runs iterations in parallel (concurrency = min(iterations, CPU cores))
  - Shows throughput (queries/second) with multiple clients
  - Reveals resource contention and bottlenecks
  - Higher mean/median times are expected due to concurrent load

**Output:**
- Timing breakdown: logical planning, physical planning, execution, total
- Statistics: min, max, mean, median for each phase
- CSV format includes `concurrency_mode` column (serial or concurrent(N))

**FlightSQL Benchmarks:**
```bash
# Benchmark FlightSQL server (requires --flightsql flag and server running)
cargo run -- -c "SELECT 1" --bench --flightsql --concurrent
```

### Testing

Tests are organized by feature and component:

```bash
# Run core database tests
cargo test db

# Run CLI tests
cargo test cli_cases

# Run TUI tests (requires tui feature)
cargo test --features=tui tui_cases

# Run feature-specific tests
cargo test --features=flightsql extension_cases::flightsql -- --test-threads=1
cargo test --features=s3 extension_cases::s3
cargo test --features=clickhouse extension_cases::clickhouse  # Requires local ClickHouse (see tests/extension_cases/clickhouse.rs)
cargo test --features=mongodb extension_cases::mongodb  # Requires local MongoDB (see tests/extension_cases/mongodb.rs)
cargo test --features=functions-json extension_cases::functions_json
cargo test --features=deltalake extension_cases::deltalake
cargo test --features="deltalake s3" extension_cases::deltalake::test_deltalake_s3  # Requires LocalStack
cargo test --features=udfs-wasm extension_cases::udfs_wasm
cargo test --features=vortex extension_cases::vortex
cargo test --features=vortex cli_cases::basic::test_output_vortex
cargo test --features=net extension_cases::net  # Requires libpcap (libpcap-dev on Debian/Ubuntu)

# Run tests for specific crates
cargo test --manifest-path crates/datafusion-app/Cargo.toml --all-features
cargo test --manifest-path crates/datafusion-functions-parquet/Cargo.toml
cargo test --manifest-path crates/datafusion-udfs-wasm/Cargo.toml
cargo test --manifest-path crates/datafusion-net/Cargo.toml --all-features

# Run a single test
cargo test <test_name>
```

Note: FlightSQL tests require `--test-threads=1` because they spin up servers on the same port.

### Code Quality

```bash
# Format code
cargo fmt --all

# Check formatting (CI check)
cargo fmt --all -- --check

# Run clippy
cargo clippy --all-features --workspace -- -D warnings

# Check for unused dependencies
cargo machete

# Format TOML files
taplo format --check
```

## Architecture

### Crate Structure

The project is organized as a workspace with multiple crates:

- **Root crate (`datafusion-dft`)**: Main binary and application logic
  - `src/main.rs` - Entry point that routes to TUI, CLI, or servers
  - `src/tui/` - TUI implementation using ratatui
  - `src/cli/` - CLI implementation
  - `src/server/` - HTTP and FlightSQL server implementations
  - `src/config.rs` - Configuration management
  - `src/args.rs` - Command-line argument parsing

- **`crates/datafusion-app`**: Core execution engine (reusable library)
  - `src/local.rs` - ExecutionContext wrapping DataFusion SessionContext
  - `src/executor/` - Dedicated executors for CPU-intensive work (inspired by InfluxDB)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datafusion-contrib/datafusion-dft](https://github.com/datafusion-contrib/datafusion-dft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
