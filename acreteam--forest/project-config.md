---
trigger: always_on
description: This repository is a work-in-progress PC port of Animal Crossing (GameCube).
---

# AGENTS.md — Animal Crossing GameCube PC port

## Project Overview

This repository is a work-in-progress PC port of Animal Crossing (GameCube).
It is based on a complete matching decompilation of the game and is currently in progress of being ported from the PowerPC/MWCC/GC platform to x86 (not amd64 yet) clang/msvc/gcc and windows/linux.

### Supported Game Versions

| ID           | Region      |
|--------------|-------------|
| `GAFE01_00`  | USA Rev 0 (default) |
| `GAFU01_00`  | Australia Rev 0     |

---

## Repository Layout

```
forest/
├── configure.py          # Master build configuration script (generates build.ninja)
├── build.ninja           # Generated ninja build file
├── compile_commands.json # Compilation database for IDE/tooling support
├── objdiff.json          # Configuration for objdiff (binary diffing tool)
│
├── config/
│   ├── GAFE01_00/        # USA version configuration
│   │   ├── config.yml    # dtk decomp config (DOL hash, symbol/split info, relocations)
│   │   ├── symbols.txt   # Known symbol addresses
│   │   ├── splits.txt    # Section split definitions
│   │   ├── build.sha1    # Expected output hash for verification
│   │   ├── ldscript.tpl  # Linker script template
│   │   └── foresta/      # REL module configuration (the main game REL)
│   └── GAFU01_00/        # Australia version configuration
│
├── orig/
│   ├── GAFE01_00/        # Original disc image files (user-supplied, not in repo)
│   └── GAFU01_00/
│
├── build/                # Build output directory
│   ├── compilers/        # Downloaded Metrowerks CodeWarrior compilers
│   ├── binutils/         # Downloaded GNU binutils for PowerPC
│   ├── tools/            # Downloaded decomp tools (dtk, sjiswrap, objdiff-cli, orthrus)
│   └── GAFE01_00/        # Per-version build artifacts
│
├── assets/               # Project assets (e.g. objdiff screenshot)
│
├── include/              # All header files
├── src/                  # All source files
└── tools/                # Python build tooling and utilities
```

---

## Source Code Structure (`src/`)

The source is split into two major linkage units: the **static DOL** (the main executable) and the **foresta REL** (a relocatable module loaded at runtime containing all game logic).

### Static DOL (`src/static/` and some top-level `src/` files)

The DOL contains system-level code, SDK libraries, and the boot/init path.

| Directory / File                 | Description |
|----------------------------------|-------------|
| `src/static/boot.c`             | System bootstrap and initialization |
| `src/static/jsyswrap.cpp`       | JSystem wrapper layer for C code |
| `src/static/version.c`          | Version identification |
| `src/static/initial_menu.c`     | Initial menu screen |
| `src/static/dvderr.c`           | DVD error handling |
| `src/static/bootdata/`          | Boot data (logo textures, window data) |
| `src/static/nintendo_hi_0.c`    | Nintendo logo data |
| `src/static/GBA2/`              | GBA Joy Boot link support |
| **`src/static/libforest/`**     | **Forest engine library** |
| `src/static/libforest/emu64/`   | N64-to-GC graphics command emulation layer (emu64) |
| `src/static/libforest/osreport.c` | OS report wrapper |
| `src/static/libforest/fault.c`  | Crash/fault handler |
| `src/static/libforest/ReconfigBATs.c` | BAT register reconfiguration |
| `src/static/libu64/`            | N64 compatibility library (debug, gfxprint, pad) |
| `src/static/libc64/`            | C64 utility library (malloc, math, printf, random) |
| `src/static/libultra/`          | N64 Ultra SDK reimplementation for GameCube |
| `src/static/libultra/gu/`       | Graphics utility math functions (matrix, trig, etc.) |
| `src/static/libjsys/`           | JSystem wrapper extensions |
| `src/static/dolphin/`           | **Dolphin SDK** (Nintendo's official GC SDK) |
| `src/static/dolphin/ai/`        | Audio interface |
| `src/static/dolphin/ar/`        | ARAM (auxiliary RAM) |
| `src/static/dolphin/card/`      | Memory card |
| `src/static/dolphin/db/`        | Debug |
| `src/static/dolphin/dsp/`       | DSP (audio processor) |
| `src/static/dolphin/dvd/`       | DVD drive |
| `src/static/dolphin/exi/`       | Expansion interface |
| `src/static/dolphin/gba/`       | GBA link |
| `src/static/dolphin/gx/`        | **Graphics (GX)** — the GC's GPU API |
| `src/static/dolphin/mtx/`       | Matrix/vector math |
| `src/static/dolphin/os/`        | OS services (threads, interrupts, cache, memory, RTC, etc.) |
| `src/static/dolphin/pad/`       | Controller input |
| `src/static/dolphin/si/`        | Serial interface |
| `src/static/dolphin/vi/`        | Video interface |
| `src/static/dolphin/OdemuExi2/` | DEV debug interface |
| `src/static/JSystem/`           | **JSystem** libraries (J2D, J3D, JFramework, JKernel, JUtility, etc.) |
| `src/static/Famicom/`           | NES/Famicom emulator (for playable NES games) |
| `src/static/jaudio_NES/`        | NES audio system (jaudio) |
| `src/static/MSL_C.PPCEABI.bare.H/` | Metrowerks Standard Library (C runtime) |
| `src/static/Runtime.PPCEABI.H/` | PowerPC EABI runtime support |
| `src/static/TRK_MINNOW_DOLPHIN/` | Target Resident Kernel (debugger support) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ACreTeam/forest](https://github.com/ACreTeam/forest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
