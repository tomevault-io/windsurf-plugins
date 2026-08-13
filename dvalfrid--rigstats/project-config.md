---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Skills (invoke with /name)

| Skill | When to use |
|---|---|
| `/commit-workflow` | Opening an issue, committing, roadmap sync |
| `/sensor-fixture` | Adding a hardware sensor fixture |
| `/run-rigstats` | Building and launching the app |
| `/verifier-gui` | Visually verifying a GUI change |

## Commands

```powershell
# Build egui binary (debug)
cargo build --manifest-path src-egui/Cargo.toml

# Restart the app (kill by PID — name-based kill silently fails; verify timestamp before launch)
$proc = Get-Process rigstats -ErrorAction SilentlyContinue
if ($proc) { Stop-Process -Id $proc.Id -Force }
cargo build --manifest-path src-egui/Cargo.toml
(Get-Item .\target\debug\rigstats.exe).LastWriteTime   # must have advanced
Start-Process .\target\debug\rigstats.exe

# Check egui + backend for errors
cargo check --manifest-path src-egui/Cargo.toml

# Build sensor sidecar (requires .NET 10 SDK)
dotnet build sensor-sidecar/sensor-sidecar.csproj

# Run Rust tests
cargo xtask test

# Full verification (sidecar + tests + clippy + fmt check)
cargo xtask verify

# Production build
cargo xtask build

# Setup (install git hooks, first-time only)
cargo xtask setup
```

> `cargo xtask verify` / `cargo xtask build` fail if the `rigstats-sensor` service is running (it holds the exe). Stop it first: `sc.exe stop rigstats-sensor` (elevated terminal).

Single test: `cargo test --manifest-path rigstats-backend/Cargo.toml <test_name>`

## Linting and formatting

```bash
cargo xtask fmt          # format Rust (modifies files)
cargo xtask fmt-check    # CI — no modifications
cargo xtask clippy       # must pass with zero warnings (-D warnings)
```

See [STANDARDS.md](STANDARDS.md) for the full code standards.

## After making code changes

**Always run the relevant checks before declaring a task complete.**

| Changed | Run |
| --- | --- |
| Any Rust file | `cargo xtask fmt` then `cargo xtask clippy` |
| Any `sensor-sidecar/*.cs` file | `dotnet build sensor-sidecar/sensor-sidecar.csproj` |
| Logic in Rust | `cargo xtask test` |
| Unsure | `cargo xtask verify` |

- `clippy` is `-D warnings` — zero warnings required.
- If `fmt` modifies files, include those changes in the same commit.
- Never add `#[allow(...)]` without a clear reason documented in the code.

## Commit message format

Mandatory — `release-please` parses commit subjects to generate `CHANGELOG.md` and bump the version.

```
<type>(<scope>): <subject>

<optional body>

Closes #N
```

- **type:** `feat`, `fix`, `perf`, `docs`, `refactor`, `test`, `build`, `chore`, `style`. Only `feat`/`fix`/`perf` surface in the changelog.
- **scope:** lower-case area, e.g. `cpu`, `gpu`, `settings`, `wallpaper`, `status`. Optional but expected.
- **subject:** imperative, lower-case start, no trailing period.
- Breaking change: `feat!:` / `fix!:` or `BREAKING CHANGE:` footer.
- Always include `Closes #N` so GitHub closes the issue automatically on push to main.

For the full issue → implement → test → commit → roadmap-sync workflow, run `/commit-workflow`.

## Design philosophy

Prefer the simplest solution that solves the problem. Before implementing, ask: is there a direct approach that avoids the complexity entirely? Flag files, shared state, and extra IPC are often signs that a simpler path exists. Question existing plans — a plan being written down is not a reason to follow it if a cleaner alternative is obvious.

## Architecture Overview

Windows-only egui desktop app ("RIGStats") displaying hardware telemetry on a secondary monitor (portrait or landscape), as a floating overlay, or reparented into the desktop wallpaper (WorkerW). No web frontend — all UI is native Rust/egui.

Cargo workspace with two members:

| Crate | Path | Role |
|---|---|---|
| `rigstats-backend` | `rigstats-backend/` | Shared lib — telemetry, hardware detection, settings, logging |
| `rigstats-egui` | `src-egui/` | egui library (`lib.rs`) + two binaries: `rigstats` (main app) and `rigstats-wallpaper` (WorkerW host). Both embed `dashboard::DashboardRuntime` and render via `dashboard::DashboardView`. |

Settings are read from `%APPDATA%\se.codeby.rigstats\`. The sidecar pipe accepts one client at a time — in wallpaper mode only the host polls; the main app pauses its `poll_loop` via `poll_paused`.

### egui binary (`src-egui/src/`)

- **`lib.rs`** — library root; re-exports all modules so both binaries share the same panel renderer
- **`main.rs`** — `RigStatsApp`/`eframe::App`: frame loop, settings reload, secondary viewports, panel rendering, wallpaper-mode supervisor (`update_wallpaper_mode`)
- **`dashboard.rs`** — `DashboardRuntime`: owned telemetry→renderer glue (sparklines, theme, thresholds, textures, `drain`/`apply_settings`/`view`); `DashboardView<'a>`: borrowed per-frame render state; `PanelThresholds` (warn/crit pairs)
- **`bin/wallpaper.rs`** — `rigstats-wallpaper` host: attaches into WorkerW, runs own `poll_loop`, exits when parent PID disappears
- **`geometry.rs`** — `profile_to_size`, monitor enumeration, pinned/auto-target position resolution; bulk of the unit tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvalfrid/rigstats](https://github.com/dvalfrid/rigstats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
