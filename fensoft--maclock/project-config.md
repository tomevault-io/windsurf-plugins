---
trigger: always_on
description: Operational notes for future Codex sessions in this repository.
---

# AGENTS.md

Operational notes for future Codex sessions in this repository.

## Project Snapshot

Maclock is ESP32-S3 firmware for replacing a Maclock's original screen with a
320x240 IPS display. It has two mutually exclusive runtime modes:

- A normal LVGL clock interface with startup animation, MP3 effects, RTC and
  weather data, touch calibration, and persistent brightness/boot settings.
- A Mini vMac emulator that can launch from Boot Options or directly at
  power-on when the saved default selects the emulator.

Main stack:

- PlatformIO with the Arduino framework and the `lolin_s3` environment.
- TFT_eSPI for the ILI9341 display.
- LVGL 9 for the normal clock UI.
- LittleFS for images, sounds, the Mini vMac ROM, and disk images.
- FreeRTOS tasks for input polling, MP3 decoding, and emulator rendering.
- A generated Mini vMac 36.04 source tree plus tracked ESP32 glue.

## Runtime Shape

`src/main.cpp` is a minimal Arduino adapter. Its static `MaclockApp` delegates
`setup()` to `begin()` and `loop()` to `tick()`; `MaclockApp` owns the boot
decision and service graph.

- It initializes NVS preferences, LittleFS, the TFT, and touch first.
- Holding the clock button at boot requests the boot-options UI.
- Otherwise, the saved `floppy_emulator` preference calls `minivmac()`
  immediately, independently of the physical floppy switch.
- Mini vMac runs synchronously. Its Clock+Alarm safe-exit chord returns from
  the call, after which the LVGL clock stack is initialized and Boot Options
  opens.
- The normal path initializes the ES8311 codec, LVGL, LittleFS's LVGL driver,
  UI assets, RTC, encoder, input/audio tasks, and weather sensor.
- `MaclockApp::tick()` then runs the UI state machine, can call `minivmac()`
  again from Boot Options, and remains the only owner of LVGL calls.
- In the normal clock state, holding Clock+Alarm for two seconds opens Boot
  Options; a Clock-only press opens date/time editing on release.

Do not try to run the clock UI and Mini vMac concurrently unless the user
explicitly asks for an architectural change. They share the display, touch,
LittleFS, and boot lifecycle.

## Important Files

- `platformio.ini`: board, libraries, pin assignments, TFT configuration,
  optimization flags, emulator include paths, and generated-source filters.
- `partitions.csv`: one 3 MiB application partition and a large LittleFS
  partition.
- `prepare.sh`: downloads and extracts Mini vMac 36.04, applies tracked
  patches, and downloads ignored ROM/disk images when absent.
- `src/main.cpp`: the under-50-line Arduino adapter.
- `include/maclock_app.h`, `src/maclock_app.cpp`: composition root, owned
  services, typed state, event sink, and private UI callback context.
- `src/ui/*.cpp`: focused Boot Options, clock-face, shared-shell, asset, and
  state-machine implementation units. They are included into
  `maclock_app.cpp` under `MACLOCK_COMBINED_SOURCE` and intentionally compile
  empty when PlatformIO discovers them separately.
- `include/*_service.h`, matching `src/*.cpp`: settings, I2C, RTC, weather,
  input, display, audio, Wi-Fi, alarm, and timer ownership.
- `include/control_panel.h` and `src/control_panel.cpp`: station-only local
  web controls and JSON routes. This is intentionally separate from the
  captive Wi-Fi setup portal in `wifi_mode.cpp`.
- `web/control-panel/`: responsive Vue source for the classic Macintosh web
  control panel. `scripts/build_control_panel.py` runs as a PlatformIO prebuild
  hook and regenerates the gzip-compressed `src/control_panel_page.h` whenever
  the web-source fingerprint changes. Never hand-edit the generated header.
- `src/init.cpp`: `DisplayService`, including TFT/LVGL, LittleFS LVGL driver,
  ES8311/I2S ownership, and the narrow Mini vMac hardware bridge.
- `src/datetime_ui.cpp`: state-owning date/time editor; it reports RTC changes
  and transitions through `AppEventSink`.
- `src/touch.cpp`, `include/touch.h`: FT6336 coordinate mapping and
  EEPROM-backed four-corner calibration.
- `src/FT6336.cpp`, `include/FT6336.h`: low-level I2C touch-controller driver.
- `include/TouchSensor.h`: discrete GPIO touch/button helper used for wake
  activity and the emulator mouse button.
- `src/es8311.cpp`, `include/es8311*.h`: audio-codec driver.
- `src/minivmac_ArduinoAPI.cpp`, `include/minivmac/ArduinoAPI.h`: Arduino,
  LittleFS, display, memory, time, and touch adapter for Mini vMac.
- `src/minivmac_OSGLUE.c`: tracked Mini vMac host glue for ROM/disk access,
  timing, input, and screen invalidation.
- `include/minivmac/*.h`: tracked generated/configured Mini vMac build
  settings, including the Macintosh Plus model and 304x224 monochrome screen.
- `patches/`: reproducible changes applied to the generated upstream tree.
- `data/`: LittleFS image contents—tracked UI/audio assets plus ignored local
  ROM and `disk*.dsk` files.
- `docs/ARCHITECTURE.md`: deeper system overview.

## Generated And Binary Inputs

`src/minivmac/` is generated by `prepare.sh` and ignored by Git. Do not treat it
as the canonical place for project-specific changes.

- Put reproducible upstream changes in `patches/` and update `prepare.sh` when
  necessary.
- Keep the tracked bridge/configuration files outside the generated subtree in
  sync with any Mini vMac change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fensoft/maclock](https://github.com/fensoft/maclock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
