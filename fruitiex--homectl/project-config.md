---
trigger: always_on
description: **homectl** is a home automation platform built as a monorepo with two main packages:
---


# AGENTS.md - AI Agent Guide for homectl

## Project Overview

**homectl** is a home automation platform built as a monorepo with two main packages:

- **`server/`** – Rust backend: core automation engine, HTTP/WebSocket API
- **`ui/`** – Vite/React Router frontend: web interface consuming the server API

The server unifies home automation systems from different brands by assuming control over individual systems, providing a common interface for configuration, advanced scene control, and reliable state management.

## Technology Stack

### Server (Rust)
- **Language**: Rust (edition 2021)
- **Web Framework**: warp
- **Async Runtime**: tokio
- **Database**: SeaORM query builders/migrations with default SQLite (`./homectl.db`), optional PostgreSQL, and fallback startup from JSON backup, legacy TOML, or an empty in-memory runtime
- **Messaging**: MQTT (via rumqttc)
- **Configuration**: TOML (via config + toml crates)
- **TypeScript Bindings**: ts-rs (generates TypeScript types from Rust structs)

### UI (Vite + React Router)
- **Framework**: Vite with React Router
- **Language**: TypeScript
- **UI Components**: React, daisyui, react-daisyui
- **Styling**: TailwindCSS 4
- **State Management**: jotai
- **Animation**: framer-motion
- **Charts**: @visx suite
- **Canvas**: konva, react-konva
- **Package Manager**: pnpm

## Directory Structure

```
/
├── server/                 # Rust backend
│   ├── src/
│   │   ├── main.rs        # Application entry point
│   │   ├── api/           # HTTP/WebSocket API routes
│   │   ├── core/          # Core automation logic
│   │   ├── db/            # SeaORM database layer (default SQLite, optional PostgreSQL, config export/import)
│   │   ├── integrations/  # Home automation system integrations
│   │   ├── types/         # Shared type definitions
│   │   └── utils/         # Utility modules
│   ├── migrations/        # Legacy SQL migrations retained for reference
│   ├── Settings.toml      # Runtime configuration
│   └── Cargo.toml
├── ui/                     # Vite frontend
│   ├── app/               # Route page components
│   │   ├── api/           # API helpers/routes
│   │   ├── config/        # Configuration UI
│   │   ├── dashboard/     # Dashboard views
│   │   ├── groups/        # Group management
│   │   └── map/           # Map visualization
│   ├── bindings/          # Auto-generated TypeScript types (from ts-rs)
│   ├── hooks/             # React hooks
│   ├── lib/               # Shared utilities
│   └── ui/                # Reusable UI components
├── .github/workflows/     # CI/CD pipelines
│   ├── server-ci.yml      # Server build/test/publish
│   ├── ui-ci.yml          # UI build/publish
│   └── release-please.yml # Automated releases
└── flake.nix              # Nix development environment
```

## Key Concepts

### Integrations
Plugins that connect to various home automation systems:
- **mqtt** – Generic MQTT devices with configurable message formats
- **circadian** – Virtual device for circadian rhythm color following
- **cron** – Scheduled actions
- **timer** – Timed actions (e.g., delay motion sensor re-activation)
- **dummy** – Testing/development without physical hardware

### Devices
Individual controllable units (lights, switches, sensors). Each device has:
- `id`, `name`, `integration_id`
- `state` (power, brightness, color, sensor values)
- `capabilities` (color modes, temperature ranges)

### Groups
Collections of devices that can be controlled together. Groups can contain other groups for hierarchical control.

### Scenes
Preset states for groups/devices. Scenes can:
- Set explicit states (power, color, brightness)
- Reference other scenes
- Link to other devices (e.g., circadian rhythm device)

### Routines
Event-driven automation rules with:
- **Rules** – Conditions that must match (sensor values, device states, group states)
- **Actions** – Operations to perform (ActivateScene, CycleScenes, DimAction, IntegrationAction)

### State actor & runtime snapshot
The server uses an actor-model architecture for `AppState`:

- **`AppState`** (`server/src/core/state/mod.rs`) is owned by a single
  tokio task, the **state actor** (`server/src/core/state/actor.rs`).
  There is no `Arc<RwLock<AppState>>`; the actor is the sole writer.
- **`StateHandle`** is the only way for non-actor code to mutate state.
  Use `handle.send_event(event)` for fire-and-forget events, or
  `handle.mutate(|state| Box::pin(async move { ... })).await?` for
  admin writes that need to read back a typed result. Each command runs
  to completion before the next is dequeued.
- **`SnapshotHandle`** is an `Arc<ArcSwap<RuntimeSnapshot>>` published
  after every command. Readers (HTTP handlers, widgets, websockets) use
  `snapshot.load()` — no locking, no `await`. The snapshot carries
  `runtime_config`, `devices`, `flattened_groups`, `flattened_scenes`,
  `routine_statuses`, `ui_state`, and `warming_up`.
- HTTP route builders take `&SnapshotHandle` + `&StateHandle`; they
  never see `AppState` directly. Warp filters `with_snapshot` and
  `with_handle` inject them into handlers.

Implications for contributors:
- For a new read endpoint, extract the data from `snapshot.load()`.
- For a new admin write, add an `AppState` method and call it from the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FruitieX/homectl](https://github.com/FruitieX/homectl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
