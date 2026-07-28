---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

Use `nix develop` to enter the development shell before running most commands. The Nix shell provides InfluxDB, Telegraf, Nomad, Holochain, and other tooling.

For one-off commands that need these tools, use `nix develop -c <command>` to run a single command in the Nix environment without entering an interactive shell.

```bash
# Build default workspace members (framework, bindings, summariser, happ_builder)
cargo build

# Run all tests
cargo test --workspace --all-targets

# Run tests for a single crate
cargo test -p holochain_summariser

# Lint (must pass with no warnings)
cargo clippy --workspace --all-targets --all-features -- --deny warnings

# Format Rust
cargo fmt --all

# Format TOML
taplo format

# All static checks (see scripts/checks.sh for full list)
nix develop -c bash -c "source scripts/checks.sh && check_all"
```

Scenarios and zomes are **excluded from the default workspace members** and must be built explicitly:

```bash
# Run a scenario locally
RUST_LOG=info cargo run -p zome_call_single_value -- --duration 60

# Smoke-test a scenario through Nix (as CI does)
nix run .#rust-smoke-test -- --package zome_call_single_value -- --duration 5 --no-progress
```

## Project Architecture

### Framework (`framework/`)

Generic load-testing infrastructure with no Holochain-specific code:

- `wind_tunnel_core` — shared types and traits
- `wind_tunnel_instruments` / `wind_tunnel_instruments_derive` — metrics collection and procedural macros
- `wind_tunnel_runner` — scenario execution engine (agents, hooks, CLI)
- `wind_tunnel_summary_model` — `RunSummary` and related serializable types read from `run_summary.jsonl`

### Bindings (`bindings/`)

Adapt the framework to specific systems:

- `holochain_client_instrumented` / `holochain_wind_tunnel_runner` — Holochain bindings
- `kitsune_client_instrumented` / `kitsune_wind_tunnel_runner` — Kitsune bindings

### Scenarios (`scenarios/`)

Each scenario is a standalone binary using `holochain_wind_tunnel_runner` (or `kitsune_wind_tunnel_runner`). A `ScenarioDefinitionBuilder` wires up global setup/teardown hooks, per-agent setup/teardown hooks, and one or more agent behaviour functions. Scenarios are not in `default-members` and are built on demand.

Scenarios that require custom zomes reference a `build = "../scenario_build.rs"` build script and declare `[package.metadata.required-dna]` / `[package.metadata.required-happ]` / `[package.metadata.fetch-required-happ]` sections in their `Cargo.toml` to build zomes and package them into hApps at build time.

Common functionality available for scenarios:
- Scenarios which use an instrumented client like `holochain_client_instrumented` will automatically record metrics for client calls. Custom metrics can also be recorded by getting a `Reporter` from the scenario context.
- Setup/teardown hooks can be used to perform common tasks before or after the scenario. Use agent setup/teardown hooks for tasks that only apply to the current agent.
- The scenario can check whether the framework is trying to shut down to break out of retry loops or stop other long-running work.
- Named behaviors allow a scenario to be comprised of multiple agents, behaving differently while interacting with each other.
- Report which environment variables affect scenario behavior. This *must* be used for any variable that changes the scenario's behavior, otherwise the summariser can't recognize different configurations of the same scenario.

### Shared Scenario Libraries (`scenarios_common/`)

Reusable library crates shared across multiple scenario binaries. Each subdirectory is a Rust library (not a standalone binary) providing common helpers for a family of related scenarios.

- `unyt_scenario` (`wind_tunnel_unyt_scenario`) — shared infrastructure for the Unyt scenarios (`unyt_chain_transaction`, `unyt_chain_transaction_zero_arc`), including network initialization, agent setup, durable object communication, and behaviour logic.

### Zomes (`zomes/`)

Holochain coordinator/integrity zome pairs. Each zome uses `build = "../../wasm_build.rs"`. Coordinator and integrity zomes are separate Rust projects.

### Summariser (`summariser/`)

A utility tool (`holochain-summariser`) that queries InfluxDB for a completed run, produces structured JSON summaries, and writes a report file.

**Data flow:**
1. Reads `run_summary.jsonl` (or `$RUN_SUMMARY_PATH`) to find runs — `RunSummary` contains the key fields: `run_id`, `scenario_name`, `started_at`, `run_duration`, `fingerprint`.
2. Dispatches to a per-scenario `summarize_*` function (registered in `lib.rs`) which queries InfluxDB.
3. InfluxDB responses are converted to Polars `DataFrame`s via `frame.rs`.
4. Analysis functions in `analyze.rs` compute statistics.
5. Results are combined into `SummaryOutput` and serialised as JSON.

### Test Data for Summariser

The summariser has two Cargo features to enable snapshot testing:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [holochain/wind-tunnel](https://github.com/holochain/wind-tunnel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
