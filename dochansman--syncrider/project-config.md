---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

**SyncRider** — a smarter Fahrradcomputer (bike computer): the phone is the "brain" (GPS, routing, storage), an ESP32-S3-AMOLED board is a dumb display/input device, connected via BLE. Two sub-projects live in this repo:

- `app/` – Android app (Kotlin, Jetpack Compose).
- `firmware/` – ESP32-S3 firmware (ESP-IDF v5.4.4, C, NimBLE + LVGL). BLE GATT peripheral (`RideData`, `RouteData`, `NavStatus`, `ControlEvent`, `DeviceStatus`, `PhoneStatus`, `SetTime`, `MediaStatus`, `DeviceCommand`, `DisplayConfig`, `RideSummary`) and AMOLED display bring-up (5-screen carousel: Datenübersicht, Navigation, Musik-Steuerung, Fahrtsteuerung, Statistik) both working on real hardware (see below).

The spec docs in `docs/` are the source of truth for architecture and protocol decisions and should be consulted (and kept in sync) when working on BLE or navigation code:

- `docs/wiki/` — the reader-facing documentation (hardware, build, app, board, internals, troubleshooting). Superseded the original spec/mockup/scaffolding notes, which now live in the untracked `docs/archiv/`.
- `docs/fahrradcomputer-ble-protokoll.md` — exact BLE GATT service/characteristic byte layout
- `docs/gestaltung.md` — the design system shared by app and board (seven principles, colour tokens,
  type scale, icon language), the **metric-ID catalog** (stable cross-device numbering used by
  `metrics/Metric.kt`, `DisplayConfig` and the planned `BoardInfo.metric_mask` — append-only), the
  app's information architecture, and the data-model/charting decisions. Code comments cite it by
  section number, so renumbering its sections orphans those references. Mockups live in
  `docs/mockups/` (board screens, round-display safe area + slot templates, app screens).
- `docs/features.md` — the feature wishlist with its verdicts, `docs/bugs.md` the record of every
  fault this project actually had and what it cost to find. Both are load-bearing: code comments
  cite them constantly.

Project docs and commit messages are in German; code identifiers and comments are in English.

## Build, lint, test

Run from the repo root (Gradle wrapper). On Windows use `gradlew.bat`; `gradlew` (bash) also works in this environment.

```
./gradlew.bat assembleDebug          # build the app
./gradlew.bat test                   # run all JVM unit tests (app/src/test)
./gradlew.bat test --tests "com.hannes.bikecomputer.navigation.RouteSimplifierTest"   # single test class
./gradlew.bat test --tests "*.RouteSimplifierTest.reduces a nearly straight line*"    # single test method
```

There is no configured lint/ktlint/detekt task beyond the default Android Gradle Plugin `lint` task (`./gradlew.bat lint`). Toolchain: JDK 21 (see `gradle/gradle-daemon-jvm.properties`), Kotlin 2.3.21, AGP 9.3.0, compileSdk/targetSdk 36, minSdk 29.

Tests are plain JUnit4 with `unitTests.isReturnDefaultValues = true` (otherwise every `android.util.Log` call in an otherwise pure class throws "not mocked" — removing the logging instead would cost the `RideRecorder` lines that are the first source in any history investigation), no instrumented/androidTest suite exists — everything testable (BLE payload encoding, GPX parsing, route simplification) is written as pure JVM logic to keep it unit-testable without an emulator.

### Firmware (ESP-IDF)

Requires **ESP-IDF v5.4.x** on the machine (v5.4+ is mandatory: both Waveshare's official display BSP and the `esp_lcd_sh8601` panel driver require it). Source its environment script first (`export.ps1` on Windows, `export.sh` elsewhere) so `idf.py` is on PATH — Git Bash/MSYS is **not** supported by ESP-IDF's tooling, use PowerShell/cmd (or WSL) instead.

```
idf.py set-target esp32s3      # once per fresh build dir
idf.py build
idf.py -p <PORT> flash monitor # <PORT> e.g. COM3 on Windows; board uses USB-Serial/JTAG, no external adapter needed
```

Dependencies (`waveshare/esp32_s3_touch_amoled_1_75` BSP, `lvgl/lvgl`) are declared in `firmware/main/idf_component.yml` and auto-fetched into `firmware/managed_components/` by the ESP-IDF Component Manager on first configure — needs network access; never hand-edit files under `managed_components/`, they get overwritten.

## Architecture

The app is intentionally split so that BLE wire format, GPS/ride math, and route processing are pure, testable Kotlin with no Android UI dependency, while `ui/` wires them together with Compose.

- `ble/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DocHansman/SyncRider](https://github.com/DocHansman/SyncRider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
