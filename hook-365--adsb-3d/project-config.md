---
trigger: always_on
description: validates/escapes everything it renders into `config.js`, renders the
---

# CLAUDE.md

Orientation for an AI assistant working in this repo. Project-specific only —
no deployment/homelab details (those live in env vars and `.env`, never here).

## What this is

ADS-B 3D — real-time 3D visualization of ADS-B aircraft traffic. A single
nginx Docker image serves a Vite / TypeScript / Three.js frontend and reverse-
proxies a user-supplied ADS-B feeder plus two optional FastAPI services. The
frontend uses **no framework** — DOM and Three.js directly.

Three feature tiers, all optional and off by default: historical playback
(needs `track-service` + TimescaleDB), ACARS messages (needs `acars-service`),
and a VHF voice scanner (needs an external voice-services stack).

## Repo layout

- `frontend/` — Vite + TS + Three.js app (~37 modules under `src/`). Built to
  static assets, served by nginx.
- `track-service/` — FastAPI + asyncpg. Live WebSocket diff stream + a
  historical-data collector writing to TimescaleDB.
- `acars-service/` — FastAPI bridge to an external acarshub TCP JSON feed.
- `nginx/` — reverse-proxy + static-host config templates.
- `Dockerfile` + `entrypoint.sh` — single image. The entrypoint renders
  `config.js` (frontend runtime config) and the nginx config from environment
  variables at container start.
- `docs/` — `VOICE.md`, `REVERSE-PROXY.md`, `PRODUCTION-CHECKLIST.md`.

## Frontend architecture

The defining pattern is **inverted dataflow + a reconciler**:

- `aircraft/store.ts` (`AircraftStore`) is the single source of truth for
  aircraft state.
- `aircraft/reconciler.ts` runs once per frame: it diffs the store against the
  Three.js scene and creates / updates / removes a per-aircraft `Group` (cone,
  ground icon, altitude line, trail, CSS label, rings). **The reconciler owns
  the aircraft scene graph — do not add or remove aircraft objects elsewhere.**

State is shared via **subscribe-pattern singletons**: a module-level singleton
exposing `getX()` / `updateX()` / `subscribeX()` backed by a listener `Set`.
No reactivity library. Examples: `core/settings.ts`, `core/theme.ts`,
`core/time-context.ts`, `core/filter.ts`, `feed/feeds.ts`,
`feed/voice-calls.ts`, `aircraft/acars-store.ts`.

`src/` directories:
- `core/` — `settings`, `theme`, `filter`, `time-context`, `units`, `coords`,
  `url-state`, `config`, `basemaps` (provider availability + attribution),
  `types`.
- `feed/` — data sources: `live`, `historical`, `history`, `acars`, `routes`,
  `feeds` (multi-feed switching), `voice-calls`, `normalize`.
- `aircraft/` — `store`, `reconciler`, `shapes` (vendored tar1090 catalog),
  `shape-geometry` (extrudes silhouettes + merges procedural 3D features),
  `shape-features` (hand/auto-measured feature annotations per shape),
  `fuselage-profiles.json` (generated width profiles; regenerate by
  rasterizing silhouettes, do not hand-edit), `shape-lab` (dev-only tuning
  harness, `npm run dev` + `?shapeLab=1`), `acars-store`.
- `world/` — `scene`, `controls` (Three.js OrbitControls), `tiles` (basemap),
  `labels`, `heatmap` (3D airway-density), `acars-pings` (transient map pings
  at ACARS-reported coordinates).
- `ui/` — DOM panels: `aircraft-list`, `aircraft-detail`, `settings-panel`,
  `time-controls`, `voice-panel`, `acars-panel`, `map-attribution`,
  `feed-selector`, etc.
- `interaction/` — `picking` (raycaster).
- `main.ts` — wires everything together at boot.

### Adding a setting

1. Add the key to `Settings` + `DEFAULTS` in `core/settings.ts`.
2. Add a row to `SETTINGS_SCHEMA` in `ui/settings-panel.ts`.
3. React to it where it matters via `subscribeSettings()`.

Settings persist to `localStorage` and are merged against `DEFAULTS` on load,
so a payload from an older version never drops new keys.

### Adding 3D detail to an aircraft shape

Silhouette markers are the tar1090 planform extruded thin, plus procedural
parts merged into one shared geometry per shape: a lofted fuselage tube
(width profile from `fuselage-profiles.json`), engine nacelles, tail fin,
rotors, and an optional `planformClip` band (used with a raised `tailplane`
on T-tails, since every drawing already contains a body-level stabilizer).

1. Add or edit the shape's entry in `aircraft/shape-features.ts`. All
   fields are fractions of the shape's own viewBox; read positions off the
   drawing (render it with a grid) so parts land on the drawn features.
2. Eyeball in the dev harness (`?shapeLab=1`) or live.
3. The drift-guard test (`tests-unit/shape-features.test.ts`, the one
   vitest file that runs under jsdom) checks catalog existence, fraction
   sanity, engine symmetry, and exact per-part triangle accounting.

Unannotated shapes keep the plain extrusion. The reconciler is untouched by
all of this: geometry stays one `BufferGeometry` per shape name.

### Adding a theme

Themes are `ThemeTokens` objects in `core/theme.ts`. Each defines ~25 base
hex colors; CSS uses `color-mix(in srgb, var(--token) NN%, transparent)` at
the use-site for all opacity variants, so a theme never has to enumerate
every tint. Three.js materials live under `three.*` on the token object
and are bridged in `world/scene.ts` + `aircraft/reconciler.ts` (sky,
range rings, home marker, trail/selection/emergency/ACARS-ping materials)
which subscribe and mutate `.color` in place — no scene rebuild on switch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hook-365/adsb-3d](https://github.com/hook-365/adsb-3d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
