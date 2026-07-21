---
trigger: always_on
description: > **AGENTS.md** is a standardized format that lets your codebase speak directly to any agentic coding tool.
---

# AGENTS.md - LuatOS AI Agent Configuration

> **AGENTS.md** is a standardized format that lets your codebase speak directly to any agentic coding tool.
> This file tells AI tools how the LuatOS project works, what conventions to follow, and where important files live.

---

## Project Overview

**LuatOS** is an embedded Lua operating system based on Lua 5.3 VM, developed by openLuat (合宙). It supports multiple hardware platforms (Air8000/Air8101/Air780E series) with 74+ core libraries, 55+ extension libraries, and 1000+ APIs.

### Tech Stack
- **Core Language**: Lua 5.3.5 (optimized by openLuat)
- **Build System**: xmake
- **Target Platforms**: Embedded MCU (ARM/RISC-V), PC Simulator (Windows/Linux/macOS)
- **License**: MIT License

---

## Agent Persona

You are a **LuatOS Development Expert** with deep knowledge of:
- Embedded systems programming in C and Lua
- Lua 5.3 VM internals and C API
- RTOS concepts (task scheduling, event loops, concurrency)
- Hardware abstraction layers (GPIO, UART, SPI, I2C, ADC, PWM)
- Network protocols (TCP/UDP, HTTP, MQTT, WebSocket, CoAP)
- GUI frameworks (LVGL, embedded displays)
- Build systems (xmake, cross-compilation)

---

## Key Directories

```
LuatOS/
├── lua/              # Lua VM source (based on 5.3.5)
├── luat/             # LuatOS core framework
│   ├── include/      # Core C headers
│   ├── modules/      # C implementations of Lua libraries
│   ├── vfs/          # Virtual file system
│   └── weak/         # Weak reference implementations
├── components/       # Extension libraries (90+ components)
│   ├── network/      # Network stacks (LwIP, MQTT, HTTP, NDK)
│   ├── airui/        # GUI framework (LVGL 9 based)
│   ├── ndk/          # NDK socket adapter (RV32C sim + hosting)
│   ├── pgfs/         # PGFS flash filesystem
│   ├── luat_image/   # Unified image decoding (JPG/PNG/WebP)
│   ├── mbedtls/      # Cryptography library
│   └── ...           # Bluetooth, audio, sensors, etc.
├── bsp/              # Board Support Packages
│   ├── pc/           # PC simulator (xmake build)
│   └── [model]/      # Hardware-specific firmware/demos
├── module/           # Module firmware and solutions
├── app_engine/       # App engine — factory firmware + app store
│   ├── factory/      # Default factory firmware image
│   └── app_store/    # Pre-packaged demo apps (horizontal/vertical)
├── script/           # Lua script libraries
│   ├── corelib/      # Core libraries (sys.lua, etc.)
│   ├── libs/         # External driver libraries
│   └── turnkey/      # Ready-to-use project templates
├── testcase/         # Test suites (utest + feature tests)
├── tools/            # Auxiliary tools
├── docs/             # Documentation (known issues, RFA, VFS)
└── bsp/pc/build/     # PC simulator build output
```

---

## Component AGENTS.md Standard

Several sub-directories carry their own `AGENTS.md` — `luat/`, `script/`, `testcase/`, `bsp/pc/`, `components/network/`, `components/airui/`, `components/airlink/`, `components/luat_image/`, `components/pgfs/`, `components/serialization/protobuf/`, `components/utest/`, `components/ndk/`, `testcase/func/appstore/`, `testcase/utest/fs/vfs_uniform/`. Use the following rule to decide whether a new component needs one.

**Create a component-level `AGENTS.md` when ANY of these is true:**
- The component has a non-obvious coding or build convention that an AI would otherwise get wrong (e.g. `bsp/pc/` GUI vs non-GUI build path, `components/airlink/` nanopb include ordering).
- The component has accumulated **3+ recurring, worth-recording pitfalls** that are not covered by the root `AGENTS.md` (e.g. `components/serialization/protobuf/` ARM stack-slot debugging).
- The component has a multi-step recipe that AI agents must follow in a specific order (e.g. `components/ndk/` regression chain).

**Do NOT create a component-level `AGENTS.md` when:**
- The component is a thin wrapper around a Lua API and inherits conventions from `luat/` — point to the root file or to `script/AGENTS.md` instead.
- The content would just duplicate the root `AGENTS.md` Build & Test Commands section.
- The component is not yet stable (active refactor in progress) — wait until the conventions settle.

**Style requirements for any new `AGENTS.md`:**
- Match the encoding & line-ending of the surrounding tree (currently mixed LF/CRLF — check `git show HEAD:<dir>/README.md` first).
- Use the canonical section shape: `## Scope` → `## Where to Look` → conventions → recipes → `## Anti-Patterns` → `## Related Docs`.
- Keep it a **single point of authority**: if the root `AGENTS.md` references the component's pitfalls, replace the duplicated bullets with a single `See <path> § <section>` pointer.

---

## Build & Test Commands

### PC Simulator (Development & Testing)

See `bsp/pc/AGENTS.md` for detailed PC Simulator build and development instructions.

**Compilation rule:**
- **NEVER run `xmake -y` directly** — it triggers a full rebuild (10+ min) and the output gets truncated in the shell tool, making it impossible to read errors.
- **ALWAYS use the helper batch scripts** in `bsp/pc/` for Windows:
  ```powershell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openLuat/LuatOS](https://github.com/openLuat/LuatOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
