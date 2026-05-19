---
trigger: always_on
description: A cross-platform Python application that automatically tracks media playback from various media players and scrobbles to Simkl.
---

# Media Player Scrobbler for Simkl (MPS) - Copilot Instructions

A cross-platform Python application that automatically tracks media playback from various media players and scrobbles to Simkl.

## Build, Test, and Lint Commands

### Development Setup
```bash
# Install dependencies with Poetry (recommended)
poetry install --with dev

# Or with pip
pip install -e ".[dev]"
```

### Testing
```bash
# Run all tests
poetry run pytest

# Run a single test file
poetry run pytest test_potplayer.py

# Run specific test function
poetry run pytest test_potplayer.py::test_potplayer_integration
```

### Linting
```bash
# Run flake8 on the codebase
poetry run flake8 simkl_mps/
```

### Building Executables
```bash
# Build with PyInstaller (uses simkl-mps.spec)
poetry run pyinstaller simkl-mps.spec

# Verify build output
python test_build.py [windows|macos|linux]
```

The build produces two executables:
- `MPSS.exe` / `MPSS` - Short executable name
- `MPS for Simkl.exe` / `MPS for Simkl` - Full name executable

### Publishing
```bash
poetry build
poetry publish
```

## Architecture Overview

### Core Components

The application follows a modular architecture with clear separation of concerns:

```
MediaTracker (coordinator)
    └── Monitor (polling & player detection)
        └── MediaScrobbler (scrobbling logic)
            ├── SimklAPI (API communication)
            ├── MediaCache (offline caching)
            ├── BacklogCleaner (offline queue)
            ├── WatchHistoryManager (local history)
            └── Player Integrations (position tracking)
```

### Key Modules

**Monitor** (`monitor.py`)
- Polls active windows to detect media players
- Coordinates player integrations for position/duration tracking
- Manages the main event loop

**MediaScrobbler** (`media_scrobbler.py`)
- Core scrobbling logic for movies, TV shows, and anime
- Handles both position-based and time-based progress tracking
- Manages offline queue when internet is unavailable
- Determines completion threshold (default 80%)

**Window Detection** (`window_detection.py`)
- Platform-specific window detection (Windows, macOS, Linux)
- Parses media titles from window titles
- Maintains list of known media player executables
- Uses `guessit` library for intelligent filename parsing

**Player Integrations** (`players/`)
- Each player has its own integration class
- Common interface: `get_position_duration()` returns `(position_seconds, duration_seconds)` or `(None, None)`
- Supported: VLC, PotPlayer, MPV, MPC-HC, MPC-BE, MPC-QT, and MPV wrappers (mpvnet, syncplay, etc.)

**Simkl API** (`simkl_api.py`)
- All API communication with Simkl
- Handles authentication, search, and scrobbling
- Internet connectivity checks

**MediaCache** (`media_cache.py`)
- Caches movie/show details to reduce API calls
- JSON-based persistent storage

**BacklogCleaner** (`backlog_cleaner.py`)
- Queues scrobbles when offline
- Retries on reconnection

**Tray Application** (`tray_app.py`, `tray_*.py`)
- Platform-specific system tray implementations
- Status display and user controls

### Data Flow

1. **Detection**: `Monitor` polls active windows → identifies media player
2. **Parsing**: Window title/filepath → `guessit` → parsed media info
3. **Search**: Parsed info → `SimklAPI.search_*` → Simkl ID
4. **Tracking**: Player integration provides position → `MediaScrobbler` tracks progress
5. **Scrobbling**: At completion threshold → mark as watched on Simkl
6. **Offline**: If no internet → queue in `BacklogCleaner` → retry later

## Key Conventions

### Platform-Specific Code
- Always check `PLATFORM = platform.system().lower()` for OS-specific logic
- Windows uses `pygetwindow` and `win32gui`
- Linux uses `Xlib` (when X11 is available)
- macOS uses `subprocess` with AppleScript
- Platform-specific tray apps: `tray_win.py`, `tray_linux.py`, `tray_mac.py`

### Player Integration Pattern
When adding a new player:
1. Create class in `players/new_player.py` with `get_position_duration()` method
2. Add to `players/__init__.py` exports
3. Add executable name to `VIDEO_PLAYER_EXECUTABLES` in `window_detection.py`
4. Instantiate in `Monitor.__init__()`

### Configuration Management
- Settings stored in `.simkl_mps.env` in app data directory
- Use `config_manager.get_setting()` to read settings
- Data folder locations vary by platform (see `docs/configuration.md`)
- Access via tray menu: "Maintenance → Open Data Folder"

### Testing Considerations
- Tests use `pytest` with `pytest-mock` for mocking
- Build verification with `test_build.py` (platform-specific)
- Player integration tests (e.g., `test_potplayer.py`) mock HTTP requests
- Use `testing_mode=True` parameter to disable real API calls

### Offline Support
- All scrobbling attempts are queued in `backlog.json` when offline
- `BacklogCleaner` retries items on reconnection
- `MAX_BACKLOG_ATTEMPTS = 5` constant defines retry limit
- Items are marked in `_processing_backlog_items` set during processing to prevent duplicates

### Media Type Handling
- `media_type` can be: `'movie'`, `'episode'`, `'show'`, or `'anime'`
- `guessit` provides initial type detection from filename
- Simkl API returns canonical type and metadata

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ByteTrix/Media-Player-Scrobbler-for-Simkl](https://github.com/ByteTrix/Media-Player-Scrobbler-for-Simkl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
