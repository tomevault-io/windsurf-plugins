---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Vision

**gonogo** is a mission control SPA for Kerbal Space Program. It operates in two modes within the same app:

- **Main screen** (`/`) — connects to KSP data sources, hosts a live telemetry dashboard, and distributes data to connected station screens via PeerJS.
- **Station screen** (`/station`) — a peer-connected dashboard whose layout and role are stored in `localStorage`. Stations can optionally pull a saved config from the main screen. There are no per-station routes; each device at `/station` is its own independent station.

The defining feature is a **context-aware, extensible dashboard component system**: components self-register into a global registry, and the dashboard orchestrator renders whatever is registered. External packages (not in this repo) can add components and themes using the same API as the built-in library.

---

## Monorepo Structure

```
packages/
  core/       — Plugin registry, shared TS types, React contexts, GO/NO-GO system
  components/ — Built-in dashboard component library (uses core registry)
  app/        — Vite + React SPA (main screen + station mode)
  proxy/      — Lightweight Fastify server: kOS telnet → WebSocket bridge
```

**Tooling:** pnpm workspaces + Turborepo. Package names use the `@gonogo/` scope.

---

## Commands

```bash
pnpm install          # install all workspace dependencies
pnpm dev              # run app (Vite) and proxy server in parallel
pnpm build            # build all packages via Turborepo
pnpm test             # run tests across all packages
pnpm lint             # lint all packages
pnpm --filter @gonogo/app dev       # run only the SPA
pnpm --filter @gonogo/proxy dev     # run only the proxy server
pnpm --filter @gonogo/core test     # test a single package
```

---

## Architecture

### Data Flow

```
KSP (Telemachus Reborn HTTP/WS) ──→ Main screen (direct, React Query)
KSP (kOS via telnet)            ──→ @gonogo/proxy (Fastify + telnet client)
                                         └──→ Main screen (WebSocket)
Main screen ←──→ Station screens (PeerJS data channels, via peerjs.com broker)
```

Telemachus Reborn is a standard HTTP/WebSocket API — the browser talks to it directly. The proxy server is **only required for kOS integration**; without it, all other features still work. The app must display proxy connection status prominently in the UI.

### `@gonogo/core`

The foundation for everything extensible:

- **Plugin registry** — `registerComponent(def)`, `registerTheme(def)`, and `registerDataSource(def)` are the three extension points. Calling these at module load time is all that's needed to extend the app.
- **Shared TypeScript types** — `ComponentDefinition`, `ThemeDefinition`, `DataSourceDefinition`, `StationConfig`, `DataRequirement`, `Behavior`, etc.
- **React contexts** — `DashboardContext` (current layout, orchestrator state), `PeerContext` (PeerJS connection state), `StationContext` (station identity/role from localStorage).
- **GO/NO-GO system** — aggregates GO/NO-GO state across all active stations. A component can declare `behaviors: ['gonogo-participant']` in its definition to contribute to the global state.
- **Data source interface (repository pattern)** — all data sources implement a common `DataSource` interface:
  ```ts
  interface DataSource {
    id: string;
    name: string;
    connect(): Promise<void>;
    disconnect(): void;
    status: DataSourceStatus; // 'connected' | 'disconnected' | 'error'
    schema(): DataKey[];                                      // available keys
    subscribe(key: string, cb: (value: unknown) => void): () => void;
  }
  ```
  The orchestrator resolves component `dataRequirements` against registered sources at runtime. Components declare keys using a normalised format (e.g. `'vessel.altitude'`); each `DataSource` implementation maps its own API response shape onto those keys. This means components are data-source-agnostic.

### `@gonogo/components`

The built-in component library. Each component file calls `registerComponent()` on import — there is no central index that manually lists them; the orchestrator just needs to import the package and registration happens automatically.

Components declare their `dataRequirements` (e.g. `['vessel.altitude']`) so the orchestrator knows what data to subscribe to. The data layer resolves requirements against registered data sources.

Components are styled with **styled-components**. Component names and styled sub-components follow BEM-inspired naming for readability (e.g. `AltitudeGauge`, `AltitudeGauge__Label`, `AltitudeGauge__Value`).

### `@gonogo/app`

The Vite SPA. Key responsibilities:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonpepler) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
