---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CinePlete is a self-hosted Docker tool that scans Plex or Jellyfin movie libraries and identifies missing content:
- Missing movies from franchises (TMDB collections)
- Missing films from directors/actors in your library
- Classic films (TMDB Top Rated)
- Personalized suggestions based on library recommendations
- Metadata diagnostics (missing TMDB GUIDs)
- Radarr integration for adding movies
- Telegram notifications for scan results

## Development Commands

### Running Tests
```bash
# Unit tests (Python)
pytest --tb=short -v

# E2E tests (Playwright) — requires the app to be running on port 8787
cd e2e && npm ci && npx playwright install --with-deps chromium
BASE_URL=http://localhost:8787 npx playwright test
```

Unit tests are in `tests/` and cover:
- `test_config.py` - Config deep-merge and validation
- `test_overrides.py` - Ignore/wishlist persistence
- `test_scoring.py` - Movie scoring algorithms
- `test_scheduler.py` - Library polling logic
- `test_telegram.py` - Notification formatting

E2E tests are in `e2e/tests/` (Playwright + Chromium):
- `smoke.spec.js` - App loads, API endpoints respond (`/api/version`, `/api/scan/status`, `/api/config`)
- `navigation.spec.js` - All sidebar nav buttons present, `.active` class toggles correctly
- `config.spec.js` - Settings form fields, TMDB key masking, config save/reload roundtrip, modal

### Running Locally
```bash
# Install dependencies
pip install -r requirements.txt

# Run FastAPI server
uvicorn app.web:app --host 0.0.0.0 --port 8787
```

Access UI at `http://localhost:8787`

### Docker
```bash
# Build
docker build -t cineplete .

# Run
docker compose up -d
```

## Architecture

### Core Components

**Backend (FastAPI)**
- `app/web.py` - All API endpoints, lifespan management, scheduler startup
- `app/scanner.py` - 8-step scan engine with background threading and progress state
- `app/config.py` - Config loader/saver with deep-merge pattern (preserves defaults)
- `app/logger.py` - Rotating file logger (2 MB × 3 files)

**Media Server Scanners**
- `app/plex_xml.py` - Fast Plex XML API scanner (~2s for 1000 movies)
- `app/jellyfin_api.py` - Jellyfin API scanner (drop-in replacement for Plex)

Both scanners return identical tuple structure:
```python
(plex_ids, directors, actors, stats, no_tmdb_guid)
```

**TMDB Integration**
- `app/tmdb.py` - TMDB API client with persistent caching in `/data/tmdb_cache.json`

**Automation & Notifications**
- `app/scheduler.py` - APScheduler-based library polling (checks movie count, triggers scan if changed)
- `app/telegram.py` - Telegram notifications with rate limiting

**Data Management**
- `app/overrides.py` - Ignore/wishlist persistence helpers

**Frontend**
- `static/index.html` - Single-page app shell with inline CSS
- `static/app.js` - All UI logic (routing, rendering, API calls, Chart.js integration)

### Data Flow

```
1. Config loaded from /config/config.yml (deep-merged with defaults)
2. Scheduler starts on app boot, polls library every N minutes
3. User triggers scan → scanner.build_async() spawns background thread
4. Scanner runs 8 steps with progress updates:
   - Load config
   - Scan Plex/Jellyfin library (determined by SERVER.MEDIA_SERVER)
   - Validate TMDB metadata
   - Analyze collections (franchises)
   - Analyze directors
   - Analyze actors
   - Build suggestions (TMDB recommendations × library overlap)
   - Build results
5. Results written to /data/results.json
6. Frontend polls /api/scan/status for progress
7. Telegram notification sent if enabled
```

### Configuration System

Config is stored at `/config/config.yml` with a deep-merge pattern:
- `DEFAULT_CONFIG` in `app/config.py` defines all possible keys
- User config is deep-merged over defaults (missing keys auto-filled)
- `save_config()` always writes the full merged result
- `is_configured()` validates based on media server type (Plex vs Jellyfin)

**Media Server Selection**
- `SERVER.MEDIA_SERVER` - `"plex"` or `"jellyfin"`
- Scanner dynamically imports the appropriate module at runtime
- Configuration requirements differ:
  - Plex: PLEX_URL, PLEX_TOKEN, LIBRARY_NAME
  - Jellyfin: JELLYFIN_URL, JELLYFIN_API_KEY, JELLYFIN_LIBRARY_NAME

### Persistent Data (`/data/`)

| File | Purpose |
|------|---------|
| `results.json` | Full scan output (regenerated each scan) |
| `tmdb_cache.json` | TMDB API response cache (permanent) |
| `overrides.json` | Ignored items, wishlist, rec_fetched_ids |
| `cineplete.log` | Rotating log file (2 MB × 3 files) |
| `last_telegram.txt` | Timestamp of last Telegram notification |

### Scan State Management

`scanner.py` uses a shared `scan_state` dict with threading lock:
```python
scan_state = {
    "running": bool,
    "step": str,
    "step_index": int,
    "step_total": 8,
    "detail": str,
    "error": None,
    "last_completed": timestamp,
    "last_duration": seconds,
}
```

Only one scan can run at a time. Concurre
t requests are rejected.

## Key Patterns

### Deep Merge
Config uses a recursive deep-merge to preserve all default keys when user saves partial config. See `_deep_merge()` in `app/config.py`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdblepas/CinePlete](https://github.com/sdblepas/CinePlete) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
