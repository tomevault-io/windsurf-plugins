---
trigger: always_on
description: Desktop app for monitoring and controlling GivEnergy solar inverters over local Modbus TCP.
---

# GivEnergy Local

Desktop app for monitoring and controlling GivEnergy solar inverters over local Modbus TCP.

## Stack

- **Frontend**: React 19 + TypeScript + Vite 8 + Tailwind CSS 4 + Zustand + Recharts + React Router 7
- **Backend**: Tauri 2 desktop shell; embedded Axum HTTP/WS server on port **7337**
- **Modbus**: Custom Rust TCP client to GivEnergy data adapter (port **8899**)
- **Testing**: Rust unit tests only (no frontend tests, no integration tests)

## Prerequisites

- **Node.js** + npm
- **Rust** toolchain (`rustup`)
- **Tauri CLI**: `cargo install tauri-cli`

## Commands

| Command | Action |
|---|---|
| `npm run dev` | Vite dev server on port 5173 |
| `npm run build` | `tsc -b && vite build` (full typecheck + bundle) |
| `npm run lint` | `eslint .` |
| `npm run preview` | `vite preview` |
| `cargo test` (in `src-tauri/`) | Run all Rust unit tests (101 tests) |
| `cargo tauri dev` | Dev mode with Tauri window + Vite + hot-reload |
| `cargo tauri build` | Production build of the desktop app |

Order for full verification: `npm run lint` → `npm run build` (typechecks) → `cargo test` in `src-tauri/`.

## Architecture

### Frontend (`src/`)

React app. Entrypoint: `src/main.tsx`.

- **Pages**: `StatusPage` (dashboard + energy flow), `BatteryPage` (cell-level detail), `HistoryPage` (charts), `ControlPage` (schedules, modes, limits), `SettingsPage` (connection config, connected clients, developer mode, about), `LogsPage` (developer console — only visible when developer mode is enabled)
- **Components**: `EnergyFlowDiagram` (radial SVG power flow), `BatteryPanel` (per-module cell data), `SummaryTiles` (power stats)
- **Hooks**: `useWebSocket` — connects to `/ws`, reconnects on drop, fetches initial snapshot via REST
- **Lib**: `api.ts` (fetch helpers), `format.ts` (power/voltage/temp formatters), `types.ts` (InverterSnapshot etc.)
- **State**: Zustand store (`useInverterStore`) holds `snapshot`, `connectionState`, `connectedHost`, `developerMode` (persisted to localStorage)
- **Version**: Injected at build time via `__APP_VERSION__` (defined in `vite.config.ts`, declared in `src/env.d.ts`)

Frontend talks exclusively to the local Axum server — never directly to the inverter.

### Backend (`src-tauri/src/`)

- **`lib.rs`** — Tauri app setup + headless CLI entry; spawns Axum server (port 7337) + Modbus polling loop
- **`history/`** — SQLite-backed history storage (`~/.givenergy-local/history.db`)
  - `mod.rs` — `HistoryDb` wrapper, schema migration, `insert_reading()`, aggregated `query_history()` with time-bucket AVG (or MAX for cumulative fields)
- **`inverter/`** — data model, register decode/encode, discovery, poll loop
  - `model.rs` — `InverterSnapshot`, `ScheduleSlot`, `BatteryMode`, `BatteryState`
  - `decoder.rs` — converts raw register blocks into `InverterSnapshot`; applies global `enable_charge`/`enable_discharge` flags to slot states
  - `encoder.rs` — translates `ControlCommand` enum into `RegisterWrite` lists (whitelist-validated)
  - `poll.rs` — main polling loop: drain pending writes → read registers → sanitize → broadcast snapshot; uses `Notify` for immediate write execution; warmup reads and grace period after connect
  - `discovery.rs` — network scanning with GivEnergy Modbus protocol verification (sends a read request and validates the 0x5959 magic header in the response)
- **`modbus/`** — GivEnergy Modbus TCP protocol
  - `client.rs` — `ModbusClient`: connect, read registers, write single register (FC6), stale frame drain
  - `framer.rs` — proprietary frame encode/decode (MBAP header + transparent sub-frame + CRC); response CRC validation is lenient (logged, not rejected)
  - `registers.rs` — register addresses, poll block definitions, safe-write whitelist, HHMM encode/decode
- **`server/`** — Axum HTTP layer
  - `api.rs` — REST endpoints for control commands; queues writes to `AppState::pending_writes` and notifies poll loop
  - `ws.rs` — WebSocket endpoint streaming `PollMessage` (snapshot or connection state)
  - `logs.rs` — Log ring buffer (`LogRing`) + tracing capture layer + `GET /api/logs` endpoint for developer console
  - `mod.rs` — router setup, server startup (graceful bind failure, no panic)
- **`settings/`** — persisted JSON config (`~/.givenergy-local/settings.json`)

### Shared state (`AppState`)

Central `Arc<Mutex<…>>`-based state shared between poll loop, API handlers, and WebSocket:

- `latest_snapshot` — most recent `InverterSnapshot`
- `connection_state` — `Connected` / `Disconnected`
- `pending_writes` — queue of `Vec<RegisterWrite>` batches from the API
- `write_notify` — `Notify` that wakes the poll loop immediately when writes are queued
- `settings` — live `PollSettings` (host, port, serial, interval)
- `history` — `HistoryDb` for time-series storage
- `log_ring` — `LogRing` (2000-entry ring buffer) of captured log lines for the developer console

## Data sanitization (register corruption defense)

The GivEnergy dongle frequently returns corrupted register values, especially
on the first reads after TCP connect. The sanitizer in `poll.rs` defends against
this with multiple layers:

### Absolute range checks (always active)

Applied on EVERY reading regardless of previous state:

| Field | Range | Notes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psylsph/givenergy-local](https://github.com/psylsph/givenergy-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
