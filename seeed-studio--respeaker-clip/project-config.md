---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit Rules

- Do not add `Co-Authored-By` lines to commit messages.
- Code must compile with zero warnings. Fix all compiler warnings before committing.

## Project Overview

reSpeaker Clip is a Zephyr RTOS firmware project for the Seeed reSpeaker Clip board, based on the Nordic nRF5340 dual-core MCU. It is a voice recording device with BLE, WiFi AP, and USB connectivity, AT command control, and UDP file transfer.

- **RTOS**: Zephyr RTOS v3.3.0 (via Nordic nRF Connect SDK) — active on `main`. v3.2.1 is no longer supported (`main` requires v3.3.0-only Kconfig).
- **Hardware**: nRF5340 (dual-core: Application core + Network core)
- **Key Features**: PDM microphone array, OLED display (CH1115), SD card, WiFi (nRF7002), external SPI flash, haptic motor, battery monitoring (NPM1300 + nRF Fuel Gauge, custom "240"/HSZ 362123 model), USB CDC serial + USB MSC (SD card mass storage)

This repo (`module.yml` → `board_root`/`dts_root`) also carries the lineage of the related **reSpeaker Lav** lavalier product (see the `reSpeaker Lav/` tree, the `240` battery, and DTS comments referencing "Lav"). The Clip is the active target.

## Environment Setup

Active development uses **NCS v3.3.0** (`main` is the active branch):
```sh
source ~/ncs/v3.3.0/zephyr/zephyr-env.sh
export ZEPHYR_EXTRA_MODULES=$(pwd)
```

`ZEPHYR_EXTRA_MODULES` must be an environment variable (not CMake), because Kconfig module discovery happens before CMake configuration.

> **v3.2.1 is dropped.** `main` migrated to v3.3.0-only Kconfig (e.g. the WPA3 `..._WPA3_IMPLEMENTATION_NONE` choice, commit `099f62f`) and will no longer build against NCS v3.2.1. The `ncs/v3.3.0` branch is an older, diverged v3.3.0 line (~12 commits behind `main`); the local `master` is only the ancient initial import.

Every app on this board builds as a Zephyr **sysbuild** (MCUboot + app core + network-core radio) **by default, with no per-app sysbuild config**. The board provides it all:

- `boards/seeed/clip/Kconfig.sysbuild` — auto-sourced by sysbuild (Zephyr `hwm_v2.cmake`). Defaults `BOOTLOADER_MCUBOOT`, overwrite-only mode, dual-image OTA, `NETCORE_IPC_RADIO` (note: a `choice` symbol — set via `choice NETCORE`, not `config ... default y`), `SECURE_BOOT_NETCORE`, and the RSA signing key (`$(ZEPHYR_RESPEAKER_CLIP_MODULE_DIR)/boards/seeed/clip/sysbuild/root-rsa-2048.pem`).
- `sysbuild/CMakeLists.txt` (module root, registered via `sysbuild-cmake:` in `zephyr/module.yml`) — points the `mcuboot` and `ipc_radio` images at the board's shared config as a **fallback** (an app overrides by providing its own `<app>/sysbuild/<image>.{conf,overlay}`).
- `boards/seeed/clip/sysbuild/` — the real shared files: `mcuboot.conf`, `mcuboot.overlay`, `ipc_radio/prj.conf`, `root-rsa-2048.pem`.
- `boards/seeed/clip/pm_static_clip_nrf5340_cpuapp.yml` — auto-discovered by the NCS partition manager.

So a sample is just `CMakeLists.txt` + `prj.conf` + `src/` and still boots under the custom signed MCUboot. See `docs/custom_app_guide.md`. Pattern copied from `xiao_esp32c6`.

## Building & Flashing

```sh
# Build (incremental)
west build --build-dir build-clip --board clip/nrf5340/cpuapp applications/clip

# Build (clean)
west build --build-dir build-clip --pristine --board clip/nrf5340/cpuapp applications/clip

# Flash and reset (required: west flash --reset does NOT work on this board)
west flash --build-dir build-clip && nrfutil device reset

# View serial output
minicom -D /dev/ttyACM0 -b 921600  # Clip UART0 debug console @921600 (board default). When a J-Link probe is also connected, the J-Link takes ttyACM0 and the Clip's UART0 bridge is ttyACM1 — adjust to whichever is the "USB Single Serial" / non-J-Link port.
```

**Board identifier**: `clip/nrf5340/cpuapp` (NOT `respeaker/...`)

### Power Management

`CONFIG_PM_DEVICE_RUNTIME=y` enables automatic peripheral power management (UART/I2C/SPI suspend when idle). The debug UART console leaks ~570µA at idle; the `production` snippet disables it, reaching ~170µA. Full idle power budget, regulator map, and measurement procedure: **docs/power.md** (that file owns the power figures).

### Build Snippets

Snippets are in `applications/clip/snippets/`. Each snippet has a conf file, optional overlay, and `snippet.yml`.

| Snippet | Purpose | Changes |
|---------|---------|---------|
| `production` | Low-power production firmware | Disables UART console + UART log backend (`CONFIG_CONSOLE=n`, `CONFIG_UART_CONSOLE=n`, `CONFIG_LOG_BACKEND_UART=n`); FS log default follows (off). Idle ~170µA vs ~higher for the debug build. |

The default (no-snippet) build is the **debug** image: UART console on, FS log to `/SD:/LOG` at INF level (`CLIP_LOG_FS_DEFAULT_ON` defaults to `LOG_BACKEND_UART`). Use the `production` snippet for battery/production builds where the console leak matters.

Build with snippet: `west build ... -- -DSNIPPET_ROOT=$(pwd)/applications/clip -DSNIPPET=production` (under sysbuild the app dir is not searched for snippets — SNIPPET_ROOT must point at it, absolute path).

### Output Firmware


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Seeed-Studio/reSpeaker_Clip](https://github.com/Seeed-Studio/reSpeaker_Clip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
