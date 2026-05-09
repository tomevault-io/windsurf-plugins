---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Dev server** (custom HTTP + WebSocket + Next.js): `npm run dev`
- **Dev Next.js only** (no backend polling): `npm run dev:next`
- **Build**: `npm run build`
- **Start production**: `npm run start`
- **Run all tests**: `npm run test` (vitest)
- **Run single test**: `npx vitest run src/__tests__/coordinates.test.ts`
- **Update ground stations data**: `npm run update-gs` (multi-source: starlinkinsider.com + starlink.sx + PoP list)
- **Type check**: `npx tsc --noEmit` (runs in CI before tests)
- No linter or formatter is configured

## Architecture

Real-time 3D Starlink satellite visualization built on Next.js 16, React Three Fiber, and a custom Node.js server.

### Server (`server.ts`)

Custom HTTP server wrapping Next.js that handles:
- **Dish client** — connects to a Starlink dish at `192.168.100.1:9200` (configurable via `DISH_ADDRESS`) via the `starlink-dish` npm package to poll status and history
- **WebSocket server** (`src/lib/websocket/server.ts`) — broadcasts dish telemetry, handoff events, and event log messages to all connected clients
- **Demo mode** — auto-detected when no dish is reachable; uses mock data via `starlink-dish` package's `useMock()`. Controllable via `DEMO_MODE` env var (`true`/`false`/`auto`) and `/api/mode` endpoint
- **Polling loops** — status (1s), history (5s), PoP detection (10s), traceroute (60s)

### API Routes (`src/app/api/`)

- `/api/tle` — Starlink TLE data from HF dataset `juliensimon/starlink-tle-latest` (CelesTrak fallback, 6h cache)
- `/api/tle-gps` — GPS TLE data from HF dataset `juliensimon/starlink-tle-latest` (CelesTrak fallback, 6h cache)
- `/api/ground-stations` — gateway catalog from HF dataset `juliensimon/starlink-ground-stations` (via `refreshGroundStations()`)
- `/api/pop` — public IP + rDNS PoP city detection (5m cache)
- `/api/isl-log` — ISL route decision log (GET last 100 lines / POST append)
- `/api/mode` — GET/POST runtime demo/live mode switching (handled in `server.ts`, not Next.js)

### Frontend

Single-page app with a full-viewport 3D canvas and HUD overlay:

- **3D Scene** (`src/components/scene/`) — React Three Fiber canvas with two view modes:
  - `SatellitePropagator` — headless SGP4 propagation, always mounted, writes positions to shared Float32Array in satellite-store
  - `ConnectionBeam` — always mounted (drives satellite selection, handovers, az/el updates); visuals hidden in sky mode via `<group visible={false}>`
  - **Space view**: Globe, Satellites (instanced mesh renderer, reads from satellite-store), GpsSatellites, Sun, Moon, DishMarker, GroundStations, Atmosphere, SceneSetup, SatelliteTooltip
  - **Sky view** (`SkyView.tsx` → `src/components/scene/sky/`):
    - `SkyDomeCamera` — Stellarium-style horizon camera, OrbitControls with 360° azimuth and zenith reach
    - `SkyEnvironment` — ground plane, sun-aware sky gradient (day/twilight/night), horizon ring with compass ticks
    - `SkyGrid` — elevation circles at 30°/60°, azimuth lines every 45°, Billboard cardinal labels
    - `SkyConstellations` — 88 IAU constellation stick figures (LineSegments) + Billboard name labels, RA/Dec→Az/El
    - `SkySatellites` — instanced mesh with az/el dome projection, sun shadow coloring (sunlit=bright, shadow=10%)
    - `SkyStars` — ~500 reference stars (Points with shader), RA/Dec→Az/El via GMST, Billboard name labels
    - `SkyBeam` — glow sprites + halo ring on connected satellite, follows handovers
    - `SkyTooltip` — screen-space nearest-neighbor picking for satellites (az/el, shell, sunlit) and stars (name, magnitude)
    - `SkyTrajectory` — ±5min trajectory arc on hover via SGP4 `propagatePosition()`
- **HUD** (`src/components/hud/`):
  - `StatusBar` — connection state, uptime, quality, firmware, GPS
  - `TelemetryPanel` — sparkline charts for ping, DL, UL, SNR
  - `SatelliteInfoPanel` — satellite link info, gateway, PoP, route type (Direct/ISL), latency confidence indicator
  - `HandoffPanel` — titled "Starlink Network": LIVE/TLE/WS indicators, per-shell satellite stats (operational/total/%), gateway counts, handoff tracking, "Fleet Health →" link to `/fleet`
  - `SkyHud` — sky view stats: sun elevation, satellite counts (sunlit/shadow), UTC time, daytime warning
  - `EventLog`, `ViewControls` (Space/Sky toggle, demo/live, rotate, altitude filter, ISL), `ColorLegend`
- **WebSocket client** (`src/components/WebSocketManager.tsx`) — receives messages and dispatches to stores

### State Management

- `src/stores/app-store.ts` — Zustand store for UI state (selected satellite, view mode, demo mode, altitude filter, ISL prediction, demo location)
- `src/stores/telemetry-store.ts` — Zustand store for dish telemetry data and geometric latency (for cross-validation)

### Satellite Propagation

- `src/lib/satellites/propagator.ts` — SGP4 orbital propagation using `satellite.js`, writes positions into Float32Array for instanced rendering. Includes `propagatePosition()` for single-satellite 3D position queries (used by trajectory arcs)
- `src/lib/satellites/tle-fetcher.ts` — fetches TLE data via API routes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juliensimon/starlink-viz](https://github.com/juliensimon/starlink-viz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
