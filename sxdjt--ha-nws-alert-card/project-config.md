---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom Lovelace card that displays active US National Weather Service (NWS) alerts. It's a single-file JavaScript Web Component with no build process or dependencies.

**Critical:** This card provides weather information but should not be relied upon for critical safety decisions. Always include appropriate disclaimers when making changes.

## Architecture

### Single-File Component

The entire card is implemented in `nws-alert-card.js` as a Web Component extending `HTMLElement`:

- Uses Shadow DOM for encapsulation
- Implements Home Assistant's custom card interface
- No external dependencies or build tools required

### Key Architectural Patterns

**State Management:**

- `_lastAlertIds` (Set): Tracks current alert IDs for change detection
- `_expandedAlerts` (Set): Tracks manually expanded alerts (when `show_expanded: false`)
- `_collapsedAlerts` (Set): Tracks manually collapsed alerts (when `show_expanded: true`)
- `_alertsCache` (Map): Caches alert data for re-rendering without re-fetching
- `_zoneCache` (Map): Caches coordinate-to-zone lookups (24-hour TTL, 10 entry LRU)
- `_currentZone` (string): Currently active zone code
- `_isMobile` (boolean): Cached mobile detection result
- `_zoneResolveTimeout` (number): Debounce timer for entity updates
- `_lastMaxSeverity` (string|null): Tracks previous highest severity level for upgrade detection
- `_actionQueue` (Array): Queue for pending actions (prevents concurrent triggers)
- `_actionInProgress` (boolean): Prevents concurrent action execution
- All state is instance-based (no global state)

**Event Handling:**

- Uses event delegation pattern via `_content.addEventListener('click')` to avoid memory leaks
- Toggle buttons use `data-alert-id` attributes for identification
- Single event handler for all toggle interactions

**API Integration:**

- Alert fetching: `https://api.weather.gov/alerts/active/zone/{zone}`
- Zone lookup: `https://api.weather.gov/points/{lat},{lon}` (extracts `forecastZone`)
- 10-second fetch timeout via `AbortSignal.timeout(10000)`
- Exponential backoff retry logic (max 3 attempts, starting at 5s delay)
- User-Agent header required: `Home Assistant Custom Card / {email}`

**Geolocation Features:**

- Mobile detection via `_isMobileDevice()`:
  - Home Assistant Companion app user agent detection
  - Mobile user agent patterns (android, iphone, etc.)
  - Screen width check (768px breakpoint)
- Coordinate resolution via `_resolveCoordinate()`:
  - Accepts numbers or entity ID strings
  - Extracts lat/lon from entity attributes
  - Validates coordinate ranges (-90 to 90 for lat, -180 to 180 for lon)
- Zone resolution via `_getActiveZone()`:
  - Priority: mobile lat/lon > base lat/lon > nws_zone fallback
  - Converts coordinates to NWS zones using Points API
  - Caches results for 24 hours (LRU eviction at 10 entries)
- Dynamic updates via `set hass()`:
  - Detects entity-based coordinate configs
  - Debounces zone re-resolution (5-second delay)
  - Triggers alert refresh when zone changes

**Rendering Approach:**

- Set-based comparison (`_setsEqual`) to detect alert changes
- Only re-renders when alert data actually changes
- Preserves expanded/collapsed state during updates using `_alertsCache`
- Uses `_escapeHtml` for XSS protection on all user-facing text

## Development Commands

### Testing Locally

No build process required. To test changes:

1. Edit `nws-alert-card.js` directly
2. Copy to Home Assistant: `/config/www/nws-alert-card.js`
3. Hard refresh browser (Cmd+Shift+R / Ctrl+Shift+R) to bypass cache
4. Check browser console (F12) for errors

### Validation

```bash
# HACS validation (runs in GitHub Actions)
# Uses hacs/action@main to validate plugin structure
```

## Configuration Constants

Located at top of constructor (nws-alert-card.js:12-15):

- `MAX_RETRIES = 3`: Maximum fetch retry attempts
- `BASE_RETRY_DELAY = 5000`: Initial retry delay in ms (doubles each attempt)
- `DESCRIPTION_THRESHOLD = 200`: Character limit before "Show more" toggle appears

## Security Considerations

**Input Sanitization:**

- Email addresses: Validated with regex, sanitized via `_sanitizeEmail()` (nws-alert-card.js:162-173)
- Zone format: Validated against `/^[A-Z]{2}[CZ]\d{3}$/` pattern
- All rendered text: Escaped via `_escapeHtml()` to prevent XSS

**External Links:**

- All external links use `rel="noopener noreferrer"` for security

## Severity Levels

Alerts are color-coded by severity (nws-alert-card.js:45-48):

- **Extreme** (red, #dc3545): Life-threatening situations (🔴🔴🔴 markers)
- **Severe** (orange, #fd7e14): Significant threat (🟠🟠 markers)
- **Moderate** (yellow, #ffc107): Possible threat
- **Minor** (green, #28a745): Minimal threat
- **Unknown** (gray): Severity not specified

## NWS API Details

**Zone Format:**

- Pattern: `SSZNNN` or `SSCNNN`
- `SS` = 2-letter state code (e.g., `WA`, `AK`)
- `Z` or `C` = Zone or County designator
- `NNN` = 3-digit number (e.g., `558`, `097`)
- Examples: `WAZ558`, `AKZ844`, `COZ097`

**Response Structure:**

- GeoJSON format with `features` array

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sxdjt/ha-nws-alert-card](https://github.com/sxdjt/ha-nws-alert-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
