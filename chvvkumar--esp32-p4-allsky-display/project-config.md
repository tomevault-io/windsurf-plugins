---
trigger: always_on
description: ESP32-P4 firmware for displaying all-sky camera images with hardware-accelerated transforms. Uses DSI display, PPA (Pixel Processing Accelerator), MQTT/Home Assistant integration, and OTA updates with A/B partitions.
---

# ESP32-P4 AllSky Display - AI Coding Agent Guide

## Project Overview
ESP32-P4 firmware for displaying all-sky camera images with hardware-accelerated transforms. Uses DSI display, PPA (Pixel Processing Accelerator), MQTT/Home Assistant integration, and OTA updates with A/B partitions.

## Architecture

### Core Components (modular .cpp/.h pairs)
- **display_manager**: Arduino_GFX wrapper for DSI panel, brightness control, debug overlay
- **ppa_accelerator**: ESP32-P4 hardware scaling/rotation (PPA engine), DMA-aligned buffers
- **network_manager**: WiFi, NTP, ArduinoOTA, captive portal fallback
- **mqtt_manager**: PubSubClient wrapper, HA auto-discovery, status publishing
- **config_storage**: NVS persistence (Preferences), multi-image sources, transforms
- **web_config**: WebServer (port 8080) + WebSocket console (port 81), ElegantOTA at `/update`
- **crash_logger**: RTC memory + NVS crash dumps, backtrace preservation, NTP timestamps
- **system_monitor**: Watchdog management, memory monitoring, task retry handler
- **captive_portal**: WiFi setup AP mode with QR code, DNS redirects

### Data Flow
1. **Image acquisition**: HTTP download → JPEG decode (JPEGDEC) → RGB565 buffer (`pendingFullImageBuffer`)
2. **Transform pipeline**: PPA scale/rotate → `scaledBuffer` → display framebuffer via `draw16bitRGBBitmap()`
3. **Double buffering**: Download to `pendingFullImageBuffer` while `fullImageBuffer` is displayed (flicker-free)

### Memory Layout (PSRAM critical)
- `imageBuffer`: Download scratch (1x display, ~1.28MB)
- `fullImageBuffer`: Current displayed image (4MB, supports 1448×1448 max)
- `pendingFullImageBuffer`: Next image being prepared (4MB double-buffer)
- `scaledBuffer`: PPA output (4x display = 2.0× max scale, ~5.12MB)
- **Order matters**: Buffers allocated in `setup()` *before* display init to ensure contiguous PSRAM

## Critical Conventions

### Display Configuration
- **Compile-time selection**: `displays_config.h` defines `CURRENT_SCREEN` (3.4" vs 4.0" display)
- **GFX Library patch required**: `Arduino_ESP32DSIPanel.cpp` must change `MIPI_DSI_PHY_CLK_SRC_DEFAULT` to `MIPI_DSI_PHY_PLLREF_CLK_SRC_PLL_F20M` (ESP-IDF 5.5+ type mismatch fix)
- **Auto-applied by script**: `compile-and-upload.ps1` patches library before compile

### Watchdog Management
- **30-second timeout**: Set via `configStorage.getWatchdogTimeout()` in `setup()`, accommodates slow HTTP downloads
- **RAII pattern**: Use `WATCHDOG_SCOPE()` macro (from `watchdog_scope.h`) in long-running functions - auto-resets on entry/exit
- **Manual resets**: `systemMonitor.forceResetWatchdog()` for loops or operations >30s

### Logging System
- **Always use LOG macros**: `LOG_INFO()`, `LOG_ERROR()`, etc. (not `Serial.print*`) - routes to Serial + WebSocket console
- **Severity levels**: `LOG_DEBUG` (hidden by default) → `LOG_INFO` → `LOG_WARNING` → `LOG_ERROR` → `LOG_CRITICAL`
- **Remote debugging**: WebSocket console at `http://[ip]:8080/console` streams all logs with severity filtering

### Configuration Management
- **Global instance**: `configStorage` (Preferences wrapper), always `saveConfig()` after writes
- **Multi-image support**: Up to 10 URLs in `imageSourceCount` array, cycling controlled by `cyclingEnabled` + `randomOrderEnabled`
- **Per-image transforms**: Use `config_storage.h` structs for scale/offset/rotation per image index

### OTA Safety
- **Partition scheme**: `partitions.csv` defines 10MB app0/app1 (A/B), bootloader handles rollback
- **Never block during OTA**: Display paused, watchdog extended, config preserved in NVS
- **Progress display**: `displayManager.showOTAProgress()` draws to framebuffer without disrupting upload

## Build Workflow

### PowerShell Script (Preferred)
```powershell
# If device is connected via USB (serial available)
.\compile-and-upload.ps1 -ComPort COM3

# If device is NOT connected to serial (compile only, then OTA)
.\compile-and-upload.ps1 -SkipUpload -OutputFolder "C:\Users\Kumar\Desktop\New folder"
```
**Auto-handles**: GFX library patching, `build_info.h` git hash injection, Arduino CLI detection, memory size reporting

**Choose method based on connection:**
- USB connected → Upload directly via serial (faster for first flash)
- Network only → Compile binary, then upload via ElegantOTA at `http://allskyesp32.lan:8080/update`

### Arduino IDE Manual Setup
- **Board**: ESP32-P4-Function-EV-Board
- **PSRAM**: Enabled (required, fail compilation if missing)
- **Partition**: 13MB app / 7MB data (32MB) - uses `partitions.csv` in sketch folder
- **Flash**: 32MB, 921600 baud
- **Apply GFX patch manually** (see `docs/02_installation.md` - Compiling From Source section)

### Git Build Info
- `build_info.h` auto-generated with `GIT_COMMIT_HASH`, `GIT_BRANCH`, `BUILD_DATE` - committed as template, updated at compile time

## Common Patterns

### Adding a New Serial Command
1. Add to `command_interpreter.h` command table
2. Implement handler in `command_interpreter.cpp` 
3. Update help text in `printHelp()`
4. Use `LOG_INFO_F()` for feedback (appears in Serial + WebSocket console)

### Adding Web Config Parameter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chvvkumar/ESP32-P4-Allsky-Display](https://github.com/chvvkumar/ESP32-P4-Allsky-Display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
