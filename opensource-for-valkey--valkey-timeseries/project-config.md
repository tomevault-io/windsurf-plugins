---
trigger: always_on
description: - Short, focused instructions to help an AI model become productive in this codebase quickly.
---

# AGENTS: Guidance for AI coding agents

Purpose

- Short, focused instructions to help an AI model become productive in this codebase quickly.

Quick start (commands you can run)

- Build + checks:
  `cargo fmt --check && cargo clippy --profile release --all-targets -- -D clippy::all && RUSTFLAGS="-D warnings" cargo build --all --all-targets --release`
- Local dev script (recommended):
    - `SERVER_VERSION=unstable ./build.sh`  # builds module, builds valkey-server, runs unit & integration tests
    - To run ASAN integration pass: `ASAN_BUILD=true SERVER_VERSION=unstable ./build.sh`
    - Run a subset of Python integration tests: `TEST_PATTERN="test_ts_add" SERVER_VERSION=unstable ./build.sh`
- Benchmarks: `cargo bench --features enable-system-alloc` (see Benchmarks below — the feature is mandatory).
- Compression report: `tools/compression_report.sh` (add `--check` to fail on regressions against a saved baseline).
- Latency report: `tools/latency_report.sh`. Wire-payload report: `tools/wire_report.sh` (see Benchmarks below).

Key ENV and behavior (from `./build.sh`)

- `SERVER_VERSION` (required): controls which valkey-server is cloned/built and stored at
  `tests/build/binaries/$SERVER_VERSION/valkey-server`. Defaults to `unstable` if not set, which tracks the latest main or branch.
- `ASAN_BUILD`: when set runs tests with LeakSanitizer checks and fails on leaks.
- `TEST_PATTERN`: passed to pytest `-k` to select tests.
- `MODULE_PATH` exported after build: `target/release/libvalkey_timeseries{.so,.dylib}` depending on OS.

Setup & Environment Notes

- Rust version: The project requires a minimum Rust version of `1.92`.
- Python tests: Integration tests use Python. Dependencies are in `requirements.txt` (or via `uv sync`). The `build.sh`
  script handles this, but if running `pytest` manually, ensure packages are installed.
- Running manually: To manually start a server with the module loaded, run
  `valkey-server --loadmodule ./target/release/libvalkey_timeseries.so` (requires building the module first).

High-level architecture (big picture)

- This is a Valkey module (Rust crate) exposing TS.* commands to the Valkey server via the `valkey_module!` macro (
  `src/lib.rs`).
- Command implementations live in `src/commands/*` and are registered in `src/lib.rs` with a one-to-one mapping to
  Valkey commands. Example:
    - `["TS.ADD", commands::ts_add_cmd, "write deny-oom", 1, 1, 1, "write timeseries"]`
- Time-series core lives under `src/series` (storage, encoding, background tasks, indexes). Index/init helpers:
  `init_croaring_allocator()` and `init_background_tasks()` are invoked from `src/lib.rs`.
  - `src/series/chunks/` implements three encoding formats: **Chimp** (ELF-on-Chimp, default),
    **Gorilla**, **Uncompressed**. The default is controlled by `DEFAULT_CHUNK_ENCODING` in `src/config.rs`.
    Storage encoding is the user's choice; the encoding used for cluster *wire* payloads is a separate, internal policy —
    see "Wire encoding policy" under conventions below.
  - ACL filtering per series: `src/series/acl.rs`.
- Cross-node fanout / clustering patterns: `src/fanout` and `src/commands/*_fanout_command.rs` use the protobuf wire
  contract in `proto/v1/` and explicit fanout registration (`register_fanout_operations`) to implement
  cluster-wide queries.
- Outlier detection: `src/analysis/outliers/` — multiple algorithms (ESD, CUSUM, EWMA, IQR, MAD, modified z-score, RCF
  variants) exposed via the `TS.OUTLIERS` command.
- Aggregation: `src/aggregators/` — aggregation handlers and iterators used by range queries.
- Supporting subsystems (all referenced from `src/lib.rs`):
  - `src/common/` — shared utilities: encoding, logging, thread pool, RDB helpers, string interning.
  - `src/labels/` — `Label` type, label filter evaluation, regex helpers.
  - `src/parser/` — Prometheus-compatible filter syntax, metric name, timestamp, and duration parsing.
  - `src/iterators/` — sample and row iterators consumed by range and multi-range queries.
  - `src/join/` — ASOF join logic backing `TS.JOIN`.

Project-specific conventions and patterns

- All Valkey commands are declared in the `valkey_module!` macro in `src/lib.rs`; change there to add/remove commands.
- Command files follow `ts_<command>.rs` naming and export `ts_<command>_cmd` functions (see `src/commands/mod.rs`).
- Fanout pattern: synchronous local implementation + `*_fanout_command.rs` files which marshal/unmarshal protobuf
  messages for cluster aggregation. Seven operations are currently registered (see `register_fanout_operations` in
  `src/commands/mod.rs`): `LabelStatsFanoutCommand`, `CardFanoutCommand`, `LabelSearchFanoutCommand`,
  `MDelFanoutCommand`, `MGetFanoutCommand`, `MRangeFanoutCommand`, `QueryIndexFanoutCommand`.
- Wire encoding policy: `samples_to_chunk` / `samples_to_chunk_lossless` in `src/series/chunks/serialization.rs` are the
  **single** decision point for how samples are encoded onto the cluster wire. Two tiers, keyed on sample count:
  below `WIRE_COMPRESSION_MIN_SAMPLES` (16) an `UncompressedChunk`, at or above it a `ChimpChunk`. Do not add a third
  tier or hand-roll an encoding at a call site — both were true before and neither survived measurement (see the wire

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensource-for-valkey/valkey-timeseries](https://github.com/opensource-for-valkey/valkey-timeseries) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
