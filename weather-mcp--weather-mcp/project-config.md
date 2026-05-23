---
trigger: always_on
description: This document provides context and guidelines for AI assistants (Claude, etc.) working with this codebase.
---

# CLAUDE.md - AI Assistant Guide for Weather MCP Server

This document provides context and guidelines for AI assistants (Claude, etc.) working with this codebase.

## Project Overview

**Weather MCP Server** is a Model Context Protocol (MCP) server providing weather data from NOAA and Open-Meteo APIs. It enables AI assistants to fetch real-time weather forecasts, current conditions, historical data, air quality, marine conditions, and severe weather alerts.

- **Language:** TypeScript (Node.js)
- **Version:** 1.6.1 (Production Ready)
- **License:** MIT
- **MCP SDK:** @modelcontextprotocol/sdk v1.21.0

## Architecture

### Core Components

```
src/
├── index.ts                 # MCP server entry point, tool registry
├── handlers/                # Tool request handlers (one per MCP tool)
│   ├── forecastHandler.ts
│   ├── currentConditionsHandler.ts
│   ├── alertsHandler.ts
│   ├── historicalWeatherHandler.ts
│   ├── statusHandler.ts
│   ├── locationHandler.ts
│   ├── airQualityHandler.ts
│   ├── marineConditionsHandler.ts
│   ├── riverConditionsHandler.ts
│   ├── wildfireHandler.ts
│   └── savedLocationsHandler.ts  # Saved locations management (v1.7.0)
├── services/                # External API clients
│   ├── noaa.ts             # NOAA Weather API client
│   ├── openmeteo.ts        # Open-Meteo API client
│   ├── nominatim.ts        # Nominatim/OSM geocoding client (v1.7.0)
│   ├── locationStore.ts    # Saved locations storage service (v1.7.0)
│   ├── nifc.ts             # NIFC wildfire API client
│   └── usgs.ts             # USGS water services client
├── types/                   # TypeScript type definitions
│   ├── noaa.ts
│   ├── openmeteo.ts
│   ├── nominatim.ts        # Nominatim API types (v1.7.0)
│   └── savedLocations.ts   # Saved locations types (v1.7.0)
├── utils/                   # Shared utilities
│   ├── cache.ts            # LRU cache with TTL
│   ├── validation.ts       # Input validation
│   ├── units.ts            # Unit conversions
│   ├── logger.ts           # Structured logging
│   ├── locationResolver.ts # Location name/coordinate resolution (v1.7.0)
│   ├── airQuality.ts       # AQI calculations
│   ├── marine.ts           # Wave/ocean utilities
│   ├── fireWeather.ts      # Fire weather indices
│   ├── distance.ts         # Haversine distance calculations
│   └── geohash.ts          # Geohash encoding/decoding
├── config/                  # Configuration
│   ├── cache.ts            # Cache TTL settings
│   └── displayThresholds.ts # Display logic constants
└── errors/                  # Custom error classes
    └── ApiError.ts
```

### Design Patterns

1. **Handler Pattern:** Each MCP tool has a dedicated handler function in `src/handlers/`
2. **Service Layer:** API clients are abstracted into service classes with retry logic
3. **Validation First:** All user inputs validated before processing (see `src/utils/validation.ts`)
4. **Caching Strategy:** LRU cache with TTL based on data volatility (see `src/config/cache.ts`)
5. **Error Hierarchy:** Custom error classes for different failure scenarios

## Key Features (16 MCP Tools)

1. **get_forecast** - 7-day forecasts (NOAA/Open-Meteo, auto-select by location) - Now supports saved locations via `location_name`
2. **get_current_conditions** - Current weather + fire weather indices (NOAA, US only)
3. **get_alerts** - Weather alerts/warnings (NOAA, US only)
4. **get_historical_weather** - Historical data 1940-present (Open-Meteo, global)
5. **check_service_status** - API health check (all services)
6. **search_location** - Location search/geocoding (Nominatim/OSM, better small town coverage)
7. **get_air_quality** - Air quality index + pollutants (Open-Meteo, global)
8. **get_marine_conditions** - Wave height, swell, currents (Open-Meteo, global)
9. **get_weather_imagery** - Weather radar/precipitation imagery (RainViewer, global)
10. **get_lightning_activity** - Real-time lightning detection (Blitzortung.org, global)
11. **get_river_conditions** - River levels and flood monitoring (NOAA/USGS, US only)
12. **get_wildfire_info** - Active wildfire tracking (NIFC, US only)
13. **save_location** - Save frequently used locations with aliases (NEW in v1.7.0)
14. **list_saved_locations** - View all saved locations (NEW in v1.7.0)
15. **get_saved_location** - Get details for a saved location (NEW in v1.7.0)
16. **remove_saved_location** - Delete a saved location (NEW in v1.7.0)

## Development Guidelines

### Code Style

- **TypeScript Strict Mode:** All strict flags enabled (see `tsconfig.json`)
- **No `any` types:** Use proper typing or `unknown` with validation
- **Explicit returns:** All functions must return on all code paths
- **No unused variables:** Compiler enforces `noUnusedLocals` and `noUnusedParameters`

### Adding New Features

1. **Types First:** Define TypeScript interfaces in `src/types/`
2. **Validation:** Add validators to `src/utils/validation.ts`
3. **Handler:** Create handler in `src/handlers/` following existing patterns
4. **Service (if needed):** Add API methods to `src/services/noaa.ts` or `openmeteo.ts`
5. **Tool Registration:** Register in `src/index.ts` (ListToolsRequestSchema and CallToolRequestSchema)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weather-mcp/weather-mcp](https://github.com/weather-mcp/weather-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
