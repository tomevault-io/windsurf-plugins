---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

GeoLibre is a single **npm workspaces monorepo** (`apps/*`, `packages/*`, `workers/*`) plus two non-npm components: a Python FastAPI sidecar (`backend/geolibre_server`) and a separate Python package (`python/`, the `geolibre` Jupyter anywidget). One `npm install` at the root wires up every JS workspace. Use **npm** (the repo tracks `package-lock.json`), Node **22+**.

The same React app ships three ways: native desktop via **Tauri v2** (`apps/geolibre-desktop/src-tauri`), a browser web build served by nginx (Docker), and embedded in Jupyter (the `python/` package bundles a build of the web app into its wheel).

## Commands

```bash
npm run dev            # web dev server → http://localhost:5173
npm run tauri:dev      # desktop app (required for filesystem dialogs, local MBTiles, local raster reads)
npm run build          # production web build → apps/geolibre-desktop/dist/
npm run tauri:build    # desktop installers → apps/geolibre-desktop/src-tauri/target/release/bundle/
npm run typecheck      # alias for the full build (tsc -b && vite build) — writes to dist/, not a pure type-check
npm run ci             # full local gate: build + frontend + worker + backend + rust check
```

Tests:

```bash
npm run test:frontend                              # node --test over tests/*.test.ts (tsx loader)
npm run test:frontend:coverage                     # same, plus a per-file coverage summary (Node built-in)
node --import tsx --test tests/<name>.test.ts      # a single frontend test file
npm run test:backend                               # pytest backend/geolibre_server/tests
npm run test:backend:coverage                      # same, plus a pytest-cov term-missing report
python -m pytest backend/geolibre_server/tests/test_x.py::test_y   # a single backend test
npm run test:worker                                # typecheck workers/viewer
npm run test:e2e                                    # Playwright smoke tests (e2e/) against the built web app
npm run check:rust                                 # cargo check the Tauri crate
```

The `:coverage` variants run the same suites and print a coverage summary; CI
runs them so every build reports coverage. They are now **gated on a floor**:
`test:frontend:coverage` fails below 78% lines / 78% branches / 63% functions,
and `test:backend:coverage` fails below 55% (`--cov-fail-under`). The floors sit
a few points under the current numbers as a **ratchet** — regressions fail CI,
and when coverage rises comfortably above a floor, raise the floor to lock in the
gain. The frontend
report only counts files a test actually imports, so a module with no test does
not appear at all rather than as 0%. The backend coverage run (and `npm run ci`,
which calls the `:coverage` variants) needs `pytest-cov` from the backend `dev`
extra. Install the **`test`** extra to run the *full* backend suite — without
the optional engines (geopandas/rasterio/sedona/httpx) the vector/raster/SQL/ML
tests skip themselves and CI is green but hollow:
`pip install -e "backend/geolibre_server[test]"`.

`npm run test:e2e` builds the web app, serves it with `vite preview`, and drives
it with Playwright (`@playwright/test`). First run: `npx playwright install
chromium`. The webServer reuses an already-running preview locally and rebuilds
in CI; add specs under `e2e/`.

Dependencies are watched two ways: **Dependabot** (`.github/dependabot.yml`)
opens grouped weekly update PRs for npm, pip (backend + `python/`), cargo, and
Actions, and the CI **`audit` job** runs `npm audit --audit-level=high`
(blocking) plus a non-blocking `pip-audit` of the resolved backend environment.

The `python/` package has its own pytest suite (`cd python && pytest`) and is built into a wheel via `npm run build:embed` (produces `apps/geolibre-desktop/dist-embed`, consumed by `python/hatch_build.py`). Its version is dynamic, sourced from `python/src/geolibre/__init__.py`.

## Pre-commit

`.pre-commit-config.yaml` includes a **local `npm-build` hook**, so `pre-commit run` compiles the whole app — it is slow and can touch unrelated build state. Scope it to the files you changed: `pre-commit run --files <paths>`. Run it before pushing.

## Architecture (the parts that span files)

The app is **store-driven**. `@geolibre/core` holds the Zustand store, domain types, and the `.geolibre.json` project schema — it is the single source of truth. Data flows one way:

1. Data enters through the Add Data menus, Tauri dialogs, the browser file picker, drag-and-drop, or a plugin control.
2. Local vector files that MapLibre can't render directly are converted to GeoJSON in-browser by **DuckDB-WASM Spatial** (`INSTALL spatial; LOAD spatial;` → `ST_Read`; GeoParquet via the Parquet reader; zipped Shapefiles via `shpjs` with a DuckDB fallback; KMZ unzipped client-side). The result calls `addGeoJsonLayer`.
3. Tile/service/raster/ArcGIS/MBTiles/plugin layers become `GeoLibreLayer` records.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opengeos/GeoLibre](https://github.com/opengeos/GeoLibre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
