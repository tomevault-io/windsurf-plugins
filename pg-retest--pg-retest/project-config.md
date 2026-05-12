---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**pg-retest** (working title for EDB Database Testing Kit / EDTK) is a tool for capturing, replaying, and scaling PostgreSQL database workloads. It enables users to validate performance across configuration changes, server migrations, and scaling scenarios.

### Core Capabilities (by milestone)

1. **Capture & Replay** — Capture SQL workload from a PG server (per-connection thread profiling), replay it against a backup database, produce side-by-side performance comparison. Support read-write and read-only (strip DML) modes.
2. **Scaled Benchmark** — Classify captured workload into categories (analytical, transactional, etc.), scale each independently to simulate increased traffic for capacity planning.
3. **CI/CD Integration** — Automate the capture/replay/compare cycle as a pipeline step with pass/fail thresholds.
4. **Cross-Database Capture** — Capture from Oracle, MySQL, MariaDB, SQL Server and transform into PG-compatible workload for replay.
5. **AI-Assisted Tuning** — Use AI to recommend config, schema, and query changes; test iterations and produce comparison reports.

### Key Design Constraints

- Workload capture must have minimal impact on production systems.
- Transactions change data, which changes query plans. For accurate 1:1 replay, restore from a point-in-time backup before replay.
- Two distinct modes are needed: **true replay** (exact 1:1 reproduction) and **simulated benchmark** (scaled workload generation).
- PII may appear in captured queries — the tool must support filtering/masking.
- Thread simulation fidelity degrades at high scale; benchmark mode accepts this tradeoff.

## Architecture

```
┌─────────────┐    ┌──────────────┐    ┌──────────────┐    ┌────────────┐
│   Capture    │───>│   Workload   │───>│    Replay     │───>│  Reporter  │
│   Agent      │    │   Profile    │    │    Engine     │    │            │
└─────────────┘    │   (storage)  │    └──────────────┘    └────────────┘
                   └──────────────┘
```

- **Capture Agent** — Connects to PG (via `pg_stat_activity` polling, log parsing, or proxy) to record per-connection SQL streams with timing metadata.
- **Workload Profile** — Serialized representation of captured workload: queries, connection/thread mapping, timing, dependencies, transaction boundaries.
- **Replay Engine** — Reads a workload profile and replays it against a target PG instance, preserving connection parallelism and timing. Supports replay modes (exact, read-only, scaled).
- **Reporter** — Compares source vs. replay metrics and produces a performance comparison report (per-query latency, throughput, errors, regressions).

## Build & Development

- **Language:** Rust (2021 edition)
- **Build:** `cargo build` (debug) / `cargo build --release`
- **Test all:** `cargo test`
- **Test single file:** `cargo test --test profile_io_test`
- **Test single function:** `cargo test --test profile_io_test test_profile_roundtrip_messagepack`
- **Test lib unit tests:** `cargo test --lib capture::csv_log`
- **Lint:** `cargo clippy`
- **Format:** `cargo fmt`
- **Run:** `cargo run -- <subcommand> [args]`
- **Verbose logging:** `RUST_LOG=debug cargo run -- -v <subcommand>`

### Crate Structure

The project is both a library (`src/lib.rs`) and binary (`src/main.rs`). Integration tests in `tests/` import from the library crate via `use pg_retest::...`. The binary crate handles CLI dispatch only.

Key modules:
- `capture::csv_log` — PG CSV log parser (pluggable backend via `CaptureSource` pattern)
- `capture::masking` — SQL literal masking for PII protection (strings→`$S`, numbers→`$N`)
- `profile` — Core data types (`WorkloadProfile`, `Session`, `Query`) + MessagePack I/O (v2 format with transaction support)
- `replay::session` — Async per-session replay engine (Tokio + tokio-postgres), transaction-aware (auto-rollback on failure)
- `replay::scaling` — Session duplication with staggered offsets for load testing; per-category scaling via `scale_sessions_by_class()`
- `classify` — Workload classification (Analytical/Transactional/Mixed/Bulk) based on read/write ratio, latency, transaction count
- `compare` — Performance comparison logic + terminal/JSON reporting + exit code evaluation
- `compare::capacity` — Scaled replay reporting (throughput QPS, latency percentiles, error rate)
- `config` — TOML pipeline config parsing and validation (`PipelineConfig`, `ThresholdConfig`, etc.)
- `pipeline` — CI/CD pipeline orchestrator (capture → provision → replay → compare → threshold → report)
- `provision` — Docker provisioner via CLI subprocess (start/teardown containers, backup restore)
- `compare::threshold` — Threshold-based pass/fail evaluation (p95, p99, error rate, regression count)
- `compare::junit` — JUnit XML output for CI test result integration
- `transform` — Composable SQL transform pipeline (`SqlTransformer` trait, `TransformPipeline`) for cross-database SQL conversion
- `transform::mysql_to_pg` — MySQL-to-PostgreSQL transform rules (backticks→double quotes, LIMIT rewrite, IFNULL→COALESCE, IF→CASE WHEN, UNIX_TIMESTAMP→EXTRACT)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pg-retest/pg-retest](https://github.com/pg-retest/pg-retest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
