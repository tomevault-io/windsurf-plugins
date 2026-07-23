---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Obsidian Map View is a sophisticated Obsidian plugin that transforms notes into an interactive geographic information system (GIS). It parses geolocation data from note frontmatter, inline links, and external files (GPX, KML, GeoJSON), renders them on interactive maps, and provides powerful querying, filtering, and display customization capabilities.

## Development Commands

```bash
# Development with watch mode
npm run dev-dist

# Production build
npm run build

# Code formatting
npm run prettier

# Check code formatting
npm run stylecheck
```

## Tech Stack

- **TypeScript** with Svelte 5 for UI components
- **Rollup** for bundling with plugins for TypeScript, Svelte, PostCSS, and images
- **Leaflet** ecosystem for mapping (leaflet, markercluster, geosearch, geoman, offline)
- **boon-js** for query language parsing
- **@tmcw/togeojson** for GPX/KML conversion
- **FontAwesome** for marker icons
- **Obsidian API** for plugin integration

## Core Architecture

### Data Flow: File → Layer → Map

1. **Parsing** (`src/geoHelpers.ts`, `src/geojsonParser.ts`):
    - `matchInlineLocation()`: Extracts `[name](geo:lat,lng) tag:foo` patterns from file content
    - `getFrontMatterLocation()`: Reads `location:` property from Obsidian metadata cache
    - `getGeoJsonLayersFromFile()`: Parses GeoJSON/GPX/KML files and inline geojson code blocks

2. **Layer System** (abstract base: `src/baseGeoLayer.ts`):
    - `FileMarker` (`src/fileMarker.ts`): Individual location markers from notes
    - `GeoJsonLayer` (`src/geojsonLayer.ts`): Paths and shapes from GeoJSON data
    - Each logical layer maintains multiple Leaflet layer instances (one per map container) via `geoLayers: Map<containerId, leaflet.Layer>`

3. **Layer Cache** (`src/layerCache.ts`):
    - Plugin-global repository indexed by layer ID and file path
    - Rebuilds affected layers when files change via `updateMarkersWithRelationToFile()`
    - Initialization can be deferred until first map opens (`loadLayersAhead` setting)

4. **Display Rules** (`src/displayRulesCache.ts`):
    - Query-based styling engine that applies icon properties, path options, and badges
    - Rules applied in sequence; matching rules override previous properties
    - Uses Query system (`src/query.ts`) to match layers via `tag:`, `path:`, `linkedfrom:`, etc.

5. **Map Rendering** (`src/mapContainer.ts`):
    - `filterAndPrepareMarkers()`: Applies user query filters and builds link edges
    - `updateMapLayers()`: Diffs old/new layers and updates Leaflet map (reuses unchanged layers)
    - Manages marker clusters, tile layers, and all UI controls

### Key Components

**Main Plugin** (`src/main.ts`):

- Entry point that registers views, commands, protocol handlers, and event listeners
- Maintains `allMapContainers` registry for all active map instances
- Handles vault file events (create, modify, delete, rename) to trigger layer updates
- Provides global handlers for geolink interactions in editor

**Map Views**:

- `MainMapView` (`src/mainMapView.ts`): Standalone full-featured map view
- `EmbeddedMap` (`src/embeddedMap.ts`): Inline maps from `mapview` code blocks with state persistence
- `BasesMapView` (`src/basesMapView.ts`): Integration with Obsidian Bases
- `MapPreviewPopup` (`src/mapPreviewPopup.ts`): Transient previews on geolink hover

**Query System** (`src/query.ts`):

- Boolean query language: `tag:#foo AND path:"bar" OR linkedfrom:"Trip Plan"`
- Parsed into RPN (reverse Polish notation) for fast evaluation
- Used for both display rule matching and user filtering

**Display Rules** (`src/displayRulesCache.ts`, `src/markerIcons.ts`):

- Rules composed of: query + icon details + path options + badges
- `IconFactory.getIconFromRules()` creates Leaflet markers with FontAwesome icons
- Badges add corner indicators (up to 4 per marker)

**State Management** (`src/mapState.ts`):

- Immutable `MapState` object: position, zoom, query, display options
- `mergeStates()` for partial updates, `areStatesEqual()` for diffing
- Persisted in embedded maps and presets

**Editor Integration**:

- `src/codemirrorViewPlugin.ts`: Decorates inline geolinks with custom event handlers
- `src/geoLinkReplacers.ts`: Post-processes reading view to make geolinks clickable
- `src/locationSuggest.ts`: Autocomplete for location search in `[](geo:)` templates
- `src/tagSuggest.ts`: Tag autocomplete for queries

### Multi-Instance Pattern

The plugin supports multiple simultaneous map views (main views, embeds, previews). Each logical layer (FileMarker or GeoJsonLayer) can exist as different Leaflet objects in different containers:

```typescript
class BaseGeoLayer {
    geoLayers: Map<string, leaflet.Layer> = new Map();
    // Same geographic data, different visual representations per container
}
```

This enables:

- Independent filtering per view (same note shown differently in two maps)
- Efficient reuse of layer data without duplication
- Container-specific display state (hover, selection)

### Performance Considerations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esm7/obsidian-map-view](https://github.com/esm7/obsidian-map-view) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
