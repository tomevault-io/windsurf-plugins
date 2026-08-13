---
trigger: always_on
description: Web-based interactive meteorological radar visualization tool processing NetCDF radar files. Developed as a university thesis project (FAMAF UNC). **Backend** (FastAPI) processes radar data; **Frontend** (React + Leaflet) renders interactive maps with tiled overlays.
---

# Radar Visualization Tool - AI Agent Guide

## Project Overview
Web-based interactive meteorological radar visualization tool processing NetCDF radar files. Developed as a university thesis project (FAMAF UNC). **Backend** (FastAPI) processes radar data; **Frontend** (React + Leaflet) renders interactive maps with tiled overlays.

## Architecture

### Core Data Flow
1. **Upload** → NetCDF files stored in `backend/app/storage/uploads/`
2. **Process** → PyART grids radar data → Rasterio generates GeoTIFF → COG (Cloud Optimized GeoTIFF) with overviews
3. **Serve** → TiTiler dynamically tiles COGs → Frontend renders via Leaflet using TileJSON endpoints
4. **Cache** → In-memory LRU cache (`backend/app/core/cache.py`) stores 2D/3D grids to avoid re-gridding (200-600 MB limits)

### Backend Structure (`backend/`)
- **`app/main.py`**: FastAPI app with TiTiler COG factory at `/cog` prefix
- **`app/routers/`**: API endpoints (`process.py`, `upload.py`, `pseudo_rhi.py`, `radar_stats.py`, `radar_pixel.py`)
- **`app/services/radar_processor.py`**: Core processing logic - PyART gridding, CAPPI/PPI/COLMAX/RHI products, COG generation
- **`app/services/radar_common.py`**: Shared utilities (field resolution, colormaps, gate filters, QC masking)
- **`app/utils/`**: PNG generation, CAPPI helpers, colormap definitions
- **`app/core/`**: Config (`settings`), cache (LRU), constants (field aliases, render params)
- **`app/schemas.py`**: Pydantic models for all API contracts

### Frontend Structure (`frontend/src/`)
- **`App.jsx`**: Main orchestrator - state management, API calls, multi-radar frame merging by timestamp
- **`components/MapView.jsx`**: Leaflet map with `COGTile` component fetching TileJSON from TiTiler
- **`api/backend.js`**: Axios client for all backend endpoints
- **`components/ProductSelectorDialog.jsx`**: UI for product/field/filter selection
- **`components/PseudoRHIDialog.jsx`**: Vertical transect generation UI with point picking
- **`components/AreaStatsDialog.jsx`**: Polygon-based stats extraction

## Critical Conventions

### 1. File Naming Convention
NetCDF files **must** follow: `{RADAR}_{STRATEGY}_{VOLUME}_{TIMESTAMP}Z.nc`
- Example: `RMA1_0315_01_20250819T001715Z.nc`
- Parsed by `backend/app/utils/helpers.py::extract_metadata_from_filename()`
- Used for radar identification, volume filtering, timestamp-based animation grouping

### 2. Field Resolution System
Fields are resolved through aliases defined in `backend/app/core/constants.py::FIELD_ALIASES`:
```python
FIELD_ALIASES = {
    "DBZH": ["DBZH", "corrected_reflectivity_horizontal"],
    "RHOHV": ["RHOHV", "rhohv"],
    # ...
}
```
- Use `radar_common.py::resolve_field()` to find actual field name in radar object
- Each field has render defaults in `FIELD_RENDER` (vmin/vmax/colormap)

### 3. Cache Strategy
**Two-tier caching** (`backend/app/core/cache.py`):
- `GRID2D_CACHE`: Stores collapsed 2D grids (post-PyART interpolation, pre-colormap) - 200 MB
- `GRID3D_CACHE`: Stores full 3D grids for vertical transects - 600 MB
- Cache keys include file hash, product, field, elevation/height, volume, but **NOT filters** (filters applied post-cache as masks)
- QC fields (RHOHV, etc.) cached separately alongside main field for post-grid masking

### 4. Filter Application
**Post-grid masking** approach (changed from older pre-grid gatefiltering):
- Filters defined as `RangeFilter` in `schemas.py` (field, min, max)
- QC filters (fields in `AFFECTS_INTERP_FIELDS` like RHOHV) applied as 2D masks after gridding
- Visual filters (same field as main) applied as direct masks on cached 2D array
- See `radar_processor.py::process_radar_to_cog()` lines ~420-470 for implementation

### 5. Multi-Radar Frame Merging
Frontend (`App.jsx::mergeRadarFrames()`):
- Groups layers from different radars by timestamp with 240-second tolerance
- Backend returns `ProcessResponse.results[]` (per-radar) → Frontend merges into unified frames
- Each frame contains multiple `LayerResult` objects sorted by `.order` field

### 6. COG Generation Pipeline
In `radar_processor.py::process_radar_to_cog()`:
1. Check cache for existing COG (early return if exists)
2. PyART reads NetCDF → builds 3D grid with `grid_from_radars()`
3. Collapse 3D → 2D via `collapse_grid_to_2d()` (PPI follows beam, CAPPI takes Z-slice, COLMAX takes max)
4. PyART's `write_grid_geotiff()` writes RGB GeoTIFF warped to Web Mercator
5. `convert_to_cog()` uses Rasterio COG driver with overviews (factors: 2,4,8,16) + DEFLATE compression
6. Return `LayerResult` with `tilejson_url` pointing to TiTiler endpoint

## Development Workflows

### Backend Setup (Windows)
```bash
conda create -n radar-env -c conda-forge python=3.11 gdal rasterio pyproj shapely
conda activate radar-env
pip install -r backend/requirements.txt
cd backend
make run  # or: uvicorn app.main:app --reload --port 8000
```

**Linux alternative**: `python3 -m venv venv && source venv/bin/activate && sudo apt install gdal-bin libgdal-dev libhdf5-dev libnetcdf-dev`

### Frontend Setup
```bash
cd frontend
npm install

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IgnaCat/radar-visualization-tool](https://github.com/IgnaCat/radar-visualization-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
