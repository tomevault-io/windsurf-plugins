---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HSG Canvas (Hackerspace.gent Canvas) - a FastAPI application that manages media streaming and playback on a Raspberry Pi. Runs alongside an SRS (Simple Realtime Server) to display streams, images, and YouTube content on a connected display. The Pi acts as both a media player and a stream republisher.

## Development Commands

```bash
# Install dependencies
pip3 install -r requirements.txt

# Run server (development, port 8000)
python3 main.py

# Run server (production, port 8000 — Angie proxies port 80)
python3 main.py --production

# Run via start script (starts Vite + FastAPI in production)
./start.sh

# One-time setup (installs Angie, systemd services, raspotify config)
sudo ./setup.sh

# Run tests
pytest tests/
```

## Architecture

### Application Wiring

`main.py` uses FastAPI's lifespan context manager to initialize everything in order:

1. **DisplayCapabilityDetector** - detects connected display resolution via DRM
2. **WebSocket managers** (3 instances) - for Spotify events, display state, and audio commands
3. **DisplayStack** - core display abstraction (base layer + stack of items)
4. **ChromiumManager** - starts Chromium once at boot, never stops
5. **BackgroundManager** - thin wrapper around DisplayStack
6. **AudioManager** - controls browser `<audio>` via WebSocket (no MPV)
7. **PlaybackManager** - pushes YouTube to DisplayStack (browser renders via IFrame API)
8. **All other managers** - each receives its dependencies via constructor injection
9. **API routes** - each `setup_*_routes()` function creates an `APIRouter` with manager references

Shutdown reverses this order. All managers are global module-level variables set during lifespan.

### Key Directories

- **`managers/`** - All business logic (there is no separate `core/` directory)
- **`models/`** - Pydantic request models (`request_models.py`)
- **`routes.py`** - Single file with 12 `setup_*_routes()` functions, each returning an `APIRouter`
- **`config.py`** - Constants: paths, network/discovery, server ports
- **`config/`** - Deployment config files (Angie, systemd service, raspotify drop-in)
- **`background_engine/`** - Advanced background image generation (layout, components, generators)
- **`tests/`** - pytest tests with pytest-asyncio for async testing
- **`static/`** - CSS/JS for the web interface
- **`index.html`** - Synthwave-themed web control panel (served at `/`)

### Manager Interactions

Managers reference each other for coordinated behavior:
- **PlaybackManager** depends on: DisplayStack, DisplayCapabilityDetector, BackgroundManager, AudioManager
- **ChromecastManager** depends on: AudioManager, PlaybackManager (stops local playback when casting)
- **OutputTargetManager** depends on: AudioManager, PlaybackManager, ChromecastManager (unified target interface)
- **SpotifyManager** depends on: AudioManager (stops audio streams when Spotify plays)
- **BackgroundManager** depends on: DisplayStack, DisplayCapabilityDetector

### Route Pattern

Each route group in `routes.py` follows the same pattern:
```python
def setup_X_routes(manager, ...) -> APIRouter:
    router = APIRouter(prefix="/X", tags=["X"])
    # Endpoint definitions using the injected manager
    return router
```

Routes are included in the app during lifespan startup via `app.include_router()`.

## Configuration (`config.py`)

- **DEFAULT_BACKGROUND_PATH**: Path to default background image
- **DEVICE_NAME**: "HSG Canvas" (used in SSDP/Chromecast discovery)
- **CHROMECAST_CACHE_DURATION**: 24 hours
- **METADATA_UPDATE_INTERVAL**: 15 seconds
- **DEFAULT_PORT / PRODUCTION_PORT**: Both 8000 (Angie proxies port 80)

## Hardware Context

This runs on a Raspberry Pi with:
- Chromium kiosk mode for display output
- PipeWire/PulseAudio for audio (shared with Raspotify service)
- Temperature monitoring via `/sys/class/thermal/thermal_zone0/temp`
- HDMI-CEC for TV power control

## Important Patterns

- **Subprocess isolation**: Chromecast discovery runs in a subprocess to prevent zeroconf file descriptor leaks
- **Audio exclusivity**: Only one audio source at a time (audio stream, Spotify, or video with audio)
- **YouTube via IFrame API**: Video ID extracted by regex, rendered in browser (no yt-dlp)

## Critical Gotchas

### setup.sh overwrites config files
`setup.sh` is a full system provisioning script. Running it **replaces** `/etc/raspotify/conf` and systemd service files. If you add config to those files outside of setup.sh, the next setup.sh run will erase it. **Always update setup.sh when changing deployment config.**

Key things setup.sh must preserve:
- `LIBRESPOT_ONEVENT=/home/hsg/srs_server/raspotify-onevent.sh` in `/etc/raspotify/conf` (without this, Spotify events don't reach FastAPI and the display never updates)
- Raspotify systemd drop-in at `config/raspotify/onevent.conf` (User=hsg, PrivateUsers=false, ProtectHome=false)

### Raspotify + PipeWire/PulseAudio

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0x20) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
