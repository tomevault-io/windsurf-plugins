---
trigger: always_on
description: M5Gotchi is a Pwnagotchi implementation for M5Cardputer—a portable hacking device running Arduino-based firmware with a full keyboard UI. The project bridges manual Wi-Fi penetration testing with social networking features (PWNGrid). **Key constraint**: This is resource-constrained embedded C++ (ESP32-S3), not general software.
---

# ESPBlaster/M5Gotchi Copilot Instructions

## Project Overview

M5Gotchi is a Pwnagotchi implementation for M5Cardputer—a portable hacking device running Arduino-based firmware with a full keyboard UI. The project bridges manual Wi-Fi penetration testing with social networking features (PWNGrid). **Key constraint**: This is resource-constrained embedded C++ (ESP32-S3), not general software.

## Architecture

### Core Layers

1. **Hardware Abstraction** ([M5Cardputer lib](lib/M5Cardputer/), M5Unified)
   - Manages display (`M5Canvas`), keyboard input, speaker/buzzer, SD card, and device-specific GPIO
   - Use `M5.update()` and `M5Cardputer.update()` to poll input
   - Display operations must be protected by `displayMutex` semaphore to prevent race conditions

2. **UI System** ([src/ui.cpp](src/ui.cpp), [src/ui.h](src/ui.h))
   - **Canvas-based drawing**: All UI uses `canvas_main` (M5Canvas sprite) with `pushAll()` to flush to hardware
   - **Global state**: `menuID` controls which screen is active; `appID` drives app execution via `runApp()`
   - **Font handling**: Custom fonts loaded from SD (`/fonts/big.vlw`, `/fonts/small.vlw`) using `canvas_main.loadFont(FSYS, path)`. Critical: **Always unload fonts after use** with `canvas_main.unloadFont()` before loading another
   - **Input handling**: Keyboard (M5Cardputer) or button-only input (M5StickS3) via `#ifdef BUTTON_ONLY_INPUT`
   - **Dialog functions**: `drawInfoBox()`, `drawQuestionBox()`, `drawMultiChoice()` for blocking UX; `drawHintBox()` with bitfield tracking to show hints once

3. **Core Features**
   - **Pwnagotchi** ([src/pwnagothi.cpp](src/pwnagothi.cpp)): Wi-Fi scanning, handshake capture, mood system
   - **PWNGrid** ([src/pwngrid.cpp](src/pwngrid.cpp)): P2P mesh messaging with cryptographic identity; uses JSON messages
   - **Storage**: Unified via `FSYS` macro—either LittleFS (m5sticks3) or SD (Cardputer). See [src/settings.h](src/settings.h) for `#ifdef USE_LITTLEFS`
   - **Wardriving** ([src/wardrive.cpp](src/wardrive.cpp)): GPS integration for location-based Wi-Fi logging
   - **WPA-Sec** ([src/wpa_sec.cpp](src/wpa_sec.cpp)): Encrypted password database upload

### Critical Global Resources

- **`displayMutex`** ([src/ui.h](src/ui.h)): **Always take before any canvas drawing**. Use `if (displayMutex) xSemaphoreTake(displayMutex, portMAX_DELAY);` and release with `xSemaphoreGive(displayMutex)`
- **`FSYS` macro** ([src/settings.h](src/settings.h)): Conditional SD/LittleFS abstraction—use this, not `SD` or `LittleFS` directly
- **`logMessage(String)`** ([src/logger.cpp](src/logger.cpp)): Thread-safe serial logging for debugging

## Build & Environments

**PlatformIO-based** with three target environments in [platformio.ini](platformio.ini):

- **Cardputer-dev / Cardputer-full**: M5Stack CardPuter (full keyboard, 8MB flash)
- **m5sticks3**: M5StickS3 (LittleFS only, minimal memory, button-only input)

### Build Commands

```bash
# Standard build (Cardputer-dev)
pio run

# Build all environments
pio run --environment Cardputer-dev && pio run --environment Cardputer-full && pio run --environment m5sticks3

# Upload to device
pio run --target upload --environment Cardputer-dev

# Monitor serial output
pio device monitor -b 115200

# With MQTT coredump logging (requires credentials)
./scripts/build_with_mqtt.sh Cardputer-dev
```

**Build flags** ([platformio.ini](platformio.ini)):
- `-DBYPASS_SD_CHECK`: Skip SD card requirement check
- `-DM5STICKS3_ENV`: Enable LittleFS and button-only input
- `-DENABLE_COREDUMP_LOGGING`: Add MQTT-based crash reporting (requires MQTT credentials via `build_with_mqtt.sh`)

## Key Development Patterns

### Input Handling (Dual-Mode)

Keyboard (M5Cardputer) and button (M5StickS3) are mutually exclusive:

```cpp
#ifndef BUTTON_ONLY_INPUT
  M5Cardputer.update();
  auto keysState = M5Cardputer.Keyboard.keysState();
  // keysState.word contains typed characters
#else
  inputManager::update();
  if (inputManager::isButtonAPressed()) { ... }
#endif
```

### Font Loading (Critical Pattern)

**Always unload before loading a new font**:

```cpp
if (FSYS.exists("/M5Gotchi/fonts/big.vlw")) {
    canvas_main.loadFont(FSYS, "/M5Gotchi/fonts/big.vlw");
    canvas_main.setTextSize(0.35);
    canvas_main.drawString(text, x, y);
    canvas_main.unloadFont();  // MUST DO THIS
    canvas_main.setFont(&fonts::Font0);  // Reset to default
}
```

### Canvas Drawing (Threadsafe Pattern)

**All canvas operations must be protected**:

```cpp
if (displayMutex) xSemaphoreTake(displayMutex, portMAX_DELAY);
canvas_main.fillScreen(bg_color_rgb565);
canvas_main.setTextColor(tx_color_rgb565);
canvas_main.drawString("text", x, y);
xSemaphoreGive(displayMutex);
pushAll();  // Flush to hardware
```

### Menu State Machine

UI is menu-driven with global `menuID`:

```cpp
void updateUi(bool show_toolbars, bool triggerPwnagothi, bool overrideDelay) {
  switch (menuID) {
    case 0: // main menu
    case 1: // submenu
    case 2: // app launcher
    ...
  }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Devsur11/M5Gotchi](https://github.com/Devsur11/M5Gotchi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
