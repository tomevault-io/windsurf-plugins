---
trigger: always_on
description: Manages WiFi in station mode. Reads credentials from NVS or from
---

# Copilot Instructions

## Code Style

- Do not use non-ASCII characters in code, comments, or string literals. All source files must be ASCII-only.

## Project Overview

Draftling is a distraction-free Markdown text editor for ESP32-S3-based
development boards with reflective LCD displays. It is built with the
ESP-IDF framework (v5.3+) and uses LVGL v9 for the graphical interface.

The user connects a Bluetooth keyboard and edits Markdown files stored on
a MicroSD card. The reflective LCD needs no backlight and works well in
daylight. On request the device connects to WiFi and synchronizes files
with a remote Git repository via the GitHub REST API.

### Supported Hardware

| Board | Display |
|-------|---------|
| Waveshare ESP32-S3-RLCD-4.2 | 4.2-inch reflective LCD, 400x300 |
| Seeed Studio reTerminal E1001 | 7.5-inch e-paper (UC8179), 800x480 |
| Waveshare E-Paper Driver HAT (on any BLE-capable ESP32 host, default ESP32-S3-DevKitC-1 wiring) | UC8179 e-paper, panel preset (default Waveshare 7.5" V2 / GDEW075T7, 800x480) |
| M5Stack PaperS3 | 4.7-inch e-paper (ED047TC1), 540x960 |

## Repository Layout

```
CMakeLists.txt              Top-level CMake project file
partitions.csv              Custom partition table (16 MB flash)
sdkconfig.defaults          Common Kconfig defaults for all targets
sdkconfig.defaults.esp32s3  ESP32-S3-specific defaults (PSRAM, BLE, WiFi)
main/                       Application entry point and hardware config
  main.cpp                  app_main(): initializes all subsystems
  app_config.h              Pin definitions and display macros per board
  Kconfig.projbuild         Menuconfig: hardware model, display size, rotation
  idf_component.yml         IDF component manifest (depends on lvgl ^9.2)
  CMakeLists.txt            Registers main as an IDF component
components/                 Reusable IDF components
  battery/                  Battery voltage monitor (ADC)
  ble_keyboard/             BLE HID keyboard host (Bluedroid)
  display/                  RLCD SPI display driver and LVGL port
  editor/                   Gap-buffer text editor, Markdown parser, LVGL UI
  fonts/                    Custom LVGL bitmap fonts (Greybeard family)
  git_sync/                 GitHub REST API file synchronization
  kb_layout/                Keyboard layout translation (US/UA/DE/FR)
  sd_card/                  SD card (SDMMC 1-bit) file operations
  standby/                  Deep-sleep / standby inactivity timer
  wifi_manager/             WiFi STA connection manager
```

## Component Details

### main/

The application entry point. `app_main()` in `main.cpp` initializes every
subsystem in order: NVS flash, display hardware, LVGL, battery monitor,
editor UI, SD card, BLE keyboard, WiFi manager, Git sync, and standby
timer. It also registers an auto-save callback that persists the current
document before entering deep sleep.

`app_config.h` maps Kconfig hardware model selections to concrete GPIO pin
numbers and display dimensions. Each supported board has its own `#if`
block defining SPI pins (MOSI, SCK, DC, CS, RST, TE/BUSY), SD card pins
(CLK, CMD, D0 for SDMMC, or MOSI/MISO/SCK/CS for SPI), I2C pins, the
battery ADC pin, and the deep-sleep wakeup GPIO. The Waveshare E-Paper
Driver HAT block resolves all of its pin macros from `CONFIG_DRAFTLING_HAT_*`
Kconfig values so users can adapt the HAT to a different ESP32-S3 host
board without editing source. The M5Stack PaperS3 block omits the panel
data-bus and control-line pins because the `m5stack/M5GFX` library
configures them internally based on the board id.

### components/battery/

Reads battery voltage through a resistive divider on a configurable
ADC pin using the ESP32 ADC. Applies exponential moving average
smoothing over 8 samples. Maps voltage to a percentage: >=4.10V is
100%, >=3.95V is 75%, >=3.80V is 50%, >=3.60V is 25%, below 3.60V
is 0%. The M5Stack PaperS3 reads its cell through ADC1 on GPIO3 with
a 1:2 divider, matching the M5Unified Power_Class configuration for
that board. The bare Waveshare EPD HAT has no on-board battery and
passes `BATT_ADC_PIN = -1`, which makes `battery_init()` a no-op and
causes `battery_read_percent()` to return -1; the editor UI hides the
battery icon in that case.

Public API: `battery_init()`, `battery_read_mv()`, `battery_read_percent()`.

### components/ble_keyboard/

BLE HID keyboard host built on ESP-IDF Bluedroid. Handles device scanning,
pairing with passkey authentication, connection/disconnection callbacks,
and keyboard event dispatching. Each key event carries the HID keycode,
ASCII character, modifier flags, and pressed/released state.

Public API: `ble_keyboard_init()`, `ble_keyboard_start_scan()`,
`ble_keyboard_is_connected()`, `ble_keyboard_set_callback()`, and
several other callback registration functions.

### components/display/

Per-board display backends behind a single C API:

- **display_rlcd.cpp** -- Waveshare ESP32-S3-RLCD-4.2 reflective LCD
  over SPI.
- **display_uc8179.cpp** -- UC8179 e-paper over SPI. Shared by the
  Seeed reTerminal E1001 and the Waveshare E-Paper Driver HAT;
  resolution and pinout (including BUSY) are passed in at init.
  When `DRAFTLING_EPD_FAST_PARTIAL` is set (auto-enabled by the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clackups/draftling](https://github.com/clackups/draftling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
