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
| `cargo clippy` (in `src-tauri/`) | Run Rust linter |
| `cargo tauri dev` | Dev mode with Tauri window + Vite + hot-reload |
| `cargo tauri build` | Production build of the desktop app |

Order for full verification: `cargo clippy` → `npm run lint` → `npm run lint:md` → `npm run build` (typechecks) → `cargo test` in `src-tauri/`.

## Linting rules

### Rust (clippy)

All clippy warnings must be fixed before committing. Known patterns that commonly trigger warnings:

- `empty_line_after_doc_comments` — no blank line after `///` doc comments
- `field_reassign_with_default` — use `Struct { field: value, ..Default::default() }` instead of mutating after default
- `manual_flatten` — use `.flatten()` / `.into_iter().flatten()` instead of `if let Some` in a loop
- `match_like_matches_macro` — use `matches!()` for boolean match expressions
- `derivable_impls` — use `#[derive(Default)]` instead of manual `impl Default`
- `new_without_default` — add `impl Default` that calls `new()`
- `same_item_push` — use `vec![val; N]` or `resize(N, val)` instead of loop + push
- `manual_clamp` — use `.clamp(min, max)` instead of `.min(max).max(min)`

Run: `cd src-tauri && cargo clippy`

### TypeScript / ESLint

All ESLint errors must be fixed before committing. Notable rules:

- `verbatimModuleSyntax: true` — use `import type` for type-only imports
- `erasableSyntaxOnly: true` — no `enum`, no `namespace`, no constructor parameter properties
- `noUnusedLocals` / `noUnusedParameters` — both on; declarations must be used
- `react-hooks/set-state-in-effect` — do not call `setState` directly inside `useEffect`; use derived values or key-based remounting instead

Run: `npm run lint`

### Markdown

Run `markdownlint` on .md files after significant edits. Notable rules:

- MD001 — heading levels should only increment by one level at a time
- MD012 — no multiple consecutive blank lines
- MD022 — headings should be surrounded by blank lines
- MD032 — lists should be surrounded by blank lines

Run: `npx markdownlint '**/*.md' --ignore node_modules`

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

- **`lib.rs`** — Tauri app setup + headless CLI entry; spawns Axum server (configurable port, default 7337) + Modbus polling loop
- **`history/`** — SQLite-backed history storage (`~/.givenergy-local/history.db`)
  - `mod.rs` — `HistoryDb` wrapper, schema migration, `insert_reading()`, aggregated `query_history()` with time-bucket AVG (or MAX for cumulative fields)
- **`inverter/`** — data model, register decode/encode, discovery, poll loop
  - `model.rs` — `InverterSnapshot`, `ScheduleSlot`, `BatteryMode`, `BatteryState`
  - `decoder.rs` — converts raw register blocks into `InverterSnapshot`; applies global `enable_charge`/`enable_discharge` flags to slot states
  - `encoder.rs` — translates `ControlCommand` enum into `RegisterWrite` lists (whitelist-validated)
  - `poll.rs` — main polling loop: drain pending writes → read registers → sanitize → broadcast snapshot; uses `Notify` for immediate write execution; warmup reads and grace period after connect
  - `discovery.rs` — network scanning with GivEnergy Modbus protocol verification (sends a read request and validates the 0x5959 magic header in the response)
- **`modbus/`** — GivEnergy Modbus TCP protocol

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psylsph/home-energy-manager](https://github.com/psylsph/home-energy-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
