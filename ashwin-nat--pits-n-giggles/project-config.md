---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
poetry install

# Run tests
poetry run pytest tests/

# By name pattern (class, method, or keyword)
poetry run pytest tests/ -k "TestWatchDogTimerAsync"
poetry run pytest tests/ -k "test_initial_state_idle"

# Only the serial (IPC/socket/process) suites
poetry run pytest tests/ -m serial

# Only the parallel-safe suites
poetry run pytest tests/ -m "not serial"

# Live API tests (OpenF1 schema checks) — requires network, excluded from default run
poetry run pytest tests/tests_openf1/tests_openf1_integration.py -m openf1 -v -n 0

# Run a single file
poetry run pytest tests/tests_version.py

# Lint
poetry run pylint --rcfile scripts/.pylintrc apps lib

# Build executable
poetry run python scripts/build.py

# Run coverage
poetry run python scripts/coverage_ut.py
poetry run python scripts/coverage_integration_tests.py
```

## Running Apps

Each app is a Python module; the launcher starts all others as subprocesses:

```bash
poetry run python -m apps.launcher          # Main GUI launcher
poetry run python -m apps.backend           # Telemetry server
poetry run python -m apps.hud               # In-game overlay
poetry run python -m apps.broker            # ZeroMQ message broker
poetry run python -m apps.save_viewer       # Post-race session viewer
poetry run python -m apps.dev_tools.telemetry_replayer --file-name example.f1pcap
```

## Architecture

**Pits n' Giggles** is a multi-process F1 telemetry suite. The F1 game broadcasts UDP telemetry; this app captures, parses, analyzes, and displays it via browser dashboards and an in-game overlay.

### Process Model

```
apps/launcher/     — Qt (PySide6) GUI; spawns and monitors all other processes via IPC
apps/backend/      — Core server: receives UDP/TCP from F1 game, runs analysis, serves WebSocket+REST
apps/hud/          — Always-on-top Qt overlay windows for in-game display
apps/broker/       — ZeroMQ pub/sub broker for multi-client telemetry forwarding
apps/save_viewer/  — Quart web server for analyzing saved session JSON files
apps/frontend/     — Vanilla JS/HTML/CSS browser UI (served by backend)
apps/dev_tools/    — Telemetry replayer and packet capture utilities
```

### Backend Layers (`apps/backend/`)

The backend is structured in three layers:

1. **`telemetry_layer/`** — UDP/TCP socket reception, packet parsing (16 F1 packet types), frame gating
2. **`state_mgmt_layer/`** — `SessionState` aggregates all parsed data; runs overtake/collision detection, tyre wear extrapolation, race analysis
3. **`intf_layer/`** — Quart web server + Socket.IO; pushes state updates to browser clients and HUD via WebSocket; exposes REST API

### Shared Library (`lib/`)

Reusable modules consumed by multiple apps:

- **`f1_types/`** — Packet dataclass definitions for F1 2023–2025 seasons (16 packet types)
- **`telemetry_manager/`** — Async UDP/TCP receiver manager and packet parser factory
- **`socket_receiver/`** — Base, UDP, TCP receiver implementations
- **`config/`** — Config loading from `png_config.json`/`app_settings.ini`; Pydantic validation models
- **`ipc/`** — ZeroMQ-based IPC with three patterns: pub/sub (`IpcPubSubBroker`, `IpcPublisherAsync`, `IpcSubscriber*`), req/rep (`IpcServer*`, `IpcClientSync`), and router/dealer (`IpcRouter`, `IpcDealerClient`, `IpcDealerAsync`); also provides `PngAppId` for app identity
- **`race_ctrl/`** — Race control event tracking: pit stops, car damage, tyre/wing changes; per-driver and per-session managers
- **`tyre_wear_extrapolator/`** — Linear regression tyre wear prediction
- **`delta/`** — Lap delta calculations
- **`openf1/`** — Integration with the external OpenF1 API
- **`wdt/`** — Watchdog timer for async task health monitoring (sync and async variants)
- **`web_server/`** — Shared async web server base (`BaseWebServer`) and uvicorn socket helper used by backend and save_viewer
- **`assets_loader/`** — Loads fonts and icons (team logos, tyre compounds) for Qt HUD
- **`event_counter/`** — Rate/count statistics tracking for telemetry performance metrics
- **`track_segment_info/`** — Track segment metadata and per-circuit sector boundary database

### Data Flow

```
F1 Game (UDP/TCP)
  → TelemetryManager (lib/telemetry_manager) parses packets
  → SessionState (state_mgmt_layer) aggregates and runs analysis
  → TelemetryWebServer (intf_layer) broadcasts via Socket.IO
  → Browser dashboard (apps/frontend) + HUD overlay (apps/hud)
```

### Key Files

- `meta/meta.py` — Single source of version truth (`APP_VERSION`, `APP_NAME_SNAKE`)
- `png_config.json` — Runtime config (ports, capture mode, privacy, HUD, HTTPS)
- `scripts/png.spec` — PyInstaller spec (entry point, hidden imports, version injection)
- `pyproject.toml` — Poetry deps; requires Python 3.12–3.14

## Procedures

These files define step-by-step procedures for common dev tasks. Read the relevant file before starting the task.

- `.claude/commands/perf-report.md` — Generate a performance metrics report from a launcher log file. Use when asked to analyse performance, check latency/loss stats, or generate a report from a log.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ashwin-nat/pits-n-giggles](https://github.com/ashwin-nat/pits-n-giggles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
