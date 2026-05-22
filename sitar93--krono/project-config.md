---
trigger: always_on
description: This file is the **single technical reference** for anyone modifying the codebase (humans or agents). End-user documentation lives in **`README.md`**. Version history is in **`CHANGELOG.txt`**.
---

# AGENTS.md — KRONO firmware (contributors & coding agents)

This file is the **single technical reference** for anyone modifying the codebase (humans or agents). End-user documentation lives in **`README.md`**. Version history is in **`CHANGELOG.txt`**.

---

## Quick orientation

| Topic | Where |
|--------|--------|
| Usage, hardware, modes (user) | `README.md` |
| OMEGA tier (modes 21–30): behavior table (user) + firmware notes (this file) | `README.md` + **OMEGA / internal Gamma** section below |
| What changed per release | `CHANGELOG.txt` |
| License | `LICENSE.txt` |
| Build, style, architecture, debug | This file (`AGENTS.md`) |

---

## Project overview

KRONO is a multi-modal rhythm generator firmware for **STM32F411CE** (Black Pill / Krono PCB), built with **PlatformIO** and **libopencm3**. Target environment: `blackpill_f411ce`.

---

## Build / test commands

```bash
platformio run -e blackpill_f411ce
platformio run -e blackpill_f411ce --target upload
platformio run -e blackpill_f411ce --target clean
platformio device monitor
```

- Output: `.pio/build/blackpill_f411ce/` — after a successful build, post-scripts produce **`krono_code_<VERSION>.bin`**, **`.elf`**, **`.hex`** (version from the last non-empty line of **`CHANGELOG.txt`**, currently **v1.4.0**).
- **No automated tests** — verify on hardware after upload.

### Upload (DFU)

1. Hold **BOOT0** (often near USB).
2. Press and release **NRST** (reset).
3. Release **BOOT0**.
4. Device should enumerate as STM32 bootloader; run upload command.

**Windows:** If DFU fails, ensure `dfu-util` is available; Zadig/WinUSB may be needed for `STM32 BOOTLOADER`.

---

## Code style

- **Repository language:** All source code, comments, and project documentation files (`README.md`, `AGENTS.md`, `CHANGELOG.txt`, etc.) are written in **English**, even when discussion happens in other languages.
- **Language:** C (C11).
- **Indentation:** 4 spaces, no tabs.
- **Braces:** K&R (opening brace on same line).
- **Line length:** max 120 characters.
- **Names:** `snake_case` functions/variables; `UPPER_SNAKE_CASE` macros; `snake_case_t` typedefs.
- **Prefixes:** drivers `io_`, `tap_`, `rtc_`, `persistence_`, `ext_clock_`; modules `input_handler_`, `status_led_`, `mode_`, `clock_manager_`.
- **Includes:** stdlib → libopencm3 → local `"headers.h"`.
- **Comments:** Doxygen `/** */` for APIs; `//` inline; `// --- Section ---` for blocks.
- **ISR:** keep short; defer work to the main loop.
- **Lint/format:** not configured; match existing files.

---

## Repository layout (firmware)

```
src/
├── main.c                 # Orchestrator, callbacks, main loop, PA3 soft blink + pattern pump, millis(); save/load glue for mode fields
├── main_constants.h       # Timing bounds, shared defines
├── variables.h            # Tunable parameters
├── krono_aux_led_pattern.c/h  # Optional multi-pulse PA3 sequences (future tiers); cooperates with main soft-blink timer
├── input_handler.c/h      # Inputs, tap averaging, op-mode SM (+ Omega + Gamma), ext clock, tempo dispatch, short-MOD mode actions (12–30)
├── clock_manager.c/h      # F1 pulse, mode_context, mode dispatch
├── status_led.c/h
├── drivers/               # io, tap (incl. tap_abort_capture), ext_clock, persistence, rtc
├── modes/                 # mode_*.c, modes.c/h, mode_mod_dispatch.c, mode_rhythm_shared.c/h
└── util/
platformio.ini
```

---

## Runtime data flow (tempo)

1. `tap.c` — EXTI0 on PA0: intervals → `tap_detected()` / `tap_get_interval()`.
2. `input_handler.c` — tap quadruple policy in `input_tempo.c`; tempo callback on boundaries + external clock; op-mode SM can drain tap events.
3. `main.c` — `on_tap_tempo_change` → `clock_manager_arm_tap_quadruple_boundary()` on tap 4/8/…, else `clock_manager_set_internal_tempo()` (external / fallback).
4. `clock_manager.c` — schedules F1; passes `mode_context_t` to active `mode_*_update()`.

**Current tap policy (v1.4.0):** Boundaries **4, 8, 12, …** each fire **F1** once and set tempo from the **median** of three gaps (carry-in after a boundary still dropped). **Leading** boundaries (4, 12, …): emit that median. **Trailing** (8, 16, …): if **≤ `TAP_QUAD_BLEND_WINDOW_MS`** (3 s) after the previous leading press, tempo = **(30×leading median + 70×trailing median) / 100** (rounded); else trailing median only. Clicks between boundaries do not retune. **`TAP_PATTERN_IDLE_RESET_MS`** (5 s) resets the pattern. **`clock_manager_arm_tap_quadruple_boundary`** runs in **`clock_manager_update`**. External clock unchanged.

**External clock:** `ext_clock.c` validated intervals override tap; timeout falls back to internal tempo.

### BETA / OMEGA selection (mode families)

Mode-family selection reuses the **same** op-mode change state machine as modes 1–10 (`handle_op_mode_sm` in `input_handler.c`). There is **no** long-press on Mode to enter a separate UI.

| Path | Condition | Confirm callback |
|------|-----------|------------------|
| ALPHA (1–10) | User qualifies with Tap hold ≥ `OP_MODE_TAP_HOLD_DURATION_MS`, releases Tap **before** `OP_MODE_TAP_OMEGA_HOLD_MS` from **first** press | `op_mode_change_cb(N)` with N = 1…10 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sitar93/krono](https://github.com/sitar93/krono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
