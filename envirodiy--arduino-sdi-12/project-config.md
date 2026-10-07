---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An Arduino library (`SDI-12`, v2.3.3) that bit-bangs the SDI-12 sensor protocol (1200 baud, 7E1, inverted logic) on a single GPIO with no extra hardware. It descends from SoftwareSerial, with NeoSWSerial's timer-based receive so the processor is not stalled during Rx. Upstream is `EnviroDIY/Arduino-SDI-12`; this checkout's `origin` is the `similie` fork, which exists because ESP32 recorders reading older sensors regressed between v2.1.4 and v2.2.0+ (see the v2.3.3 entry in `ChangeLog.md` for the analysis). Fork releases are tagged `vX.Y.Z` and consumed by HyphenOS via `lib_deps = https://github.com/similie/Arduino-SDI-12.git#v2.3.3`. Version must agree in four places: `VERSION`, `library.properties`, `library.json`, and `PROJECT_NUMBER` in `docs/Doxyfile`.

## Commands

There is no root `platformio.ini`, no unit tests, and no host-side build — this is a library, and "the build" means compiling the example sketches against the boards CI targets. `pio` (PlatformIO Core) is installed locally; `arduino-cli`, `doxygen`, `clang-format`, and `cspell` are not.

```bash
# Compile one example for one board (this is exactly what CI does)
PLATFORMIO_CI_SRC=examples/d_simple_logger pio ci --board=uno --lib="."

# Several boards at once
PLATFORMIO_CI_SRC=examples/a_wild_card pio ci --board=mayfly --board=uno --board=featheresp32 --lib="."

# Override the pins an example uses (all examples guard these with #ifndef)
PLATFORMIO_BUILD_FLAGS="-D SDI12_DATA_PIN=5 -D SDI12_POWER_PIN=-1" PLATFORMIO_CI_SRC=examples/f_basic_data_request pio ci --board=uno --lib="."

# Example J is special: needs SDI12_EXTERNAL_PCINT and the EnableInterrupt library (AVR only)
PLATFORMIO_BUILD_FLAGS="-D SDI12_EXTERNAL_PCINT" PLATFORMIO_CI_SRC=examples/j_external_pcint_library \
  pio ci --board=uno --board=mayfly --lib="." --project-option="lib_deps=greygnome/EnableInterrupt@^1.1.0"

# Docs (needs doxygen 1.14; output lands in ../../Arduino-SDI-12_Doxygen, a sibling of the repo)
cd docs && doxygen Doxyfile

# Format (Google-based style, 88 cols, 2-space, aligned consecutive assignments/declarations)
clang-format -i src/*.cpp src/*.h examples/*/*.ino
```

CI boards (`.github/workflows/build_examples.yaml`): `mayfly envirodiy_stonefly_m4 uno megaatmega2560 leonardo zeroUSB arduino_nano_esp32 feather328p feather32u4 adafruit_feather_m0 adafruit_feather_m4 huzzah featheresp32 esp32-c3-devkitm-1 esp32-s3-devkitm-1`. Only `atmelavr` and `espressif32` platforms are installed locally; others download on first use. A change to the board-selection preprocessor in `SDI12_boards.h` should be compiled for at least one AVR, one SAMD21, one SAMD51, and one ESP32 target because each takes a different branch.

The per-example `platformio.ini` files are for users who installed the library from the registry (`src_dir` points into `.piolibdeps/`) — they are not usable for in-tree builds; use `pio ci` as above.

CI also runs `arduino-lint` in strict library-compliance mode (`verify_library_structure.yaml`) and a Doxygen build with `WARN_AS_ERROR = FAIL_ON_WARNINGS_PRINT` and `WARN_IF_UNDOCUMENTED = YES` — **an undocumented public member fails CI.** Commits containing `ci skip` bypass all three workflows. A bot nags PRs that don't touch `ChangeLog.md`; add entries under `## [Unreleased]`. Pushing a change to `VERSION` on `master` triggers the release workflow.

## Architecture

Two classes, four files in `src/`:

- **`SDI12` (`SDI12.h/.cpp`)** — a `Stream` subclass. Owns the protocol: state machine, Tx bit-banging, the Rx ISR, the receive ring buffer, CRC helpers, and SDI-12-specific `parseInt`/`parseFloat` overrides.
- **`SDI12Timer` (`SDI12_boards.h/.cpp`)** — the only board-specific layer. Picks a hardware timer/prescaler per MCU, saves and restores the timer registers, and converts tick deltas to bit counts.

### Board selection is entirely preprocessor

`SDI12_boards.h` has two `#if` ladders. The first keys on MCU/board macros and defines `TIMER_INT_SIZE` (8/16/32), `READTIME`, `TICKS_PER_SECOND`, and the timer to use; the second keys on `(TICKS_PER_SECOND, TIMER_INT_SIZE)` and derives `TICKS_PER_BIT`, `BITS_PER_TICK_Q10` (8-bit timers only), and `RX_WINDOW_FUDGE`. `SDI12_boards.cpp` has a matching ladder providing `configSDI12TimerPrescale()` / `resetSDI12TimerPrescale()` / `SDI12TimerRead()`. Unmatched boards hit `#error`. Adding a board means touching all three ladders.

| Target | Timer | Notes |
|---|---|---|
| ATmega 168/328/644/1284/1280/2560 | Timer2, 8-bit | conflicts with `tone()`; prescaler depends on `F_CPU` (16/12/8 MHz) |
| ATtiny 25/45/85 | Timer1, 8-bit | |
| ATmega32U4 (Leonardo, Feather 32u4) | Timer4 low 8 bits | |
| SAMD21 | GCLK4 → TC3, 16-bit | TC3 shares GCLK with TCC2 |
| SAMD51/SAME51 | GCLK6 → TC2, 16-bit | reading COUNT requires a READSYNC command first |
| ESP32/ESP8266, Particle, Giga, anything ≥ 48 MHz | `micros()`, 32-bit | no prescaler changes; `SDI12_YIELD_MS` defaults to 8 here |

Prescalers are changed in `begin()` and restored in `end()`, so `end()` matters on AVR.

### Receive path (see `docs/CreatingACharacter.md`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnviroDIY/Arduino-SDI-12](https://github.com/EnviroDIY/Arduino-SDI-12) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
