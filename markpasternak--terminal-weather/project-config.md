---
trigger: always_on
description: Agent instructions for `terminal-weather`, an animated Rust terminal weather app with both an interactive TUI and a `--one-shot` CLI mode.
---

# AGENTS.md

Agent instructions for `terminal-weather`, an animated Rust terminal weather app with both an interactive TUI and a `--one-shot` CLI mode.

---

## Project Overview

This is a single-binary Rust 2024 project built around:

- `ratatui` + `crossterm` for terminal rendering and input
- `tokio` for async tasks and timers
- `reqwest` + `serde` for Open-Meteo, geocoding, geo-IP, and update-check HTTP calls
- `clap` for CLI parsing
- `anyhow` + `thiserror` for error handling
- Test tooling: `insta`, `proptest`, `wiremock`
- Optional local analysis tooling: `jq`, `rust-code-analysis-cli` `0.0.25`, `cargo-dupes` `0.2.1`, `cargo-llvm-cov` `0.8.5`

Two runtime paths matter:

- Interactive TUI: launched normally, with async event loop, rendering, overlays, settings persistence, and background update checks
- One-shot mode: `--one-shot`, which resolves a location, fetches a forecast, prints to stdout, and exits without entering the TUI

---

## Commands

Run this before calling work complete:

```bash
./scripts/check.sh
```

Useful variants:

```bash
./scripts/check.sh --verbose
cargo check --all-targets --all-features
cargo test --all --all-features
cargo run -- Stockholm
cargo run -- --demo
cargo run -- --one-shot Stockholm
```

Install optional tooling once for full local coverage:

```bash
cargo install --locked rust-code-analysis-cli --version 0.0.25
cargo install --locked cargo-dupes --version 0.2.1
cargo install --locked cargo-llvm-cov --version 0.8.5
# install jq via brew/apt/etc.
```

`./scripts/check.sh` classifications:

| Step | Classification |
|------|---------------|
| `cargo fmt --all -- --check` | REQUIRED |
| `cargo clippy --all-targets --all-features -- -D warnings` | REQUIRED |
| `cargo clippy --all-targets --all-features -- -D warnings -D clippy::pedantic ...` | REQUIRED |
| `scripts/complexity-gate.sh` | REQUIRED |
| `cargo check --all-targets --all-features` | REQUIRED |
| `cargo test --all --all-features` | REQUIRED |
| `cargo build --release` | REQUIRED |
| `scripts/duplication-gate.sh` | RECOMMENDED |
| `scripts/complexity-audit.sh` | RECOMMENDED |
| `scripts/coverage.sh` | RECOMMENDED |
| `scripts/file-length-audit.sh` | RECOMMENDED |

Checks that require optional tooling auto-skip with install hints.

Current local thresholds:

- Complexity gate: cyclomatic `< 20`, cognitive `< 30`, MI `>= 30`
- Complexity audit: file NLOC medium `> 500`, critical `> 1000`; function NLOC medium `> 50`, critical `> 100`
- Duplication gate: advisory by default unless `TW_DUPES_ENFORCE=1`
- Coverage gate: line `>= 85%`, function `>= 85%`, branch `>= 75%`
- File-length audit: warns on Rust files over `500` lines

Update snapshots only when UI changes are intentional:

```bash
INSTA_UPDATE=always cargo test --all --all-features
```

---

## Source Map

Prefer these as anchors when exploring:

```text
src/
  main.rs                 # CLI entrypoint
  lib.rs                  # interactive runtime + one-shot path
  cli.rs                  # clap definitions and validation
  update.rs               # background Homebrew update-check logic
  app/
    events.rs             # AppEvent enum, input/timer/demo tasks
    settings.rs           # persisted settings and config path resolution
    state.rs              # AppState/AppMode and shared state
    state/                # async, fetch, input, and UI state handlers
  data/
    forecast.rs           # Open-Meteo forecast + AQI client
    geocode.rs            # forward/reverse geocoding
    geoip.rs              # IP-based location lookup
  domain/
    weather.rs            # weather domain surface, re-exports
    weather/              # types, conversions, insights, tests
    alerts.rs             # alert domain types
  resilience/
    backoff.rs            # retry scheduling
    freshness.rs          # stale/offline freshness semantics
  ui/
    mod.rs                # render entrypoint and overlays
    animation.rs          # motion tiers and transitions
    layout.rs             # responsive layout rules
    particles.rs          # particle engine
    narrative.rs          # weather narrative text
    theme/                # palette/capability resolution
    widgets/              # hero, hourly, daily, alerts, help, settings, pickers, landmarks

tests/
  render_snapshots.rs
  flows.rs
  freshness_integration.rs
  geocode_ambiguity.rs
  property_range_bar.rs
  caching.rs

scripts/
  check.sh
  complexity-gate.sh
  complexity-audit.sh
  duplication-gate.sh
  coverage.sh
  file-length-audit.sh
```

Do not mirror this tree mechanically in new docs or comments. It changes often.

---

## Architecture Contracts

State machine:

- `Loading` -> `SelectingLocation` -> `Ready` -> `Error` -> `Quit`

Core event model:

- `Bootstrap`
- `TickFrame`
- `TickRefresh`
- `ForceRedraw`
- `Input`
- `FetchStarted`
- `GeocodeResolved`
- `FetchSucceeded`
- `FetchFailed`
- `UpdateCheckFinished`
- `Demo`
- `Quit`

Important boundaries:

- `src/data/` owns HTTP DTO parsing and external API interactions
- `src/domain/` owns app-facing weather and alert types
- `src/ui/` should consume domain/app state, not raw API DTOs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markpasternak/terminal-weather](https://github.com/markpasternak/terminal-weather) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
