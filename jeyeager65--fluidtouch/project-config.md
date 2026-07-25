---
trigger: always_on
description: FluidTouch is an ESP32-S3 embedded touchscreen CNC controller for FluidNC machines, running on Elecrow CrowPanel 7" displays (800x480). The project uses PlatformIO, LVGL 9.4 for UI, and LovyanGFX for hardware-accelerated display rendering.
---

# FluidTouch - AI Coding Agent Instructions

## Project Overview
FluidTouch is an ESP32-S3 embedded touchscreen CNC controller for FluidNC machines, running on Elecrow CrowPanel 7" displays (800x480). The project uses PlatformIO, LVGL 9.4 for UI, and LovyanGFX for hardware-accelerated display rendering.

**Supported Hardware**: Elecrow CrowPanel ESP32 7" HMI Display (Basic and Advance variants)

### Basic Version
- Product: https://www.awin1.com/cread.php?awinmid=82721&awinaffid=2663106&ued=https%3A%2F%2Fwww.elecrow.com%2Fesp32-display-7-inch-hmi-display-rgb-tft-lcd-touch-screen-support-lvgl.html
- Display: 800x480 TN RGB TFT LCD with GT911 capacitive touch
- MCU: ESP32-S3-WROOM-1-N4R8 (4MB Flash + 8MB Octal PSRAM)
- Backlight: PWM on GPIO2
- Touch I2C: SDA=19, SCL=20, RST=38

### Advance Version  
- Product: https://www.awin1.com/cread.php?awinmid=82721&awinaffid=2663106&ued=https%3A%2F%2Fwww.elecrow.com%2Fcrowpanel-advance-7-0-hmi-esp32-ai-display-800x480-artificial-intelligent-ips-touch-screen-support-meshtastic-and-arduino-lvgl-micropython.html
- Display: 800x480 IPS RGB LCD with GT911 capacitive touch
- MCU: ESP32-S3-WROOM-1-N16R8 (16MB Flash + 8MB Octal PSRAM)
- Backlight: I2C controlled via STC8H1K28 microcontroller (address 0x30)
- Touch I2C: SDA=15, SCL=16 (RST handled by STC8H1K28 via I2C)
- **Key Differences**: Completely different RGB pin mapping, different sync pins, I2C backlight control

**Build Environments**: 
- `platformio run -e elecrow-crowpanel-7-basic` (4MB flash, PWM backlight)
- `platformio run -e elecrow-crowpanel-7-advance` (16MB flash, I2C backlight)

**Dual Hardware Support**: COMPLETE
- Conditional compilation via `#ifdef HARDWARE_ADVANCE` handles hardware-specific code
- Separate firmware builds for each hardware variant
- GitHub Actions builds both variants automatically
- Web installer allows users to select their hardware variant (manifest_basic.json / manifest_advance.json)
- Touch, display, and backlight fully working on both hardware variants

## Architecture & Design Patterns

### Hardware Platform
- **Flash**: 4MB (Basic) or 16MB (Advance), DIO/QIO mode
- **PSRAM**: 8MB Octal PSRAM (`dio_opi` or `qio_opi` memory type)
- **Display**: 800x480 RGB parallel interface (16-bit RGB565)
- **Touch**: GT911 I2C controller (address 0x5D)
- **Critical**: ALL large buffers MUST use `heap_caps_malloc(size, MALLOC_CAP_SPIRAM)` to allocate in PSRAM, never regular heap

### Dual Hardware Support Pattern
The project uses conditional compilation (`#ifdef HARDWARE_ADVANCE`) to support both hardware variants:

1. **Display Driver** (`src/core/display_driver.cpp`):
   - Basic: GPIO pins 15,7,6,5,4 (B), 9,46,3,8,16,1 (G), 14,21,47,48,45 (R), sync pins 41,40,39,0
   - Advance: GPIO pins 21,47,48,45,38 (B), 9,10,11,12,13,14 (G), 7,17,18,3,46 (R), sync pins 42,41,40,39
   - Timing: Basic 10MHz / Advance 14MHz (reduced from 18MHz for stability)
   
2. **Backlight Control**:
   - Basic: Direct PWM on GPIO2 via `ledcWrite()`
   - Advance: I2C commands to STC8H1K28 (wake 0x19, config 0x10/0x18, brightness 0x00-0xF5)

3. **Touch Controller**:
   - Both use GT911 at 0x5D, but different I2C pins
   - Basic: SDA=19, SCL=20, RST=38
   - Advance: SDA=15, SCL=16, RST=-1 (handled by STC8H1K28 via I2C)
   - **Touch panel configured in LGFX class** (display_driver.cpp) with `lgfx::Touch_GT911 _touch_instance`
   - LovyanGFX provides GT911 initialization and touch reading via `lcd->getTouch()`
   - TouchDriver delegates to LovyanGFX using `lcd_instance->getTouch(&x, &y)` in callback

### Module Organization Pattern
The codebase follows a **strict modular pattern** with clear separation:

1. **Driver Modules** (`core/` subdirectory):
   - `DisplayDriver` - LovyanGFX RGB parallel display with LVGL integration and GT911 touch panel configuration (`core/display_driver.h/cpp`)
   - `TouchDriver` - LVGL input device that delegates touch reading to LovyanGFX (`core/touch_driver.h/cpp`)
   - `PowerManager` - Power management for battery-powered operation with three power states (`core/power_manager.h/cpp`)
     - **States**: FULL_BRIGHTNESS → DIMMED → SCREEN_OFF → DEEP_SLEEP (optional)
     - **Brightness Storage**: NVS stores percentages (0-100), DisplayDriver converts to hardware values (0-255)
     - **NVS Keys** (shortened to fit 15-char limit): `pm_enabled`, `pm_dim_to`, `pm_sleep_to`, `pm_deepsleep`, `pm_norm_bri`, `pm_dim_bri`
     - **State-aware**: Only applies power saving in IDLE and DISCONNECTED states - all other states (RUN, ALARM, HOLD, JOG) stay at full brightness
     - **User activity**: Touch events call `onUserActivity()` to reset timer and restore full brightness
     - **Deep sleep**: Enters ESP32 deep sleep after timeout (0 = disabled), only reset button wakes
     - **Display control**: Uses `DisplayDriver::setBacklight(percentage)` for dimming, `DisplayDriver::powerDown()` for sleep
     - **Brightness initialization**: Applied immediately on init after loading settings from NVS

2. **Network Modules** (`network/` subdirectory):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeyeager65/FluidTouch](https://github.com/jeyeager65/FluidTouch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
