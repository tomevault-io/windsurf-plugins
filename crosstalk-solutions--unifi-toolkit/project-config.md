---
trigger: always_on
description: FastAPI-based web dashboard for UniFi network management and monitoring. Deployed via Docker (Synology NAS, Unraid, etc.).
---

# UniFi Toolkit

## Overview
FastAPI-based web dashboard for UniFi network management and monitoring. Deployed via Docker (Synology NAS, Unraid, etc.).

## Tech Stack
- **Backend:** Python 3.9+, FastAPI, SQLAlchemy (async, SQLite), Alembic migrations, aiohttp (UniFi API)
- **Frontend:** Jinja2 templates, vanilla JS (main dashboard), Alpine.js (Network Pulse)
- **Auth:** Optional session-based auth (production mode)

## Architecture
```
app/
├── main.py              # FastAPI app, /api/system-status endpoint, lifespan
├── __init__.py          # __version__
├── routers/             # auth, config endpoints
├── templates/           # Jinja2 (dashboard.html is the main UI)
├── static/              # CSS, JS assets
shared/
├── unifi_client.py      # UniFi API client (1800+ lines) — core data fetching
├── unifi_session.py     # Shared singleton session management
├── database.py          # Async SQLite via SQLAlchemy
├── cache.py             # In-memory cache with TTL
├── config.py            # Settings via environment
├── crypto.py            # Password/API key encryption
tools/
├── wifi_stalker/        # Client tracking tool
├── threat_watch/        # IDS/IPS monitoring
├── network_pulse/       # Network health dashboard (Alpine.js frontend)
```

## Key Patterns

### Version Management
Version is maintained in THREE files — keep them in sync:
- `pyproject.toml` → `version = "X.Y.Z"`
- `app/__init__.py` → `__version__ = "X.Y.Z"`
- `app/main.py` → `version="X.Y.Z"` (FastAPI constructor)

### UniFi API Client (`shared/unifi_client.py`)
- **UniFi OS only** — legacy standalone controller support was removed in v1.11.0 (aiounifi dependency dropped)
- All API calls use `/proxy/network/api/` prefix via aiohttp
- Health endpoint returns subsystems: wan, wan2+, lan, wlan, vpn, www
- WAN detection is dynamic via `startswith('wan')` — supports N WANs
- **Signal strength:** UniFi API returns separate `rssi` and `signal` fields — use `signal` (matches console display) with `rssi` fallback
- **v2 traffic-flows payload:** The v2 endpoint supports a filtered payload format with `pageNumber`/`pageSize`/`timestampFrom`/`timestampTo` and a `policy_type` array for server-side filtering (e.g., `["INTRUSION_PREVENTION"]` for IPS-only events). The old `limit`/`offset`/`timeRange` format returns ALL flows unfiltered. Auto-detection via `_v2_uses_new_payload` flag handles both formats.

### Schema Repair (`run.py → _repair_schema()`)
- Runs on every startup after Alembic migrations
- Safety net for when `create_all` causes Alembic to stamp-to-head, skipping ADD COLUMN ops
- Must cover ALL migration-added columns — when adding a new Alembic migration that adds a column, also add it to `_repair_schema()`
- Uses `_add_missing_columns()` helper — pass table name and dict of `{col_name: col_sql}`

### Threat Watch Retention & Purge
- Events older than 30 days are auto-purged by the scheduler (runs at most once per hour, piggybacks on the 60s refresh cycle)
- `RETENTION_DAYS` and `PURGE_INTERVAL_SECONDS` constants in `tools/threat_watch/scheduler.py`
- Frontend defaults to 7-day view via `time_range` filter; backend supports `24h`, `7d`, `30d`

### Debug Info (`/api/debug-info`)
- Returns non-sensitive system info (versions, deployment, gateway, firmware) for issue reporting
- Dashboard footer has "Debug Info" link → modal with copy-to-clipboard
- "Report Issue" link also uses this endpoint to pre-populate GitHub issues

### Firmware Compatibility
- **Only stable/GA UniFi firmware is supported** — Early Access (EA) firmware frequently changes API endpoints without notice
- Do NOT suggest users switch firmware channels or attempt to support EA builds
- When users report API issues, firmware version is the first thing to check (now included in debug info)

### Data Flow (Dashboard)
```
UniFi Controller → unifi_client.py (get_health, get_system_info)
  → /api/system-status endpoint (main.py)
  → dashboard.html JS (fetches every 60s)
```

### Threat Watch Data Flow
```
UniFi Controller → get_traffic_flows() → _normalize_v2_event() (flattens to legacy field names)
  → get_ips_events() returns normalized events
  → scheduler.parse_unifi_event() → _parse_legacy_ips_event() (single parser for both v2 and legacy)
  → ThreatEvent DB model
```
All v2 events are normalized before the scheduler sees them — the scheduler only has one parser.

### Network Pulse
- Uses its own scheduler for background polling
- Alpine.js for reactive frontend
- Models in `tools/network_pulse/models.py`
- Extra WANs stored in `NetworkHealth.extra_wans` dict

## Completed Work

### v1.11.2
- Fix Network Pulse chart panels not resizing responsively (#96) — `min-width: 0` on `.chart-card` and `overflow: hidden` on `.chart-container` fix CSS Grid min-width:auto gotcha that prevented canvas-based chart cards from shrinking on narrow viewports
- Remove legacy standalone controller references from README and INSTALLATION.md (#97) — added UniFi OS requirement callout, removed port 8443 examples, lifted Python 3.13 restriction, reordered auth to lead with API key

### v1.11.1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crosstalk-Solutions/unifi-toolkit](https://github.com/Crosstalk-Solutions/unifi-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
