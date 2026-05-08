---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Apple //e Browser Based Emulator - A cycle-accurate Apple II Enhanced emulator running in the browser using WebAssembly (C++ backend) and WebGL rendering. No JavaScript frameworks; vanilla ES6 modules with Vite for bundling.

## Build Commands

```bash
npm install           # Install dependencies
npm run build:wasm    # Build WASM module (required first time and after C++ changes)
npm run dev           # Start dev server at localhost:3000 (hot-reload for JS only)
npm run build         # Full production build (WASM + Vite bundle)
npm run clean         # Clean build artifacts
npm run deploy        # Deploy to VPS via rsync
```

## Testing

All tests use the Catch2 framework and are built/run via CMake's native build:

```bash
mkdir -p build-native && cd build-native
cmake ..
make -j$(sysctl -n hw.ncpu)
ctest --verbose
```

Test suites cover CPU (6502/65C02), memory (MMU, slots), video, audio, disk images (DSK/WOZ/GCR), expansion cards (Disk II, Mockingboard, Thunderclock, Mouse, SmartPort, SSC), filesystems (DOS 3.3, ProDOS, Pascal), BASIC tokenizer/detokenizer, assembler, disassembler, keyboard, condition evaluator, and full emulator integration.

## Architecture

### Two-Layer Design

**C++ Core (src/core/)** - Pure emulation logic compiled to WebAssembly:

- `cpu/6502/cpu6502.cpp` - Cycle-accurate 65C02 processor (1.023 MHz)
- `mmu/mmu.cpp` - 128KB memory management, soft switches ($C000-$CFFF), expansion slots
- `video/video.cpp` - TEXT/LORES/HIRES/DHIRES per-scanline rendering
- `audio/audio.cpp` - Speaker emulation from $C030 toggles
- `disk-image/` - Disk image format support (DSK/DO/PO/NIB/WOZ) with GCR encoding
- `disassembler/` - 65C02 instruction disassembler
- `input/keyboard.cpp` - Keyboard input handling
- `cards/` - Pluggable expansion card system (ExpansionCard interface)
- `cards/disk2/` - Disk II controller card
- `cards/mockingboard/` - AY-3-8910 sound chip + VIA 6522 timer + Mockingboard card
- `cards/mouse/` - Apple Mouse Interface Card
- `cards/smartport/` - SmartPort hard drive controller (2 block devices, self-built ROM)
- `cards/softcard/` - Microsoft Z-80 SoftCard with Z80 CPU emulation
- `cards/ssc/` - Super Serial Card with ACIA 6551
- `cards/thunderclock/` - Thunderclock Plus real-time clock card
- `filesystem/` - DOS 3.3 and ProDOS filesystem parsers
- `basic/` - Applesoft and Integer BASIC detokenizer and tokenizer
- `debug/` - Condition evaluator for breakpoint expressions (supports BV/BA/BA2 for BASIC variable/array reads)
- `noslot_clock.cpp` - DS1215 No-Slot Clock (ProDOS RTC at $C300)
- `emulator.cpp` - Core coordinator
- `emulator/emulator_state.cpp` - State serialization (exportState/importState)
- `emulator/emulator_debug.cpp` - Debug facilities (breakpoints, watchpoints, trace, beam)

**JavaScript Layer (src/js/)** - Browser integration:

- `main.js` - AppleIIeEmulator class orchestrating all subsystems
- `worker/` - Web Worker infrastructure for WASM isolation (see Worker Architecture below)
- `audio/` - Web Audio API driver and AudioWorklet
- `display/` - WebGL renderer, CRT shader effects, display settings, screen window
- `disk-manager/` - Disk drive UI, SmartPort hard drives, persistence, surface rendering, drive sounds
- `file-explorer/` - DOS 3.3 and ProDOS disk browser with disassembler
- `debug/` - Debug window implementations (see Debugging section)
- `help/` - Documentation and release notes windows
- `input/` - Keyboard input, text selection, joystick, mouse
- `ui/` - Menu wiring, reminders, slot configuration, custom confirm dialogs
- `state/` - State serialization and persistence (autosave + 5 manual slots)
- `config/` - App version
- `utils/` - Shared utilities (storage, string, BASIC)
- `windows/` - Base window class and window manager

### Theming

Light, dark, and system-follow themes controlled by `ThemeManager` (`src/js/ui/theme-manager.js`). Sets `data-theme` attribute on `<html>` for CSS variable switching. All accent and syntax highlighting colours are derived from the six-stripe Apple rainbow logo palette (Green `#61BB46`, Yellow `#FDB827`, Orange `#F5821F`, Red `#E03A3E`, Purple `#963D97`, Blue `#009DDC`), with brightness adjusted per theme for contrast. Speaker, Mockingboard, and disk drive sound volumes are all wired to a single main volume slider with a unified mute toggle.

Control sytles, sizes and layout must be consistent across the entire app.

### Worker Architecture

The WASM emulator runs in a dedicated Web Worker to keep the main thread free:

```
Main Thread                    Worker Thread                AudioWorklet Thread
-----------                    -------------                -------------------
WasmProxy (ES6 Proxy)  ←msg→  emulator-worker.js           audio-worklet.js
  - WebGL renderer               - WASM module                - ring buffer playback
  - Debug windows                 - audio generation           - requests samples
  - Input capture                 - framebuffer copy             when buffer low
  - Agent tools                   - RPC handler
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikedaley/web-a2e](https://github.com/mikedaley/web-a2e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
