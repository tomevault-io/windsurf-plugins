---
trigger: always_on
description: > **Purpose**: qBitrr orchestrates qBittorrent ↔ Radarr/Sonarr/Lidarr communication, handling torrent health checks, instant imports, smart cleanup, and request automation. This guide ensures AI agents/contributors maintain consistency across Python backend and React frontend.
---

# AGENTS.md – Agent Coding Guide

> **Purpose**: qBitrr orchestrates qBittorrent ↔ Radarr/Sonarr/Lidarr communication, handling torrent health checks, instant imports, smart cleanup, and request automation. This guide ensures AI agents/contributors maintain consistency across Python backend and React frontend.

## Project Overview
- **Language**: Python 3.12+ (backend), TypeScript + React 18 (WebUI)
- **Architecture**: Multi-threaded event loops per Arr instance; Flask/Waitress REST API; Peewee ORM (SQLite)
- **Entry Point**: `qBitrr.main:run` → spawns WebUI, ArrManager loops, auto-update watchers
- **Key Modules**:
  - `qBitrr/main.py` – orchestrates multiprocessing, launches arr managers and WebUI
  - `qBitrr/arss/` – Arr package (split from legacy monolith):
    - `arr_base.py` – `ArrBase` shared torrent pipeline, config, loops, qBit side effects
    - `radarr.py` / `sonarr.py` / `lidarr.py` – `RadarrArr` / `SonarrArr` / `LidarrArr` concretes
    - `factory.py` – section name → concrete Arr class + client builder
    - `arr.py` – compatibility alias (`Arr = ArrBase`)
    - `manager.py` – `ArrManager` orchestration and instance factory wiring
    - `placeholder_arr.py` – `PlaceHolderArr` role subclass for special/qBit categories
    - `torrent_policy.py` – `TorrentPolicyManager` free-space / tracker-sort role worker
    - `torrent_dispatch.py` / `torrent_limits.py` / `torrent_inspect.py` / `torrent_batch.py` – pipeline roles composed into `ArrBase`
    - `qbit_side_effects.py` – shared pause/resume/delete helpers
    - `db_queries.py` / `request_providers.py` – DB search selection and Ombi/Overseerr leaves
    - `db_update_handlers.py` – per-Arr-type DB update leaf functions
    - `arr_shared.py` – shared imports/constants for arss submodules
  - `qBitrr/arr_client.py` – Pyarr v6 client builders and shared JSON types
  - `qBitrr/arr_tracker_index.py` – shared tracker config → derived URI/host sets (`build_tracker_index`, `extract_tracker_host`)
  - `qBitrr/qbit_seeding_config.py` – qBit-managed category seeding settings loader
  - `qBitrr/quality_profile_helpers.py` – shared quality-profile/search-state helpers for db_update paths
  - `qBitrr/config.py` – TOML config parsing, validation, migrations, live-reload getters
  - `qBitrr/gen_config/` – config schema builders, validate/fill, and migrations
  - `qBitrr/config_reload_policy.py` – classifies config key changes into reload strategies (live, qbit_hot, arr preserve/reset DB, full restart)
  - `qBitrr/process_lifecycle.py` – spawn/restart helpers used by `qBitManager`
  - `qBitrr/webui/` – Flask WebUI package (`app`, route registrars, catalog queries) for `/api/*` (token-protected) and `/web/*` (helpers)
  - `qBitrr/ffprobe.py` – media file verification via ffprobe
  - `qBitrr/tables.py` – Peewee models for persistent state (downloads, searches, expiry)
  - `webui/src/` – React dashboard with custom CSS (`styles.css` + Tailwind), Context API, @tanstack/react-table
- **Config**: `~/config/config.toml` (native) or `/config/config.toml` (Docker). Generated on first run via `qBitrr/gen_config/`
- **Logging**: Structured logs in `~/logs/` or `/config/logs`; `Main.log`, `WebUI.log`, per-Arr logs
- **Deployment**: PyPI package (`qBitrr2`), Docker image (`feramance/qbitrr:latest`), or source install

## Build/Lint/Test Workflow

### Python Backend
1. **Create Environment**: `make newenv` (or `python -m venv .venv && source .venv/bin/activate`)
2. **Install Dependencies**: `make syncenv` (installs `.[all]` from setup.cfg, includes dev deps + WebUI build)
3. **Format & Lint**: `make reformat` → runs pre-commit hooks:
   - `black` (99-char line length, py312 target)
   - `isort` (black profile, known_third_party listed in pyproject.toml)
   - `autoflake` (removes unused imports/variables)
   - `pyupgrade` (modernizes syntax to py38+)
   - `check-yaml`, `check-toml`, `check-json`, `detect-private-key`, `end-of-file-fixer`, `trailing-whitespace`, `mixed-line-ending`
4. **Manual Testing**: No pytest suite; test against live qBittorrent + Arr instances or use Docker Compose setup
5. **Version Bump**: Do not bump locally — CI runs `bump2version patch|minor|major|build` on release commits (`MAJOR.MINOR.PATCH-BUILD`)
6. **Build Package**: `python setup.py sdist bdist_wheel` → dist/qBitrr2-*.whl
7. **Docker Build**: `docker build -t feramance/qbitrr:test .` (multi-stage: Node build → Python install)

### TypeScript/React WebUI
1. **Install**: `cd webui && npm ci` (package-lock.json locked to exact versions)
2. **Dev Server**: `npm run dev` → http://localhost:5173 with HMR (Vite)
3. **Lint**: `npm run lint` → ESLint with `@eslint/js`, `typescript-eslint`, `react-hooks`, `react-refresh`
4. **Type Check**: `tsc -b` (tsconfig.app.json + tsconfig.node.json)
5. **Build**: `npm run build` → outputs to `webui/dist/`, copied to `qBitrr/static/` for bundling
6. **Preview**: `npm run preview` → serve production build locally

### CI/CD
- **pre-commit.ci**: Auto-formats PRs, runs weekly autoupdates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Feramance/qBitrr](https://github.com/Feramance/qBitrr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
