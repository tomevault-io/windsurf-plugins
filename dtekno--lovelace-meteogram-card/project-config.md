---
trigger: always_on
description: This is a **Home Assistant Lovelace custom card** that displays weather forecasts as interactive meteogram charts using D3.js. The card supports both Home Assistant weather entities and the Met.no API as data sources.
---

# GitHub Copilot Instructions for Meteogram Card

## Project Overview

This is a **Home Assistant Lovelace custom card** that displays weather forecasts as interactive meteogram charts using D3.js. The card supports both Home Assistant weather entities and the Met.no API as data sources.

**Tech Stack:**
- TypeScript + Lit Element (Web Components)
- D3.js v7 for SVG chart rendering
- Home Assistant Custom Card API
- localStorage for aggressive caching

## Core Architecture Principles

### 1. Dual Data Source Support

The card implements TWO distinct data retrieval patterns:

#### Weather Entity (Modern HA 2023.9+)
- Uses **subscription-based architecture** via `hass.connection.subscribeMessage`
- Forecast type: `'hourly'`
- **CRITICAL**: `entity.attributes.forecast` does NOT exist in modern HA - never reference it
- Service calls: `weather.get_forecasts` for manual refresh
- Cache: `meteogram-card-entity-weather-cache` (per-entity timestamps)

#### Met.no API
- Direct HTTP calls with User-Agent header required
- Cache: `metno-weather-cache` with `expiresAt` timestamps
- Respects API rate limits and caching headers

### 2. Intelligent Subscription Management

**Pause/Resume System:**
- Pauses subscriptions when tab is hidden or card scrolled out of view (saves CPU/battery)
- Resumes with data freshness check (compares `entity.last_updated` vs cached timestamp)
- Uses `IntersectionObserver` for viewport detection
- Uses `document.visibilitychange` for tab visibility

**When suggesting subscription code:**
- Always include pause/resume logic
- Always validate data freshness on resume
- Use `this._unsubWeather` pattern for cleanup

### 3. Two-Tier Cache Strategy

Both caches require the following arrays to be valid:
```typescript
['time', 'temperature', 'rain', 'rainMin', 'rainMax', 'snow', 
 'cloudCover', 'windSpeed', 'windGust', 'windDirection', 'symbolCode', 'pressure']
```

**Cache Operations:**
- Validate structure before use (arrays present and have length)
- Handle JSON parse errors gracefully
- Clean up entries older than 24h
- Never clear cache just because it's expired (fallback for offline mode)

### 4. Temperature Gradient Rendering

The temperature line uses a dynamic SVG gradient centered on 0°C (freezing point):

**Key Implementation Details:**
- Uses `gradientUnits="userSpaceOnUse"` for absolute SVG coordinate mapping
- Gradient spans from `yTemp(maxTemp)` to `yTemp(minTemp)` (actual temperature range)
- Color stops positioned at exact Y coordinates using `yTemp()` scale function
- Sharp transition AT 0°C: warm colors (red/orange) above, cold colors (blue) below
- Additional transitions at 20°C (deep red), 10°C (orange-red), -5°C (deep blue)

**Color Scale:**
- ≥20°C: Deep red (#cc0000)
- 10°C: Orange-red (#ff6600)
- 0°C: Light orange (#ff9933) → Light blue (#66b3ff) [sharp transition]
- -5°C: Deep blue (#0066cc)
- ≤-5°C: Very deep blue (#003d7a)

**CSS Compatibility:**
- Check for custom `--meteogram-temp-line-color` CSS variable
- If set, use custom color instead of gradient
- Never set stroke in CSS (breaks gradient) - apply in JavaScript conditionally

### 5. Wind Barb Rendering for Mixed-Resolution Data

Wind barbs display wind speed and direction using meteorological conventions. The card handles **mixed-resolution data** where forecasts transition from hourly (0-48h) to 6-hourly (48h+) intervals.

**Key Implementation Details:**
- **Data availability check**: Validate BOTH `windSpeed` AND `windDirection` arrays have non-null values
  - Previously only checked `windSpeed`, causing barbs to disappear when direction was missing
  - Wind barbs require both speed and direction to render
- **Resolution detection**: Detect transition point by comparing time intervals between data points
  - Hourly data: ~1 hour intervals
  - 6-hourly data: ~6 hour intervals
  - Transition typically occurs around index 52-53 in Met.no forecasts
- **Adaptive rendering**:
  - **Hourly section**: Place wind barbs between even-hour grid lines (every 2 hours)
  - **6-hourly section**: Place wind barbs every other data point (every 12 hours)
- **Timezone-agnostic**: Use index-based filtering (`i % 2 === 0`) rather than absolute time values
  - Ensures consistent spacing regardless of timezone or data start time
  - More robust than checking for specific hours like 00:00, 12:00

**Wind Barb Spacing Strategy:**
```typescript
// High-resolution (hourly): Every 2 hours
const highResIndices = [];
for (let i = 0; i < transitionIdx; i++) {
  if (time[i].getHours() % 2 === 0) highResIndices.push(i);
}

// Low-resolution (6-hourly): Every 12 hours = every other point
for (let i = 0; i < lowResIndices.length; i++) {
  if (i % 2 === 0) {  // Index-based, not time-based
    // Draw barb
  }
}
```

**Responsive Rendering:**
- Normal width: Show all calculated positions
- Narrow screens (`width < 400`): Skip additional barbs to prevent overlap
  - Hourly section: Every 4 hours
  - 6-hourly section: Every 24 hours

**Wind Speed Scaling:**
- Wind barb length scales with wind speed using `d3.scaleLinear()`
- Domain: `[0, max(15, maxWindSpeed)]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DTekNO/lovelace-meteogram-card](https://github.com/DTekNO/lovelace-meteogram-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
