---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flowrs is a Terminal User Interface (TUI) application for Apache Airflow built with Rust and the ratatui library. It allows users to monitor, inspect, and manage Airflow DAGs from the terminal.

## Build and Development Commands

### Requirements
- **Minimum Supported Rust Version (MSRV):** 1.87.0

### Building
- `cargo build`: Build the project in debug mode
- `cargo build --release`: Build optimized release binary
- `make build`: Build release version and copy to `/usr/local/bin/flowrs`

### Running
- `cargo run`: Run the TUI application (equivalent to `flowrs run`)
- `FLOWRS_LOG=debug cargo run`: Run with debug logging enabled
- `make run`: Run with debug logging

### Testing
- `cargo test --workspace`: Run all workspace tests
- `cargo test --workspace --lib --bins`: Run unit tests only (matches CI)
- `cargo test <test_name>`: Run specific test by name
- `cargo test -- --nocapture`: Run tests with output visible

### Linting
- `cargo clippy --workspace --all-targets --all-features -- -D warnings`: Run Clippy (matches CI)
- `cargo fmt --all --check`: Check formatting (matches CI)

## Workspace Structure

The project is a Cargo workspace with three crates:

```
flowrs-airflow  (self-contained Airflow HTTP client library, zero workspace deps)
      ↑
flowrs-config   (TUI configuration: FlowrsConfig, ConfigPaths, TOML parsing)
      ↑
flowrs-tui      (binary: TUI app, view models, traits, UI, commands)
```

### flowrs-airflow (`crates/flowrs-airflow/`)
Self-contained Airflow API client library. Has no dependencies on other workspace crates.
- `src/auth.rs`: Auth types (`AirflowAuth`, `BasicAuth`, `TokenSource`, `MwaaAuth`, `AstronomerAuth`, `ComposerAuth`)
- `src/config.rs`: Server config types (`AirflowConfig`, `AirflowVersion`, `ManagedService`, `GccConfig`)
- `src/client/`: HTTP client layer
  - `base.rs`: `BaseClient` wrapping reqwest with auth
  - `auth/`: `AuthProvider` trait and implementations (basic, token, managed service providers)
  - `v1/`: Airflow v2 API client (`V1Client`, uses `/api/v1`), with response models in `v1/model/`
  - `v2/`: Airflow v3 API client (`V2Client`, uses `/api/v2`), with response models in `v2/model/`
- `src/managed_services/`: Managed service discovery (Conveyor, MWAA, Astronomer, Cloud Composer), feature-gated
  - `expand.rs`: `expand_managed_services()` takes `ManagedServiceConfig`, returns discovered `AirflowConfig`s

### flowrs-config (`crates/flowrs-config/`)
TUI-specific configuration management. Depends on `flowrs-airflow` for auth/server types (re-exports them).
- `src/lib.rs`: `FlowrsConfig` struct (servers, managed_services, poll_interval, etc.), TOML parsing/writing
- `src/paths.rs`: `ConfigPaths` for XDG-compliant config file resolution
- `src/auth.rs`: Re-exports auth types from `flowrs-airflow`
- `src/server.rs`: Re-exports server config types from `flowrs-airflow`

### flowrs-tui (root crate, `src/`)
The TUI binary. Depends on both `flowrs-airflow` and `flowrs-config`.
- `src/airflow/`: Airflow integration layer (view models, traits, client wrapper)
  - `client.rs`: `FlowrsClient` enum wrapping `V1Client`/`V2Client`, implements all TUI traits, contains From conversions and URL building
  - `model/`: Domain/view model types (`Dag`, `DagRun`, `TaskInstance`, `Log`, `Task`, `DagStatistic`, `GanttData`, `OpenItem`, newtype IDs, duration utils)
  - `traits/`: Async operation traits (`AirflowClient`, `DagOperations`, `DagRunOperations`, `TaskInstanceOperations`, `LogOperations`, `DagStatsOperations`, `TaskOperations`)
  - `graph.rs`: `TaskGraph` for topological sorting of task instances
- `src/app.rs`: Main event loop
- `src/app/worker/`: Async worker processing `WorkerMessage`s via mpsc channel
- `src/app/model/`: Panel models implementing the `Model` trait
- `src/app/model/popup/`: Modal popup interactions
- `src/ui/`: UI rendering (ratatui widgets, gantt charts, theming)
- `src/commands/`: CLI subcommands (run, config add/list/remove/update/enable)

## Configuration

Flowrs stores configuration in TOML format, following the XDG Base Directory Specification:
- **Primary (XDG):** `$XDG_CONFIG_HOME/flowrs/config.toml` (defaults to `~/.config/flowrs/config.toml`)
- **Legacy fallback:** `~/.flowrs` (read if XDG path doesn't exist)

Config paths are managed via `CONFIG_PATHS` static in `src/main.rs`, which uses `ConfigPaths` from `crates/flowrs-config/src/paths.rs`. Writes always go to the XDG path; reads check XDG first, then legacy. A warning popup is shown if both files exist.

Configuration structure:
- `servers`: Array of Airflow server configurations
- `managed_services`: Array of managed service integrations (Conveyor, MWAA, Astronomer, GCC)
- `active_server`: Name of currently active server
- `poll_interval_ms`: API poll interval in milliseconds (default 2000, minimum 500)

## Architecture

### Event Loop (src/app.rs)
1. Draw UI via `draw_ui()`
2. Wait for events (keyboard input or tick)
3. Route events to active panel's `update()` method
4. Process returned `WorkerMessage`s by sending to worker channel
5. Handle global events (quit, panel navigation)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvanbuel/flowrs](https://github.com/jvanbuel/flowrs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
