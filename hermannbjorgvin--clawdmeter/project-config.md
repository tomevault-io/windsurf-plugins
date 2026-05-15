---
trigger: always_on
description: ESP32-S3 firmware for a desk-side Claude Code usage monitor on a **Waveshare ESP32-S3-Touch-AMOLED-2.16** board (480×480 square AMOLED). Connects to a host daemon over BLE; daemon polls Anthropic API for usage data.
---

# Project context

ESP32-S3 firmware for a desk-side Claude Code usage monitor on a **Waveshare ESP32-S3-Touch-AMOLED-2.16** board (480×480 square AMOLED). Connects to a host daemon over BLE; daemon polls Anthropic API for usage data.

This file is for future Claude Code sessions to bootstrap quickly. Read this first.

## Hardware (critical pins)

- Display: **CO5300** AMOLED via QSPI (CS=12, SCLK=38, SDIO0..3=4..7, RST=2)
- Touch: **CST9220** via I2C (SDA=15, SCL=14, INT=11, addr=0x5A)
- PMU: **AXP2101** on same I2C bus (addr=0x34) — battery, USB VBUS, PWR button IRQ
- IMU: **QMI8658** on same I2C bus (addr=0x6B) — accelerometer for auto-rotation
- Buttons: GPIO 0 (left → Space/voice-mode), GPIO 18 (right → Shift+Tab/mode-toggle), AXP PKEY (middle → cycle screens; on splash → cycle animations)

## Architecture

```
main.cpp        — setup(), loop(), button polling (left→Space, right→Shift+Tab, mid→cycle), rotation flash
display_cfg.h   — pin defines, extern object decls
ui.{h,cpp}      — 3-screen UI (splash, usage, bluetooth); splash is touch-toggled, usage↔bluetooth via mid button
splash.{h,cpp}  — 20×20 pixel-art animation engine, 24× upscale to 480×480
imu.{h,cpp}     — accelerometer-driven rotation tracker (returns 0..3)
power.{h,cpp}   — AXP2101 wrapper (battery %, charging, VBUS, PWR button)
touch.{h,cpp}   — minimal tap detector → ui_toggle_splash() (Usage/Splash) or ble_clear_bonds() (BT reset zone)
ble.{h,cpp}     — NimBLE peripheral: custom data service + HID keyboard
data.h          — UsageData struct
icons.h         — icon arrays. Battery (5×) are RGB565A8 with alpha; rest are raw RGB565.
logo.h          — 80×80 RGB565 logo
font_*.c        — pre-compiled LVGL 9 bitmap fonts (Tiempos 56, Styrene 48/28/24/20, Mono 32)
splash_animations.h — generated, do not hand-edit
```

## Build / flash

```bash
pio run -d firmware                                       # build
pio run -d firmware -t upload --upload-port /dev/ttyACM0  # flash (binary path uses USB JTAG)
```

`/home/hermann/.platformio/penv/bin/pio` if `pio` isn't on PATH.

Device shows up as `/dev/ttyACM0` (Espressif USB JTAG/serial debug unit). No boot-mode gymnastics needed — direct flash works.

## QA your own UI changes — don't ask the user

The firmware ships a `screenshot` serial command that dumps the LVGL framebuffer over `/dev/ttyACM0`. `./screenshot.sh out.png /dev/ttyACM0` captures a 480×480 PNG. **Use this on every UI iteration** — Read the PNG with the Read tool, verify the change visually, iterate.

The boot screen is `SCREEN_SPLASH` and only advances on a physical button press, so a fresh flash will sit on the splash. To screenshot the screen you're actually editing without asking the user to press a button, **temporarily change the default boot screen** in `main.cpp` (search for `ui_show_screen(SCREEN_SPLASH);`) to `SCREEN_USAGE` / `SCREEN_CONTROLLER` / `SCREEN_BLUETOOTH`, do your iteration, then revert before committing.

## Critical gotchas

1. **CO5300 cannot rotate.** Its MADCTL only supports axis flips, not column/row exchange. Rotation is done by **CPU pixel remapping in `my_flush_cb`** in main.cpp. We use **PARTIAL render mode with strip rotation** (small 480×40 strips, fast). On rotation change → AMOLED brightness flash → force redraw.
2. **OPI PSRAM** required: `board_build.arduino.memory_type = qio_opi` in platformio.ini. Without this, `MALLOC_CAP_SPIRAM` returns NULL and the screen is black.
3. **pioarduino platform required.** GFX Library for Arduino needs Arduino Core 3.x (`esp32-hal-periman.h`), not the 2.x that standard `espressif32` ships. We pin `pioarduino/platform-espressif32` 55.03.38-1.
4. **LVGL 9 font patching.** `lv_font_conv` outputs LVGL 8 format. Must remove `#if LVGL_VERSION_MAJOR >= 8` guards, drop `.cache` field, add `.release_glyph`, `.kerning`, `.static_bitmap`, `.fallback`, `.user_data`. Without patching, fonts render invisible.
5. **Touch reading must be centralized.** CST9220's `getPoint()` does a full I2C transaction. Calling it from multiple places consumed each other's data and broke input. `touch_read()` is called once per loop in main.cpp; both LVGL `my_touch_cb` and `touch.cpp` read from shared `touch_pressed/touch_x/touch_y` state.
6. **CO5300 needs even-aligned flush regions.** `rounder_cb` enforces this.
7. **Touch `setSwapXY(true)` and `setMirrorXY(true, false)`** are the empirically-correct values for default rotation 0. IMU rotation logic doesn't change touch mapping (it does CPU-side rotation of the rendered pixels, so LVGL still thinks the display is portrait at 0°).
8. **LVGL RGB565A8 is planar.** `w*h` RGB565 pixels followed by `w*h` alpha bytes; `data_size = w*h*3`, `stride = w*2`. Use `init_icon_dsc_rgb565a8()` for icons that overlap non-uniform backgrounds (e.g. battery over splash). Lucide source PNGs are black-on-transparent — converter must tint to white or icons render invisible. See `tools/png_to_lvgl.js`.

## Icons


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HermannBjorgvin/Clawdmeter](https://github.com/HermannBjorgvin/Clawdmeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
