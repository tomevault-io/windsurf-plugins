---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WeGui-ARGB is a lightweight embedded GUI framework for MCU / SoC targets plus an SDL2 PC simulator. The platform-independent GUI kernel and demos live in `Core/` and `Demo/`; each hardware/simulator target provides only the port layer, startup code, and build project.

Primary targets currently present in this repository:
- `Simulator/` — SDL2 PC simulator built with CMake + MinGW/Ninja or MinGW Makefiles.
- `STM32F103/` — Keil MDK-ARM AC5 hardware target, with LCD, input, and W25Qxx external flash ports.
- `STM32F030/` — Keil MDK-ARM AC5 hardware target, with LCD and input ports.

Full API reference: `WEGUI_API_REFERENCE.md`.

## Build Commands

### Simulator (CMake + MinGW)

Use the repository wrapper scripts rather than calling CMake directly:

```powershell
# Clean configure + build
powershell -NoProfile -ExecutionPolicy Bypass -File "Simulator/build_sim.ps1" -Clean

# Incremental build
powershell -NoProfile -ExecutionPolicy Bypass -File "Simulator/build_sim.ps1"

# Run latest built simulator
powershell -NoProfile -ExecutionPolicy Bypass -File "Simulator/run_latest_sim.ps1"
```

`Simulator/build_sim.ps1` auto-detects `ninja + gcc + g++` first and falls back to `mingw32-make + gcc + g++`. If the simulator build is stale or broken, delete `Simulator/build/` and rebuild with `-Clean`.

`Simulator/CMakeLists.txt` globs core/widget sources (`../Core/*.c` and `../Core/widgets/*/*.c`, excluding `*_bckup.c`) plus the preview zone (`../Core/widgets_preview/*/*.c` and `../Demo/preview/*.c`), but lists stable `DEMO_SOURCES` **explicitly** (demo `.c` files plus font/bin resource `.c` files). Adding a new widget or preview demo compiles automatically; adding a new stable demo requires appending its `demo_xxx.c` to `DEMO_SOURCES`.

### STM32F103 Hardware (Keil MDK-ARM AC5)

```powershell
UV4.exe -r "STM32F103\MDK-ARM\Project.uvprojx" -t "WeGui_ARGB"
```

Build log: `STM32F103/MDK-ARM/Objects/Project.build_log.htm`

### STM32F030 Hardware (Keil MDK-ARM AC5)

```powershell
UV4.exe -r "STM32F030\MDK-ARM\Project.uvprojx" -t "STM32F030"
```

Build log: `STM32F030/MDK-ARM/STM32F030/STM32F030.build_log.htm` (Keil writes F030 output into a folder named after the target, unlike F103's `Objects/`)

### VS Code Tasks

`.vscode/tasks.json` currently provides simulator tasks (`sim: stop running`, `sim: build`, `sim: clean and build`, `sim: run latest`, `sim: build and run`; the build tasks run `sim: stop running` first so the linker can overwrite a running `wegui_sim.exe`) and STM32F103 Keil tasks (`stm32: build (AC5)`, `stm32: rebuild (AC5)`, `stm32: open MDK project`). The Keil tasks depend on local VS Code settings such as `wegui.keilUv4Path`, `wegui.stm32ProjectFile`, and `wegui.stm32TargetName`.

## Tests / Validation

There is **no standalone automated test suite or lint target** in this repository. Validation is done by building a target and running one demo as an integration smoke test.

Closest equivalent to a single test:
- **Simulator**: change `#define DEMO_ID` near the top of `main` in `Simulator/main_sim.c` (`0` = showcase), rebuild, run `wegui_sim`, and verify rendering/animation/input behavior.
- **STM32F103**: change `#define DEMO_ID` near the top of `main` in `STM32F103/main.c`, rebuild/flash, and verify on the LCD.
- **STM32F030**: change `#define DEMO_ID` near the top of `main` in `STM32F030/main.c`, rebuild/flash, and verify on the LCD.

Hardware flashing is done outside the build command (CMSIS-DAP / DAPLink / pyOCD are usable depending on the board). If flashed content appears stale, verify the relevant `.build_log.htm` timestamp before reflashing.

## Architecture

### Directory Layout

- `Core/` — platform-independent GUI kernel, drawing, widget implementations, dirty-rectangle engine, image/font support.
- `Demo/` — demo applications; each widget type has its own `demo_xxx.c`, with declarations in `simple_widget_demos.h`.
- `Core/widgets_preview/` + `Demo/preview/` — **preview incubation zone** for experimental widgets (currently 26, e.g. `mask_group`, `ime_pinyin`). Compiled by all three targets (simulator via CMake globs; both Keil projects carry `we_widget_preview`/`demo_preview` file groups — the linker strips whatever the selected demo doesn't reference), DEMO_ID range 100+ (currently 101..126, resequenced 2026-07 with the 12 most-used widgets first; future graduations leave holes), ID table and demo entry declarations in `Demo/preview/preview_demos.h`, function naming `we_<name>_preview_demo_init/tick`. New preview widgets/demos are picked up automatically by the simulator globs but must be added to both `.uvprojx` file groups by hand. These widgets are unpolished and may be removed at any time; graduation moves them into `Core/widgets/` and the stable numbering range.
- `Simulator/` — SDL2 entry (`main_sim.c`), SDL LCD/input/storage port (`sdl_port.c/h`), simulator config (`we_sim_port_config.h`), and build/run scripts.
- `STM32F103/` — STM32F103 entry (`main.c`), Keil project, LCD SPI ports, button/input port, and W25Qxx external flash port.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KOUFU-DIY/wegui-argb](https://github.com/KOUFU-DIY/wegui-argb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
