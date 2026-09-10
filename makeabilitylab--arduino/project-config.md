---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose & context

This repo is a teaching collection: ~260 Arduino sketches plus supporting Python/PowerShell/Processing scripts that accompany the **Makeability Lab Physical Computing textbook** (https://makeabilitylab.github.io/physcomp/, source in `d:\git\physcomp`). The code is instructional material — student readability and pedagogical clarity matter as much as correctness.

We are preparing the textbook's **v2.0** release (summer 2026) and modernizing the code here: improving code quality, comments, understandability, and correctness, and linking examples back to the relevant textbook lessons. When improving an example, optimize for a student reading it, not just for working code. The Signals repo (`d:\git\signals`) is the closest analog undergoing the same modernization and is a useful reference for conventions.

The detailed, beginner-oriented `Python/SerialReader/README.md` (prerequisites → usage → troubleshooting → "Learn more" with lesson links) is a good model for the modernized documentation voice.

## Build / run / test

There is **no build system, package manifest, or automated test suite.** Each program is run with its native toolchain:

- **Arduino sketches** (`.ino`): compiled and uploaded via the Arduino IDE (canonical, student-facing). For our internal dev loop, `arduino-cli` (v1.5.x) is installed at `C:\Program Files\Arduino CLI\arduino-cli.exe` — it may not be on every shell's PATH, so call it by full path if `arduino-cli` isn't found. Compile-check a sketch without uploading:
  ```
  arduino-cli compile --fqbn <board-fqbn> <SketchFolder>
  ```
  Each sketch lives in its own folder whose name matches the `.ino` file (Arduino requirement). There is no committed board config; choose the FQBN per sketch by hardware folder:
  | Folder | Board | FQBN |
  |---|---|---|
  | `Basics/`, `OLED/`, `Filters/`, `Sensors/`, `Servo/`, etc. | Arduino Uno (AVR) | `arduino:avr:uno` |
  | `ESP32/` | ESP32 dev board | `esp32:esp32:esp32` |
  | `UnoR4WiFi/` | Uno R4 WiFi | `arduino:renesas_uno:unor4wifi` |
  | `HumanInterfaceDevice/`, `GameController/` | Leonardo/Micro (USB HID) | `arduino:avr:leonardo` |
  | `CPX/` | Circuit Playground Express | `adafruit:samd:adafruit_circuitplayground_m0` |
  | `nRF52840/` | nRF52840 board (verify exact board) | `adafruit:nrf52:feather52840` |

  Installed cores: `arduino:avr`, `arduino:renesas_uno`, `esp32:esp32`, `adafruit:samd`, `adafruit:nrf52`. The AVR/HID split is inferred from each sketch's APIs (`Mouse.h`/`Keyboard.h` ⇒ Leonardo-class); the `nRF52840/` board defaults to Feather Express but confirm the exact board from the sketch header. Legacy `RedBearDuo/` and Processing have no installable core — skip compile-checking them.
- **Python helpers** (`Python/`): require Python 3 and `pip install pyserial`. Run directly, e.g. `python Python/SerialReader/serial_reader.py --list`. Note the common pitfall: install `pyserial`, **not** `serial`.
- **Processing sketches** (`.pde`, `Processing/`): legacy/historical only — the project has shifted to p5.js. Don't invest in modernizing these unless asked.

**Legacy / CI-excluded sketches:** some sketches are intentionally kept but not expected to compile (e.g., they're the source behind textbook demo videos, written for an older core). These are listed in [`docs/legacy-sketches.md`](docs/legacy-sketches.md) and carry a `LEGACY EXAMPLE` banner at the top of the `.ino`. The compile sweep and CI must read that list and skip them. Before deciding an ESP32 sketch is "broken vs. migrate," **check its corresponding physcomp lesson first** — the lesson may have already moved to a new approach (as the `esp32/tone` lesson did), making the sketch legacy rather than a migration target.

## Repository structure

Top-level folders group examples by topic or hardware platform — flat and self-contained; there is no shared application or cross-sketch dependency graph. Each sketch is independent.

Topic folders include `Basics/` (analogRead/analogWrite/digitalRead/digitalWrite/serial/tone), `OLED/` (SSD1306 graphics & games), `AddressableLEDs/` (NeoPixel, APA102), `Filters/` (smoothing filters), `Sensors/`, `Servo/`, `Bluetooth/`, `Serial/`, `GameController/`, `HumanInterfaceDevice/` (USB HID), `PlayingMusic/`.

Hardware-specific folders: `ESP32/`, `UnoR4WiFi/`, `CPX/` (Circuit Playground Express), `nRF52840/`, and `RedBearDuo/` (legacy).

`MakeabilityLab_Arduino_Library/` in this repo is now **just a README pointer** — the shared library lives in its own repo, **`MakeabilityLab`** (https://github.com/makeabilitylab/makelab-arduino-lib), cloned locally at `d:\git\makelab-arduino-lib`. It is a proper installable Arduino library (`library.properties` declares `depends=Adafruit SSD1306, Adafruit GFX Library`, so installing it pulls those in).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makeabilitylab/arduino](https://github.com/makeabilitylab/arduino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
