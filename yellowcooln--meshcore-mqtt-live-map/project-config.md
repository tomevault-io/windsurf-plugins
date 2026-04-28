---
trigger: always_on
description: Current version: `1.8.4` (see `VERSIONS.md`).
---

# Repository Guidelines

Current version: `1.8.4` (see `VERSIONS.md`).

## Project Structure & Module Organization
- `backend/app.py` wires FastAPI routes, MQTT lifecycle, and websocket broadcast flow.
- `backend/config.py` centralizes env configuration.
- `backend/state.py` holds shared in-memory state + dataclasses.
- `backend/decoder.py` contains payload parsing, multibyte MeshCore decoding, and route helpers.
- `backend/los.py` contains LOS math + elevation fetch helpers.
- `backend/history.py` handles 24h route history persistence/cleanup.
- `backend/static/index.html` is the HTML shell + template placeholders.
- `backend/static/styles.css` holds all UI styles.
- `backend/static/app.js` holds all client-side map logic.
- `backend/static/sw.js` is the PWA service worker.
- `backend/requirements.txt` and `backend/Dockerfile` define Python and Node dependencies.
- `docker-compose.yaml` runs the service as `meshmap-live`.
- `data/` stores persisted state (`state.json`), route history (`route_history.jsonl`), role overrides (`device_roles.json`), and optional neighbor overrides (`neighbor_overrides.json`).
- `.env` holds dev runtime settings; `.env.example` mirrors template defaults.
- `VERSION.txt` tracks the current version (now `1.8.4`); append changes in `VERSIONS.md`.

## Build, Test, and Development Commands
- `docker compose up -d --build` rebuilds and restarts the backend (preferred workflow).
- `docker compose logs -f meshmap-live` follows server logs for MQTT activity.
- `curl -s http://localhost:8080/snapshot` checks current device state.
- `curl -s http://localhost:8080/stats` shows ingest/route counters.
- `curl -s http://localhost:8080/debug/last` inspects recent decoded packets.

## Coding Style & Naming Conventions
- Python in `backend/*.py` uses **2-space indentation**; keep it consistent.
- The project enforces an **80-character column limit** for Python code to maintain readability.
- Formatting is handled by `yapf` using the config in `backend/.style.yapf`.
  - To format code manually: `yapf --in-place --recursive --style backend/.style.yapf backend/`
- HTML/CSS/JS in `backend/static/index.html` uses 2 spaces as well.
- Use lowercase, underscore-separated names for Python variables/functions.
- Prefer small helper functions for parsing/normalization; keep logging concise.

## Testing Guidelines
- Run automated tests with `pip install -r requirements-dev.txt && pytest -q`.
- Validate runtime behavior manually with `/snapshot`, `/stats`, and `/debug/last`.

## Commit & Pull Request Guidelines
- No git history is available in this workspace, so there is no established commit convention.
- If you add git later, use short, imperative commit messages and describe behavioral changes in PRs.

## Configuration & Operations
- Most behavior is controlled by `.env` (MQTT host, TLS, topics, TTLs, map start lat/lon/zoom, MQTT online window, default map layer).
- Current dev defaults: `DEVICE_TTL_HOURS=96`, `PATH_TTL_SECONDS=172800`, `MQTT_ONLINE_SECONDS=600`, `ROUTE_TTL_SECONDS=60`, `TRAIL_LEN=0`, `DISTANCE_UNITS=mi`.
- Node size default is `NODE_MARKER_RADIUS` (pixels); users can override via the HUD slider.
- History link size default is `HISTORY_LINK_SCALE`; users can override via the History panel slider.
- Map radius filter: `MAP_RADIUS_KM=0` disables filtering; `.env.example` uses `241.4` km (150mi). Applies to nodes, trails, routes, and history edges.
- `MAP_RADIUS_SHOW=true` draws a debug circle centered on `MAP_START_LAT/LON`.
- Set `TRAIL_LEN=0` to disable trails entirely; the HUD trail hint is removed when trails are off.
- Coverage button only appears when `COVERAGE_API_URL` is set.
- `QR_CODE_BUTTON_ENABLED=true` adds a `Generate QR Code` button to node popups that opens a theme-aware MeshCore-compatible contact QR modal; default is off.
- Geographic filtering defaults to radius mode; polygon mode is optional via `MAP_BOUNDARY_MODE=polygon` and `MAP_BOUNDARY_FILE`.
- Standalone boundary builder: `tools/map-boundary-builder.html` outputs the JSON consumed by `MAP_BOUNDARY_FILE`; hosted copy: [https://yellowcooln.com/map-boundary-builder/](https://yellowcooln.com/map-boundary-builder/).
- Radar country-bounds controls:
  `WEATHER_RADAR_COUNTRY_BOUNDS_ENABLED` and
  `WEATHER_RADAR_COUNTRY_LOOKUP_URL`.
  `WEATHER_RADAR_COUNTRY_LOOKUP_URL` defaults to
  `/weather/radar/country-bounds` and is an HTTP route path, not a
  filesystem directory.
- Weather wind controls:
  `WEATHER_WIND_ENABLED`, `WEATHER_WIND_API_URL`,
  `WEATHER_WIND_GRID_SIZE`, `WEATHER_WIND_REFRESH_SECONDS`.
- LOS curvature controls:
  `LOS_CURVATURE_ENABLED` and `LOS_CURVATURE_FACTOR`.
  When unset, LOS curvature defaults to `true` with factor `1.333333`.
- `DEVICE_COORDS_FILE` points to optional coordinate overrides (default `/data/device_coords.json`).
- `NEIGHBOR_OVERRIDES_FILE` can point at a JSON map/list of neighbor pairs to resolve hash collisions.
- Auto-neighbor overrides are controlled by:
  `AUTO_NEIGHBOR_OVERRIDES_ENABLED`, `AUTO_NEIGHBOR_OVERRIDES_FILE`,
  `AUTO_NEIGHBOR_ACTIVE_DAYS`, `AUTO_NEIGHBOR_MIN_EDGE_COUNT`, and
  `AUTO_NEIGHBOR_REFRESH_SECONDS`.
- Optional custom HUD link appears when `CUSTOM_LINK_URL` is set.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yellowcooln/meshcore-mqtt-live-map](https://github.com/yellowcooln/meshcore-mqtt-live-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
