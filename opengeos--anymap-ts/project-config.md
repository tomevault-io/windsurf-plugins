---
trigger: always_on
description: This document provides essential information for AI coding agents working on the anymap-ts project.
---

# anymap-ts Developer Guide

This document provides essential information for AI coding agents working on the anymap-ts project.

## Project Overview

**anymap-ts** is a Python package for creating interactive maps in Jupyter notebooks using [anywidget](https://anywidget.dev/) and TypeScript. It provides a unified Python interface to multiple JavaScript mapping libraries.

### Supported Mapping Libraries

| Library | Module | Use Case |
|---------|--------|----------|
| MapLibre GL JS | `MapLibreMap` | Default, general-purpose vector maps |
| Mapbox GL JS | `MapboxMap` | Advanced styling, 3D terrain |
| Leaflet | `LeafletMap` | Lightweight, mobile-friendly maps |
| OpenLayers | `OpenLayersMap` | Enterprise, WMS/WMTS, projections |
| DeckGL | `DeckGLMap` | GPU-accelerated data visualization |
| Cesium | `CesiumMap` | 3D globe, terrain, 3D Tiles |
| KeplerGL | `KeplerGLMap` | Data exploration UI |
| Potree | `PotreeViewer` | LiDAR point cloud visualization |

### Architecture

The project follows a hybrid Python/TypeScript architecture:

- **Python side (`anymap_ts/`)**: Widget classes extending `anywidget.AnyWidget` with traitlets for state synchronization
- **TypeScript side (`src/`)**: Renderer classes extending `BaseMapRenderer` that handle the actual map rendering
- **Communication**: Bidirectional via anywidget's traitlet synchronization (`_js_calls`, `_js_events`)

## Project Structure

```
anymap-ts/
├── src/                          # TypeScript source code
│   ├── index.ts                  # Main entry point (exports MapLibre by default)
│   ├── core/                     # Base classes
│   │   ├── BaseMapRenderer.ts    # Abstract base for all renderers
│   │   └── StateManager.ts       # State persistence
│   ├── maplibre/                 # MapLibre implementation
│   │   ├── MapLibreRenderer.ts   # Main renderer class
│   │   ├── adapters/             # Layer adapters (COG, Zarr, DeckGL)
│   │   ├── plugins/              # UI plugins (GeoEditor, LayerControl)
│   │   └── index.ts              # Widget entry point (render function)
│   ├── mapbox/, leaflet/, ...    # Other library implementations
│   └── types/                    # TypeScript type definitions
│       ├── anywidget.ts          # Widget communication types
│       ├── maplibre.ts           # MapLibre-specific types
│       └── ...
├── anymap_ts/                    # Python package
│   ├── __init__.py               # Exports all map classes
│   ├── base.py                   # MapWidget base class
│   ├── maplibre.py               # MapLibreMap widget
│   ├── mapbox.py, leaflet.py...  # Other widget implementations
│   ├── basemaps.py               # Basemap URL utilities
│   ├── utils.py                  # Helper functions
│   └── static/                   # Built JavaScript/CSS bundles
├── examples/                     # HTML examples for development
├── docs/                         # MkDocs documentation
├── tests/                        # Test suites
│   ├── python/                   # pytest tests
│   └── typescript/               # vitest tests
└── package.json                  # npm scripts and dependencies
```

## Technology Stack

### Python
- **Python**: 3.10+
- **Build system**: hatchling + hatch-jupyter-builder
- **Core dependencies**: anywidget>=0.9.0, traitlets>=5.0.0, xyzservices>=2023.10.0
- **Optional**: geopandas (vector), localtileserver (raster)
- **Testing**: pytest, pytest-cov
- **Linting**: ruff, black, mypy

### TypeScript/JavaScript
- **Target**: ES2022, ESNext modules
- **Build tools**: esbuild (library bundles), Vite (examples)
- **Testing**: vitest with jsdom environment
- **Key dependencies**:
  - maplibre-gl >=5.16.0
  - mapbox-gl ^3.18.1
  - leaflet ^1.9.4
  - deck.gl 9.2.6
  - cesium ^1.138.0
  - ol (OpenLayers) ^10.8.0

## Build Commands

### Setup
```bash
# Install Python package in editable mode
pip install -e ".[dev]"

# Install Node.js dependencies
npm install --legacy-peer-deps
```

### Build JavaScript Bundles
```bash
# Build all library bundles (production)
npm run build:lib
# or
npm run build:prod

# Build specific libraries
npm run build:maplibre    # Primary target, builds by default
npm run build:mapbox
npm run build:leaflet
npm run build:deckgl
npm run build:openlayers
npm run build:cesium
npm run build:keplergl
npm run build:potree

# Build all including examples
npm run build:all

# Watch mode (development)
npm run watch
```

### Python Package Build
The Python package build automatically triggers npm builds via `hatch-jupyter-builder`:
```bash
# Build wheel (includes JS bundles)
pip install build
python -m build
```

### Development Server
```bash
# Start Vite dev server for examples
npm run dev           # Serves examples/ directory on port 5173
```

## Testing

### Python Tests
```bash
# Run all Python tests with coverage
pytest

# Run specific test file
pytest tests/python/test_basemaps.py
```

Configuration in `pyproject.toml`:
- Test path: `tests/python/`
- Coverage includes: `anymap_ts/` module
- Coverage omits: tests, `__pycache__`

### TypeScript Tests
```bash
# Run all TypeScript tests once
npm run test

# Run in watch mode
npm run test:watch
```

Configuration in `vitest.config.ts`:
- Environment: jsdom

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opengeos/anymap-ts](https://github.com/opengeos/anymap-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
