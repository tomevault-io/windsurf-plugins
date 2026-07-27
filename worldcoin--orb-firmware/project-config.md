---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with the orb-firmware repository.
---

# AGENT.md

This file provides guidance to AI coding assistants when working with the orb-firmware repository.

## Overview

This repository contains the MCU firmware for the Worldcoin Orb, built with Zephyr RTOS.

## Prime Directive

- Keep changes small and compiling.
- Prefer existing patterns; do not introduce new dependencies without asking.
- When design intent is unclear, inspect commit history before changing behavior.
- If unsure, propose 2 options with tradeoffs (don't guess).

## Path Discovery

This document uses semantic path references. Resolve them as follows:

| Reference         | How to Find                                           | Contains              |
| ----------------- | ----------------------------------------------------- | --------------------- |
| `<WORKSPACE>`     | Run `west topdir`                                     | West workspace root   |
| `<PUBLIC_REPO>`   | Find directory containing `main_board/` and `VERSION` | Public firmware repo  |
| `<PRIVATE_REPO>`  | Find directory containing `sec_board/` (if available) | Private firmware repo |
| `<MANIFEST_REPO>` | Run `west manifest --path` and get parent directory   | Active west manifest  |

**Quick discovery commands:**

```bash
# Workspace root
west topdir

# Find public repo (contains main_board/)
find "$(west topdir)" -type d -name "main_board" -path "*/orb/*" 2>/dev/null | head -1 | xargs dirname

# Check if private repo exists (contains sec_board/)
find "$(west topdir)" -type d -name "sec_board" 2>/dev/null | head -1 | xargs dirname
```

## Build System

- **RTOS**: Zephyr
- **Build Tool**: West (CMake under the hood)
- **Language**: C
- **Structure**: West workspace with multiple repositories managed by west

## Key Folders

Relative to `<WORKSPACE>`:

- `<PUBLIC_REPO>/main_board/` - Main MCU firmware source
- `<PUBLIC_REPO>/VERSION` - Firmware version file
- `<PRIVATE_REPO>/sec_board/` - Security MCU firmware (internal only)
- `boards/` - Board overlays / definitions (if present)
- `dts/` - Devicetree overlays (if present)
- `tests/` - Tests (if present)

## Environment Assumptions

- Commands are run from `<WORKSPACE>` (where `west topdir` works).
- Toolchain is installed and Zephyr env is set up.

## Build & Flash Commands

All commands are run from `<WORKSPACE>` root.

**Available boards:** `pearl_main`, `diamond_main` (main MCU), `pearl_security`, `diamond_security` (security MCU, internal only)

### Build (first time / new board)

Let's build the service image by default with warnings treated as errors to maintain code quality.

```bash
# Main board (public)
west build -b <BOARD> -d build/<BOARD> <PUBLIC_REPO>/main_board -- -DCMAKE_BUILD_TYPE="Service" -DEXTRA_COMPILE_FLAGS="-Werror"

# Example for Pearl main board
west build -b pearl_main -d build/pearl_main orb/public/main_board -- -DCMAKE_BUILD_TYPE="Service" -DEXTRA_COMPILE_FLAGS="-Werror"
```

### Incremental build

```bash
west build -d build/<BOARD>
```

### Clean rebuild (use when build system is "stuck")

```bash
west build -p always -d build/<BOARD>
```

### Release build (smaller binary, optimized)

```bash
west build -b <BOARD> -d build/<BOARD> <PUBLIC_REPO>/main_board -- -DCMAKE_BUILD_TYPE=Release
```

### Flash

```bash
west flash -d build/<BOARD>
```

### Debug

```bash
west debug -d build/<BOARD>
```

## Testing

Preferred: Twister for hardware-in-the-loop (HIL) tests.

> **Important:** AI agents should **propose** the twister command but **NOT run it automatically**. The user must review and execute it manually.

### Prerequisites

Twister requires `ZEPHYR_BASE` to be set and Zephyr scripts in `PATH`.

**Nix environment (recommended):** If using nix/direnv (`direnv allow`), this is handled automatically.

**Manual setup:** If not using nix, source from `<WORKSPACE>`:

```bash
source zephyr/zephyr-env.sh
```

### Discover Required Parameters

Before running twister, find the following:

**1. Serial port** (for test output):

```bash
# macOS
ls /dev/cu.usbserial-*

# Linux
ls /dev/ttyUSB* /dev/ttyACM*
```

**2. Debugger unique ID** (for flashing):

```bash
pyocd list
# or for ST-Link
st-info --probe
```

### Twister Command Template

Run from `<PUBLIC_REPO>/main_board/` directory:

```bash
twister -vv -ll DEBUG \
  -T . \
  -A ./../ \
  -p <BOARD>/<QUALIFIER> \
  --device-serial <SERIAL_PORT> \
  -c \
  --device-testing \
  --west-flash="-i=<DEBUGGER_ID>"
```

**Arguments explained:**

| Argument                 | Description                                         | Example                                                  |
| ------------------------ | --------------------------------------------------- | -------------------------------------------------------- |
| `-vv`                    | Very verbose output                                 |                                                          |
| `-ll DEBUG`              | Log level debug                                     |                                                          |
| `-T .`                   | Test suite root (current dir)                       |                                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [worldcoin/orb-firmware](https://github.com/worldcoin/orb-firmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
