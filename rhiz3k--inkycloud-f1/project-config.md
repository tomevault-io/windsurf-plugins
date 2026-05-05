---
trigger: always_on
description: This is a FastAPI service that generates **800x480 1-bit BMP images** for E-Ink displays (specifically LaskaKit ESP32 devices). The service fetches F1 race data from Jolpica API, converts times from UTC to Europe/Prague timezone, and renders server-side calendar images using Pillow.
---

# Copilot Instructions for F1 E-Ink Calendar

## Architecture Overview

This is a FastAPI service that generates **800x480 1-bit BMP images** for E-Ink displays (specifically LaskaKit ESP32 devices). The service fetches F1 race data from Jolpica API, converts times from UTC to Europe/Prague timezone, and renders server-side calendar images using Pillow.

**Key components:**
- `app/main.py` - FastAPI endpoints with async/await pattern
- `app/config.py` - Configuration management from environment variables
- `app/models.py` - Pydantic data models
- `app/state.py` - Application state management
- `app/services/renderer.py` - Pixel-perfect 1-bit BMP rendering engine
- `app/services/spectra6_renderer.py` - Spectra6 multi-color E-Ink renderer
- `app/services/f1_service.py` - Jolpica API client with timezone conversion
- `app/services/teams_service.py` - Teams & drivers data management
- `app/services/standings_service.py` - Championship standings data
- `app/services/weather_service.py` - Weather forecast integration
- `app/services/database.py` - SQLite operations for data persistence
- `app/services/scheduler.py` - APScheduler background jobs
- `app/services/backup.py` - S3 database backup automation
- `app/services/i18n.py` - Translation loader with caching
- `app/services/analytics.py` - Fire-and-forget Umami tracking
- `app/services/version_service.py` - Version management
- `translations/*.json` - i18n strings for cs/en

## Critical Patterns

### 1-Bit Rendering (Must Follow Exactly)

All images MUST be 1-bit mode (`Image.new("1", ...)`) for E-Ink compatibility. Never use "L" or "RGB" modes.

```python
# ✓ Correct
image = Image.new("1", (800, 480), 1)  # 1 = white background

# ✗ Wrong
image = Image.new("RGB", (800, 480), (255, 255, 255))
```

When drawing, use `fill=0` for black and `fill=1` for white. The renderer uses pixel-precise layout constants in `self.layout` dict - **never hardcode coordinates**.

### Timezone Handling (Prague-Specific)

ALL race times are stored in UTC in Jolpica API and MUST be converted to `Europe/Prague` timezone. See `F1Service._convert_race_times()` for the canonical pattern:

```python
dt_utc = datetime.fromisoformat(dt_str.replace("Z", "+00:00"))
dt_prague = dt_utc.astimezone(self.prague_tz)
```

Display format: `dt_prague.strftime("%a %H:%M")` (e.g., "Sun 17:00")

### Translation Keys

Always use `translator.get(key, fallback)` pattern. Session names use prefix `session_` (e.g., `session_race`, `session_qualifying`). See [translations/en.json](../translations/en.json) for all keys.

### Error Handling

Endpoints NEVER raise - always return a rendered error BMP via `renderer.render_error()`. Exceptions are logged and sent to Sentry/GlitchTip:

```python
except Exception as e:
    logger.error(f"Error: {e}", exc_info=True)
    sentry_sdk.capture_exception(e)
    return StreamingResponse(BytesIO(renderer.render_error(str(e))), ...)
```

### Async Patterns

- HTTP calls use `httpx.AsyncClient` (never `requests`)
- Analytics tracking is fire-and-forget: `asyncio.create_task(_send_analytics(...))`
- FastAPI endpoints are `async def` with proper context managers

## Development Commands

```bash
# Setup environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -e ".[dev]"

# Local dev (auto-reload)
uvicorn app.main:app --reload

# Run with debug logging
DEBUG=true python -m app.main

# Test suite (must pass before PR)
pytest

# Run tests with coverage
pytest --cov=app tests/

# Lint & format (CI enforced)
ruff check .
ruff format .
```

## Testing Requirements

1. All renderer changes MUST include tests in `tests/test_renderer.py`
2. Tests verify exact BMP properties: 800x480, mode="1", format="BMP"
3. Use `mock_race_data` fixture pattern for consistent test data
4. Test both Czech (`cs`) and English (`en`) translations

Example test structure:
```python
def test_new_feature(mock_race_data):
    translator = get_translator("en")
    renderer = Renderer(translator)
    bmp_data = renderer.render_calendar(mock_race_data)
    
    img = Image.open(BytesIO(bmp_data))
    assert img.mode == "1"
    assert img.size == (800, 480)
```

## Configuration Philosophy

All config comes from environment variables via `app/config.py`. Never hardcode:
- API URLs (use `config.JOLPICA_API_URL`)
- Display dimensions (use `config.DISPLAY_WIDTH/HEIGHT`)
- Monitoring DSNs (use `config.SENTRY_DSN`)

Feature flags like `UMAMI_ENABLED` control optional services - code must handle disabled features gracefully.

## Adding Translations

1. Add key to `translations/en.json` (source of truth)
2. Add same key to `translations/cs.json`
3. Use in code: `translator.get("your_key", "Fallback Text")`
4. Update `app/main.py` language validation if adding new locale

**Adding a New Language (e.g., German):**
1. Create `translations/de.json` with all keys from `en.json`
2. Update language validation in `app/main.py`:
   ```python
   if lang not in ["cs", "en", "de"]:
   ```
3. Test both translations render correctly:
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rhiz3K/InkyCloud-F1](https://github.com/Rhiz3K/InkyCloud-F1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
