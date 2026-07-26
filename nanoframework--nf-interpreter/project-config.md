---
trigger: always_on
description: This is the **nf-interpreter** repository — the C/C++ firmware for **.NET nanoFramework**, a managed-code runtime (CLR) for resource-constrained embedded devices. It produces two binaries per target board: **nanoBooter** (bootloader, optional) and **nanoCLR** (the CLR that executes .NET managed applications).
---

# Copilot Cloud Agent Instructions for nf-interpreter

## Project Overview

This is the **nf-interpreter** repository — the C/C++ firmware for **.NET nanoFramework**, a managed-code runtime (CLR) for resource-constrained embedded devices. It produces two binaries per target board: **nanoBooter** (bootloader, optional) and **nanoCLR** (the CLR that executes .NET managed applications).

The project targets multiple RTOS platforms (ESP32/ESP-IDF, ChibiOS, FreeRTOS, TI SimpleLink, Eclipse ThreadX) and MCU architectures (ARM Cortex-M, Xtensa, RISC-V).

## Repository Structure

```
src/
├── CLR/              # Common Language Runtime engine
│   ├── Core/         # Heap, GC, type system, execution engine
│   ├── Include/      # Core CLR headers (nanoCLR_Runtime.h, nanoCLR_Types.h, etc.)
│   ├── CorLib/       # Core library native implementations
│   ├── Debugger/     # Debug engine
│   ├── Startup/      # CLR initialization
│   ├── Messaging/    # Wire protocol messaging
│   ├── WireProtocol/ # Debug/deployment wire protocol
│   ├── Helpers/      # Utilities (Base64, nanoprintf, ring buffer)
│   └── Diagnostics/  # Diagnostic support
├── HAL/              # Hardware Abstraction Layer (low-level HW interfaces)
│   └── Include/      # HAL headers (nanoHAL.h, nanoHAL_Boot.h, etc.)
├── PAL/              # Platform Abstraction Layer (OS/platform abstraction)
│   ├── BlockStorage/ # Storage management
│   ├── COM/          # Communications (sockets)
│   ├── Events/       # Event management
│   └── ...
├── DeviceInterfaces/ # Device interface definitions
├── System.Device.*/  # Native implementations for managed device APIs
│                     # (Gpio, I2c, Spi, Pwm, Adc, Dac, I2s, Wifi, etc.)
└── nanoFramework.*/  # Native implementations for nanoFramework-specific APIs
                      # (Graphics, Collections, Security, CAN, OneWire, etc.)

targets/              # Per-RTOS and per-board implementations
├── ESP32/            # ESP-IDF based targets (ESP32, S2, S3, C3, C5, C6, H2, P4)
│   └── defconfig/    # Board defconfig files (Kconfig fragments)
├── ChibiOS/          # ChibiOS RTOS targets (STM32 boards)
├── ThreadX/          # Eclipse ThreadX targets (STM32, Nordic, SiliconLabs, etc.)
├── FreeRTOS/         # FreeRTOS targets (NXP)
└── TI_SimpleLink/    # TI SimpleLink targets

CMake/                # CMake toolchain files, presets, and build modules
├── Modules/          # Per-target GCC options and source file lists
├── toolchain.*.cmake # Cross-compilation toolchain files
└── binutils.*.cmake  # Platform-specific build utilities

config/               # User-local config templates (git-ignored when copied)
Kconfig*              # Kconfig configuration system (feature flags, APIs, RTOS, etc.)
```

## Build System

### Prerequisites

- **CMake 3.31+** and **Ninja** build system.
- Cross-compilation toolchains (ARM GCC, Xtensa GCC, RISC-V GCC) depending on target.
- Platform SDKs (ESP-IDF v5.5.4, ChibiOS, ThreadX, etc.).
- Python 3 with `kconfiglib>=14.1.0` (from `requirements.txt`).
- **Dev containers** are the recommended way to build. Pre-built images:
  - `ghcr.io/nanoframework/dev-container-all` (all platforms)
  - `ghcr.io/nanoframework/dev-container-esp32`
  - `ghcr.io/nanoframework/dev-container-chibios`
  - `ghcr.io/nanoframework/dev-container-azure-rtos` (ThreadX)
  - `ghcr.io/nanoframework/dev-container-freertos-nxp`
  - `ghcr.io/nanoframework/dev-container-ti`

### Building Firmware

The build uses **CMake presets** combined with a **Kconfig** configuration system:

```bash
# 1. Configure (pick a target preset, e.g. ESP_WROVER_KIT)
cmake --preset=ESP_WROVER_KIT -DCMAKE_BUILD_TYPE=Debug

# 2. Build
cmake --build build
```

Each target preset is defined in `targets/<RTOS>/CMakePresets.json` and references a `defconfig` file that sets all Kconfig options (APIs, features, RTOS selection) for that board.

### Key Build Concepts

- **Target presets** inherit from toolchain presets (in `CMake/*.json`) and user config (in `config/`).
- **defconfig files** are Kconfig fragments specifying board features (e.g., `targets/ESP32/defconfig/ESP_WROVER_KIT_defconfig`).
- **User local config**: Copy `config/user-prefs.TEMPLATE.json` → `config/user-prefs.json` and `config/user-tools-repos.TEMPLATE.json` → `config/user-tools-repos.json`, then `config/user-kconfig.conf.TEMPLATE` → `config/user-kconfig.conf` for local overrides. These are git-ignored.
- **Out-of-source builds are mandatory** — building in the source tree is a fatal error.
- **Local builds are supported** when the developer machine has the required toolchains (ARM GCC, Xtensa GCC, RISC-V GCC) and RTOS SDKs installed. The dev containers are the easiest path, but not the only one.
- **Before initiating a build**, check with the user whether they want to build locally (assuming tools are installed) or use a dev container. Do not assume one approach — ask first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanoframework/nf-interpreter](https://github.com/nanoframework/nf-interpreter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
