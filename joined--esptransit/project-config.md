---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### ESP32-P4 Hardware

Use `scripts/idf.sh <board> [idf.py args...]` instead of `idf.py` directly — it handles board selection (`-DBOARD=<board>`), per-board build directories (`-B build_<board>`), and IDF environment setup automatically. Mise tasks `esp`, `esp-small`, and `esp-medium` are shortcuts for the three boards.

```bash
# JC8012P4A1C — large board, 800x1280 (default)
scripts/idf.sh jc8012p4a1c build flash monitor
# Same via mise task
mise run esp build flash monitor

# JC4880P443C — small board, 480x800
scripts/idf.sh jc4880p443c build flash monitor
# Same via mise task
mise run esp-small build flash monitor

# JC1060P470C — medium board, 1024x600 native landscape
scripts/idf.sh jc1060p470c build flash monitor
# Same via mise task
mise run esp-medium build flash monitor

# Flash/monitor only (no rebuild)
scripts/idf.sh jc8012p4a1c flash monitor
scripts/idf.sh jc4880p443c flash monitor
scripts/idf.sh jc1060p470c flash monitor

# Set target (already configured for esp32p4)
scripts/idf.sh jc8012p4a1c set-target esp32p4

# Menuconfig for SDK settings
scripts/idf.sh jc8012p4a1c menuconfig
# Same via mise task
mise run esp menuconfig
```

### Desktop Simulator

```bash
# Build and run the simulator (from project root)
mise run sim

# Build only (no run) — use this to verify compilation
mise run sim-build
```

### Python Scripts

The project venv (`.venv`) is **not** auto-activated. Python scripts that depend on project packages must be run via `uv run` (or their corresponding mise task), not `python3` directly:

```bash
uv run scripts/ui_golden_viewer.py    # or: mise run ui-golden-view
uv run scripts/regenerate_fonts.py    # or: mise run fonts-regen
```

This keeps the project venv separate from ESP-IDF's own Python venv.

## Architecture

This is a departure board display application for German rail/transit systems running on ESP32-P4 with a touchscreen. Three board variants are supported with different native resolutions (800x1280, 480x800, 1024x600).

### Code Sharing Design

The codebase is structured to share as much code as possible between the ESP32 hardware target and a desktop simulator. The goal is to make the simulator behave identically to the ESP while minimizing code duplication.

```
esptransit/
├── shared/              # Platform-agnostic code (compiles for both targets)
│   ├── app_manager.cpp  # Core state machine and command handling
│   ├── app_platform.h   # Platform abstraction interface
│   ├── http_client.h    # HTTP interface definition
│   └── ui/
│       ├── common.cpp/h        # Shared UI utilities, colors, fonts
│       └── screens/            # All LVGL screen classes
│           ├── screen_base.h   # Abstract base class for screens
│           ├── departures.*    # Departure board screen
│           ├── settings.*      # Settings screen
│           ├── station_search.*# Station search screen
│           └── wifi_setup.*    # WiFi setup screen
├── esp/                 # ESP32-P4 specific code
│   └── main/
│       ├── main.cpp           # ESP entry point
│       ├── app_platform_esp.cpp  # Platform impl (WiFi, NVS, SNTP)
│       └── http_client.cpp    # HTTP via esp_http_client
└── simulator/           # Desktop simulator
    └── src/
        ├── main.cpp              # SDL2/LVGL setup
        ├── app_platform_sim.cpp  # Platform impl (mock WiFi, JSON storage)
        └── mock/                 # Mock hardware abstractions
```

**Key patterns:**
- `AppManager` (shared) contains all state machine logic, screen transitions, and command handling
- `state_configs_` in `shared/app_manager.cpp` defines per-state screen lifecycle (`init`/`on_enter`/`on_exit`) and state-scoped timers
- Each screen is a class inheriting from `ScreenBase` (in `shared/ui/screens/screen_base.h`), constructed in the state config's `init` callback
- `AppPlatform` (interface in shared, impl per-target) abstracts WiFi, storage, and hardware operations
- All UI code lives in `shared/ui/screens/` and compiles identically for both targets
- The simulator uses FreeRTOS POSIX port so task/queue/notification code works unchanged

### Target Hardware

Three boards are supported, selectable via board Kconfig symbols (`CONFIG_ESPTRANSIT_BOARD_*`), typically through `SDKCONFIG_DEFAULTS` overlays:

| Feature | JC8012P4A1C (default) | JC4880P443C | JC1060P470C |
|---------|----------------------|-------------|-------------|
| Native resolution | 800x1280 (portrait) | 480x800 (portrait) | 1024x600 (landscape) |
| Native orientation | Portrait | Portrait | Landscape |
| LCD controller | JD9365 | ST7701 | JD9165 |
| Touch controller | GSL3680 (custom) | GT911 (standard) | GT911 (standard) |
| Build command | `mise run esp build` | `mise run esp-small build` | `mise run esp-medium build` |

All boards share:
- **MCU**: ESP32-P4 with companion ESP32-C6 for WiFi (via esp_wifi_remote)
- **Memory**: 16MB Flash, PSRAM enabled (XIP mode)

### State Machine
The app uses a simple state machine in `shared/app_manager.cpp`:
- **BOOT** → Storage version check → **WIFI_SETUP** (if no saved credentials) or **WIFI_CONNECTING**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joined/ESPTransit](https://github.com/joined/ESPTransit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
