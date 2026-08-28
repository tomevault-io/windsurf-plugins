---
trigger: always_on
description: QgisStreamMCP exposes a full QGIS Desktop instance as an MCP Server.
---

# CLAUDE.md — QgisStreamMCP

## Project

QgisStreamMCP exposes a full QGIS Desktop instance as an MCP Server.
Single Docker container with QGIS GUI + Xvfb + noVNC + MCP Server.

## Architecture

```
Container (single)
  supervisord
  ├── Xvfb :99 (virtual display)
  ├── fluxbox (window manager)
  ├── QGIS Desktop (GUI, PyQGIS bridge via startup script)
  ├── x11vnc → websockify/noVNC (:6080)
  ├── api_server.py (FastAPI REST :8080)
  ├── main_mcp.py (MCP Server :8100, 40 tools)
  └── stream_server.py (MJPEG :8081)
```

Communication: MCP Server → UNIX socket → QGIS Bridge (runs inside QGIS)

## Key files

- `main_mcp.py` — MCP Server with 40 tools, 10 resources, 3 prompts
- `src/qgis_bridge.py` — Runs inside QGIS, UNIX socket listener, 45 actions
- `src/qgis_helpers.py` — Python helpers injected into execute_python
- `src/api_server.py` — FastAPI REST wrapper
- `src/stream_server.py` — MJPEG stream
- `skills/*.md` — MCP Resources (PyQGIS, Processing, cartography, smart loading, recipes, etc.)
- `templates/*.qpt` — Print layout templates (A3 landscape, A4 portrait)
- `templates/web/` — Leaflet HTML templates (standard, flood, temporal)
- `recipes/*.json` — Workflow recipes (5 pre-built analyses)
- `datasources.json` — 30+ pre-configured French data sources (IGN, OSM, BD TOPO, Georisques)
- `Dockerfile` — Single container build
- `supervisord.conf` — Process orchestration
- `entrypoint.sh` — Container startup

## MCP Tools (40)

### Core
- `execute_python` — Run PyQGIS code with `helpers` module
- `get_screenshot` — Capture QGIS canvas
- `get_project_info` / `new_project` / `open_project` / `save_project`
- `qgis_desktop_ui` — Open interactive QGIS view in conversation

### Data loading
- `set_study_zone` — Define study area (geocodes, stores bbox)
- `get_study_zone` — Read stored zone
- `smart_load` — Load from catalog (WFS → local GPKG, rasters stream)
- `list_datasources` / `add_from_catalog` — Browse and add data sources
- `add_layer` / `remove_layer` — Manual layer management

### Analysis
- `run_processing` — 1000+ Processing algorithms
- `search_algorithms` — Search algorithms
- `get_features` — Query features from vector layers

### Cartography & export
- `set_layer_style` / `set_layer_visibility` — Symbology and visibility
- `apply_layout_template` / `list_layout_templates` — Print layout templates
- `export_pdf` — Export layout to PDF
- `export_web_map` — Export visible layers as interactive Leaflet HTML
- `export_flood_map` — Interactive flood analysis HTML (water height slider, building exposure stats)
- `export_temporal_map` — Interactive temporal analysis HTML (year slider, animated playback, per-band stats)
- `export_layer` — Export layer to GPKG/GeoJSON/Shapefile/CSV
- `export_qfield` — QField-ready ZIP package (.qgz + GPKGs + editable Observations layer with form widgets)
- `export_grist` — Grist document (.grist SQLite) from QGIS project or any HTML with GeoJSON (Choice/Date/Ref columns, custom map widget, form pages)

### Recipes (workflow automation)
- `list_recipes` — Browse workflow recipes
- `get_recipe` — Get recipe with parameter substitution
- `run_recipe` — Execute a complete recipe automatically (all steps in one shot)

### Files & interaction
- `upload_file` / `download_file` / `list_files` / `delete_file`
- `download_project` — Save project as .qgz
- `mouse_click` / `mouse_scroll` / `key_press` / `mouse_drag` — GUI control
- `zoom_to` — Zoom canvas

## Workflow context

Every mutating tool response includes a `_context` with:
- **phase**: setup / analysis / cartography / export (auto-detected)
- **study_zone**: current zone name
- **layers**: loaded vector layers with feature counts
- **hint**: suggested next action

This guides the AI through structured workflows without hard mode restrictions.

## Recipes

Pre-built workflow templates in `recipes/*.json`:
- `densite_bati` — Building density (hex grid + graduated symbology)
- `urbanisme_general` — Complete urban overview
- `risque_inondation` — Flood risk analysis (buffer zones + building exposure + interactive flood map)
- `occupation_sol` — Land cover analysis
- `pression_fonciere_cotiere` — Coastal land pressure evolution (DVF 2020-2024 + coastal bands + temporal web map)

Usage: `list_recipes()` → `run_recipe(id="risque_inondation", zone="Nimes")` (automated)
Or manual: `get_recipe(id="densite_bati", zone="Montpellier")` → follow steps one by one

## Interactive web exports

Three specialized Leaflet HTML templates for interactive deliverables:
- **Standard** (`leaflet_template.html`) — Vector layers + basemap
- **Flood** (`leaflet_flood_template.html`) — Water height slider, building exposure stats, animation
- **Temporal** (`leaflet_temporal_template.html`) — Year slider, per-band statistics, trend arrows, animated playback

## Grist export

`export_grist` converts geographic data into a complete `.grist` document (SQLite).

### Two modes

1. **From QGIS project** (default) — exports loaded layers with typed columns, map widget, stats, forms
2. **From HTML file** (`html_path`) — universal converter for any HTML containing GeoJSON

### HTML→Grist pipeline

Any Leaflet HTML (from `export_web_map`, `export_flood_map`, `export_temporal_map`, qgis2web, or custom) is converted:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nic01asFr/QgisStreamMCP](https://github.com/nic01asFr/QgisStreamMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
