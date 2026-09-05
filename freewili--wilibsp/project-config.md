---
trigger: always_on
description: > **Reading requirement:** read this file through EOF before changing code.
---

# AGENTS.md — guide for AI agents and contributors

> **Reading requirement:** read this file through EOF before changing code.
> It is intentionally long. If your tool truncates it, continue from the last
> line in additional chunks until EOF. Do not treat the first chunk as the
> complete contract. External app repositories must link here from their own
> root `AGENTS.md`; see `docs/app-project-setup.md`.

This file orients coding agents (Claude Code, Cursor, Copilot, etc.) and new
human contributors working in `wilibsp`. It is intentionally dense: the goal
is that you do **not** have to rediscover the hard-won facts this project was
built on. Read it before making changes.

## What this is

`wilibsp` is a board-support **monorepo** for the **FreeWili 2** (Raspberry Pi
**RP2350B**, 48 GPIO, 16 MB flash, 8 MB PSRAM). Importantly, today it covers the display processor only. (This means you must use OpenOCD interface 0 — FreeWili 2 exposes multiple debug interfaces.) It provides:

- `bsp/` — the shared `freewili2_bsp` CMake **STATIC library**: platform
  bring-up, display, touch, and LED drivers, harvested and normalized from the
  owner's proven repos (primarily `subghz`).
- `apps/` — individual CMake executables that link `freewili2_bsp`
  (`template` — starter scaffold; `hello_display` — v1 on-hardware smoke
  test: display renders, touch responds, LEDs light).
- `libs/` — optional static libraries apps can link in addition to the BSP.
  Today: `libs/onewili` — the generated OneWili C command API for driving the
  **main CPU** (GPIO, LEDs, radio, …) over the FwGUI display link (UART0,
  8 Mbaud), plus `ow_sd_*` for reading and writing the **SD card** the main
  CPU owns (SDFS over the same link). See `libs/onewili/README.md`;
  `apps/toggleled` and `apps/hello_sdcard` are the worked examples.
- `tools/fw.py` (+ `tools/fw` / `tools/fw.cmd` launchers) — a cross-platform
  CLI that drives CMake + OpenOCD identically on Windows and Linux.
- `tests/` — a standalone host CTest tree for pure logic (no Pico SDK, no
  hardware).

The umbrella header is `bsp/fw2.h` — include this from an app to pull in the
board + drivers.

**Status:** the v1 smoke test and every driver increment since have passed on
real hardware — display, touch, LEDs, platform, I2S audio, PDM mics, CC1101
radio, I2C sensors, DVI, and the agentio harness. The per-increment records
live in `docs/superpowers/findings/`, summarized in
`docs/hardware/facts.md` ("Hardware verification status") and tracked per
peripheral in `docs/hardware/catalog.md`. Anything still marked TODO in the
catalog (NFC, buttons, PIO-USB) is unverified because its driver has not been
harvested yet.

**Do not assume a doc's description of behavior is a confirmed result.** Where
this repo describes what something does, check whether a findings file backs
it. If none does, it is design intent — say so rather than repeating it as
fact.

## Command vocabulary

All commands run from the repo root and are identical on Windows and Linux
(the CLI is Python; `tools/fw` is the POSIX launcher, `tools/fw.cmd` the
Windows one — both just call `python tools/fw.py "$@"`).

| Command             | What it does                                                                                                                            |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `fw configure`      | Configure `build/` against the pinned Pico SDK + toolchain (`--clean` wipes it first). Rarely needed directly — `fw build` calls it.    |
| `fw build [app]`    | Configure + build `apps/<app>` for the RP2350B target via `cmake --build --preset target --target <app>` (default app: `hello_display`) |
| `fw flash [app]`    | Program `build/apps/<app>/<app>.elf` over the cmsis-dap debug probe via OpenOCD (`tools/openocd/freewili2.cfg`). **Refuses an ELF whose loadable segments sit in QSPI flash** — that would replace the stock DISPLAY firmware. Prefer `fw install-app`; override with `--replace-display-firmware` |
| `fw rtt`            | Attach to the target and stream SEGGER RTT diagnostics (OpenOCD RTT server on port 9090)                                                |
| `fw test`           | Configure + build + run the standalone host CTest tree in `tests/` (MinGW GCC + Ninja on Windows; no Pico SDK, no hardware)             |
| `fw new-app <name>` | Scaffold `apps/<name>` by copying `apps/template` and rewriting the CMake target name                                                   |
| `fw install-app <uf2> [<uf2> ...]` | Find MAIN with fwFinder, hand its SD reader to the PC, copy and flush one or more UF2s to `/apps/` (or `--folder path` beneath it), wait for writes to settle, and return the SD to MAIN without a Windows eject/unmount |
| `fw screenshot`     | Capture the screen to a PNG (`--surface lcd|dvi`, `--crop x,y,w,h`, `--scale N`) via the agentio RTT channel (verified on hardware 2026-07-26) |
| `fw press <btn>`    | Inject a button press+release (`fw hold` / `fw release` for a sustained hold) |
| `fw touch <x> <y>`  | Inject a touch tap (`--down` / `--up` for a sustained touch)              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freewili/wilibsp](https://github.com/freewili/wilibsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
