---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

VEIL is a standalone, region-agnostic, fully georeferenced 3D digital-twin engine: point it at a DEM (and optionally imagery and map layers) and it builds a browser-viewable twin of that ground. No database server, no cloud, no build step at view time: a zero-dependency Node static server (`server.js`) serves a Three.js viewer (`public/`) over a self-contained bundle of geospatial data (a twin's `data/` or any `--data-dir`/`TWIN_DATA_DIR`). Nothing is fetched from the network at view time. **Twin data is private and gitignored** — the repo ships only code plus one small demo AOI (`packs/us-national/demo/`); see "Privacy" in the README.

The system of record is the **twin store** — canonically, the append-only write journal in `<data>/journal/`, materialized as a GeoPackage at `<data>/twin.gpkg` (see "Twin store" below). The flat JSON the viewer loads is an **export** of the store — the viewer is a window on the store, not the product.

## Commands

```bash
npm start                 # serve at http://127.0.0.1:4173 (PORT/HOST/TWIN_DATA_DIR env override)
npm run demo              # build the Flatirons demo twin into twins/demo/data (needs internet + GDAL)
npm run serve-demo        # serve the demo twin at http://127.0.0.1:4174
npm test                  # query-layer tests against the offline committed fixture twin
npm run test:demo         # query-layer tests against the real-data demo twin (builds it if missing)
npm run build-test-fixture # regenerate the committed offline test fixture twin
npm run build-from-aoi    # build a whole US twin from one AOI polygon (3DEP + NAIP + LANDFIRE)
npm run ingest-dem        # genesis: any DEM (+ AOI) -> terrain grid + georef.json
npm run ingest-imagery    # align aerial imagery to the terrain footprint
npm run add-layer         # import any vector/raster file as a draped, clickable layer
npm run rebuild-store     # reconstruct <data>/twin.gpkg from its journal
npm run migrate           # genesis only: seed a store (+ journal) from a flat data/ bundle
npm run export            # regenerate every viewer JSON payload from the store
npm run build-vegetation  # tree/shrub instances -> store -> export (analyze_vegetation.py + build_surrounding_vegetation.py)
npm run canopy-history    # temporal query: canopy density per pipeline run (--bbox to scope)
npm run build-survey-package  # generate the QField survey package (Survey companion)
npm run analyze-hydrology # Tier-1 terrain hydrology: flow/wetness/ponding/seep layers + summary
npm run hydro-scenario    # snowmelt scenario CLI (the Simulation window calls it via /api/simulate)
python3 scripts/fetch_distant_terrain.py # viewshed distant-terrain manifest/fetch metadata
python3 scripts/analyze_viewshed.py      # precomputed horizon + AOI cumulative viewshed drape
python3 scripts/mcp_server.py       # MCP server (stdio) over the twin store — see docs/mcp.md
npm run test:plan            # Plan engine/journal/GAIA surface + real REST integration
python3 -m venv .venv-live && .venv-live/bin/python -m pip install -r requirements-live.txt
                              # install/update live telemetry bridge deps

docker compose up --build              # serve in a container (Node + full Python pipeline, pinned)
docker compose run --rm veil <cmd>     # run any npm/pipeline step in that image (twins persist to ./twins)
```

`npm test` runs `scripts/twin_query_test.py`, which asserts against the deterministic Flatirons demo twin (`twins/demo/data`, built from the committed AOI on first run; set `TWIN_DATA_DIR` to test another twin). Visual verification: `scripts/screenshot.js` is a headless Playwright harness (`node scripts/screenshot.js <out.png> [layerLabelsCsv] [waitMs] [click]`) that loads the viewer against a running `npm start`, optionally isolates layers by label, clicks the canvas to exercise click-to-identify, and reports page/console errors (`npm i -D playwright` first; it is not a dependency).

The Python pipeline scripts need GDAL (`osgeo`), numpy, pyproj, and Pillow; `requirements.txt` pins the pip-installable subset (`mcp`, `pyproj`, `Pillow`) and documents that GDAL/numpy come from the OS, not pip (the bindings must match the system GDAL, and `ReadAsArray`/`WriteArray` need a matching numpy). The fetch scripts (`national_fetch.py`, `packs/us-national/fetch_landfire.py`, `build_from_aoi.py`) need internet for one-time API snapshots. The build scripts are one-time/occasional data-pipeline steps; day-to-day viewer work needs only `npm start`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zymazza/mazzap](https://github.com/zymazza/mazzap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
