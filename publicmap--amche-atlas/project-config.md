---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm start` or `npm run dev` - Start development server on port 4035
- `npm run build` - Build for production using Vite
- `npm run preview` - Preview built files on port 4035
- `npm test` - Run tests with Vitest
- `npm run test:watch` - Run tests in watch mode
- `npm run test -- path/to/test.js` - Run a single test file
- `npm run lint` - Run JSON linting for atlas configuration files
- `npx playwright test` - Run end-to-end tests (requires dev server running)

## Architecture Overview

This is **amche-atlas**, a web-based GIS platform for interactive spatial data visualization. The application uses a static site architecture with minimal dependencies, designed for simplicity and community contribution.

### Technology Stack
- **Mapbox GL JS** - Client-side map rendering and interactivity
- **jQuery** - DOM manipulation
- **Shoelace** - UI components (primarily for layer controls)
- **Tailwind CSS** - Responsive CSS framework
- **Vite** - Build tool and development server
- **Vitest** - Testing framework

### Core Architecture

**Debounced Updates Pattern**

The application uses setTimeout-based debouncing rather than direct event-driven updates in several critical areas:

- **URL Updates** (`url-manager.js`): 300ms debounce prevents excessive browser history entries when multiple state changes occur rapidly (e.g., selecting multiple features, adjusting layers)
- **Map Interactions**: Debouncing prevents performance issues from frequent map events (pan, zoom, hover)

**Why debouncing instead of direct events:**
- **Browser History Management**: Each URL change creates a history entry; debouncing batches related changes into a single entry
- **Performance**: Serializing layer configurations (including GeoJSON) and updating URLs is expensive; batching reduces overhead
- **User Experience**: Prevents the "back" button requiring many clicks to return to a previous meaningful state

**Tradeoffs:**
- More complex debugging (asynchronous updates, race conditions)
- Potential for option conflicts when multiple debounced calls override each other (see `setStateManager` handling selection layer updates)
- Less predictable timing compared to synchronous event handlers

**When working with debounced updates:**
- Be aware that the last call's options will override earlier calls within the debounce window
- Use console logging to trace the sequence of calls and their options
- Consider whether state changes need to merge options rather than replace them

**Configuration-Driven Maps**
The entire application is driven by JSON configuration files in `/config/`:
- `_defaults.json` - Default styling for atlas and layer types
- `index.atlas.json` - Main map configuration
- `*.atlas.json` - Additional themed configurations

The JSON configurations are cascaded as follows:
1. `_defaults.json` is loaded first
2. `index.atlas.json` is loaded second
3. `*.atlas.json` are loaded third

This makes it possible to scope customizations to specific atlases or layers without affecting other atlases or repeating definitions.

**URL API**

The application supports a URL API for deep linking and sharing map configurations. The following parameters are supported:
- `?atlas=filename` - Load local config file
- `?atlas=https://...` - Load remote config
- `?atlas={"name":"..."}` - Inline JSON config
- `?layers=layer1,layer2` - Override visible layers

Available API options and examples are maintained in `/docs/API.md`

**Modular JavaScript Structure (`/js/`)**
- `map-init.js` - Application entry point and map initialization. This is the main file that is loaded when the application is started and resolves the various atlas configurations and layer presets to apply.
- `layer-registry.js` - Central registry managing layer presets and atlas configurations. This is used to load the various layer presets and atlas configurations.
- `map-layer-controls.js` - Main UI for layer toggles (uses Shoelace components)
- `map-feature-control.js` - Feature inspection and interaction
- `url-manager.js` - URL parameter handling and permalink management
- `mapbox-api.js` - Mapbox GL JS abstraction layer
- `layer-creator-ui.js` - UI for creating custom layers dynamically
- `map-search-control.js` - Location search functionality
- `map-export-control.js` - Export map to PDF/image
- `terrain-3d-control.js` - 3D terrain visualization toggle

**Layer System**
The application supports multiple data layer types:
- `style` - Uses existing Mapbox style sources
- `vector` - Vector tiles (.pbf/.mvt) with sourceLayer
- `geojson` - GeoJSON / KML vector data
- `tms` - Raster XYZ tile services
- `wmts` - OGC WMTS endpoints
- `wms` - OGC WMS endpoints
- `cog` - Cloud Optimized GeoTIFF (HTTP range requests via Mapbox `TileProvider`)
- `csv` - Tabular data with auto-detected lat/lng columns
- `img` - Single georeferenced image overlay
- `raster-style-layer` - Toggle for a raster layer in the base Mapbox style
- `layer-group` - Bundled toggle for multiple existing layer IDs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [publicmap/amche-atlas](https://github.com/publicmap/amche-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
