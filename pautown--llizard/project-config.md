---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

llizardgui-host is a plugin-based GUI host application for the Spotify CarThing device. It uses raylib/raygui for graphics and provides an SDK that plugins use to draw UIs with a unified input/display abstraction.

**Target Platform:** ARM Linux (Spotify CarThing - Cortex-A7, ARMv7, hard float ABI)
**Graphics:** raylib with DRM backend for CarThing, standard raylib for desktop
**Display:** 800×480 logical resolution (SDK handles DRM rotation)

## Build Commands

### Desktop Build (Development)
```bash
mkdir -p build && cd build
cmake ..
make -j$(nproc)
```

### Cross-Compilation for CarThing (DRM)
```bash
mkdir -p build-armv7-drm && cd build-armv7-drm
cmake -DCMAKE_TOOLCHAIN_FILE=../toolchain-armv7.cmake -DPLATFORM=DRM ..
make -j$(nproc)
```

### Build Outputs
- `build/llizardgui-host` - Desktop executable
- `build/nowplaying.so`, `build/redis_status.so` - Plugins copied to `plugins/` automatically
- `build-armv7-drm/llizardgui-host` - ARM binary for CarThing
- `build-armv7-drm/*.so` - ARM plugins

### Deployment
CarThing device: IP `172.16.42.2`, user `root`, password `llizardos`
```bash
scp build-armv7-drm/llizardgui-host root@172.16.42.2:/tmp/
scp build-armv7-drm/nowplaying.so root@172.16.42.2:/usr/lib/llizard/plugins/
```

## Architecture

### Host Application (`src/`)
- **main.c** - Plugin menu and main loop using SDK display/input functions
- **plugin_loader.c** - Dynamic loading of `.so` plugins from `./plugins/` directory via `dlopen`

### Plugin API (`include/llizard_plugin.h`)
Plugins export `LlzGetPlugin()` returning a `LlzPluginAPI` struct with callbacks:
- `init(width, height)` - Called when plugin is selected
- `update(input, deltaTime)` - Per-frame update with input state
- `draw()` - Render plugin content
- `shutdown()` - Cleanup when exiting plugin
- `wants_close()` - Optional signal to return to menu

### SDK (`sdk/`)
Abstracts raylib/DRM platform differences:
- **llz_sdk_display.h** - `LlzDisplayInit/Begin/End/Shutdown` for render target management
- **llz_sdk_input.h** - `LlzInputState` with buttons, touch, gestures, scroll wheel
- **llz_sdk_layout.h** - Helper functions for layout math
- **llz_sdk_media.h** - Redis-backed media state access and playback commands
- **llz_sdk_image.h** - Blur effects and cover/contain image scaling
- **llz_sdk_config.h** - Global and per-plugin configuration
- **llz_sdk_background.h** - 9 animated background styles
- **llz_sdk_subscribe.h** - Event callbacks for media changes
- **llz_sdk_navigation.h** - Plugin-to-plugin navigation requests
- **llz_sdk_font.h** - Centralized font loading with automatic path resolution
- **llz_sdk_connections.h** - Service connection status (Spotify auth, etc.)

Plugins include `llz_sdk.h` and receive `const LlzInputState*` in their update callback.

### Fonts (`fonts/`)
Centralized font storage for consistent text rendering:
- **ZegoeUI-U.ttf** - Primary UI font (CarThing system font)
- Copy fonts from CarThing: `./scripts/copy-carthing-fonts.sh`
- SDK searches: `/var/local/fonts/`, `/tmp/fonts/`, `./fonts/`, system fonts
- See `fonts/README.md` for detailed font API usage

### Shared Libraries (`shared/`)

**Notification System** (`shared/notifications/`)
- Reusable popup notifications that plugins can opt-in to use
- Three styles: Banner (top/bottom bar), Toast (corner popup), Dialog (modal with buttons)
- Queue system for sequential display
- Callbacks for tap, dismiss, timeout, button press
- Built-in plugin navigation on tap
- Usage: `#include "llz_notification.h"` and link with `llz_notifications`

### Available Plugins

**nowplaying** (`plugins_src/nowplaying/`)
- Music player UI with multiple display modes (ADVANCED, NORMAL, BAREBONES, ALBUM_ART)
- Theme system with multiple color variants
- Clock overlay, progress scrubbing, playback controls
- Modular structure: `core/`, `widgets/`, `screens/`, `overlays/`

**redis_status** (`plugins_src/redis_status/`)
- Displays live Redis/MediaDash state from the golang_ble_client daemon
- Two-column layout: Connection status (left), Now Playing media (right)
- Header with Redis/BLE connection indicators
- Requires Redis running on CarThing (`sv start redis`)
- Uses SDK's `LlzMedia*` APIs to fetch state from Redis

**spotify** (`plugins_src/spotify/`)
- Full-featured Spotify control interface leveraging Janus Android companion app
- Three screens: Hub (now playing), Queue (browser with skip-to), Controls (shuffle/repeat/like/volume)
- Spotify-themed color palette (green accent on dark background)
- Album art display with automatic loading
- Connection status indicator for Spotify auth
- Swipe/scroll navigation between screens
- Track change detection with automatic queue refresh
- **Shuffle/Repeat/Like controls** via Spotify Web API:
  - Toggle shuffle on/off
  - Cycle repeat modes (off → track → context → off)
  - Like/unlike current track (saves to Spotify library)
- Requires Spotify OAuth authentication in Janus companion app

**albums** (`plugins_src/albums/`)
- Browse saved Spotify albums in a horizontal carousel
- Large album art cards with smooth spring-based scrolling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pautown/llizard](https://github.com/pautown/llizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
