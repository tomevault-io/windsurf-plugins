---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Style
- This codebase should be readable by somebody who's not familiar with the NES or emulation, so add more comments and documentation than you might otherwise do, and explain context.

## Essential Commands

- `npm test` - Run code formatting check and tests (required before commits)
- `npm run build` - Build distribution files (`dist/jsnes.js` and `dist/jsnes.min.js`)
- `npm run format` - Auto-format all code (core and web) with Prettier
- `npm run test:watch` - Run tests in watch mode for development
- `npm run typecheck` - TypeScript type checking (verifies `.d.ts` files)
- `node bench.js` - Run performance benchmark (~1800 fps baseline). Run this on major changes or when you suspect a performance regression/improvement.

## Code Architecture

JSNES is a JavaScript NES emulator with component-based architecture mirroring actual NES hardware:

### Core Components (all in `src/`)

**Main Orchestrator**: `nes.js` - Central class that coordinates all emulation components. Accepts callback functions for frame rendering, audio output, and status updates.

**CPU**: `cpu.js` - Implements 6502 processor with 64KB address space, instruction execution, and interrupt handling (NMI, IRQ, reset).

**PPU**: `ppu.js` - Picture Processing Unit handles 256x240 graphics rendering, VRAM management, background/sprite rendering, and scrolling.

**PAPU**: `papu.js` - Audio Processing Unit implements NES's 5 audio channels (2 square waves, triangle, noise, DMC) with 44.1kHz/48kHz sample generation.

**Memory Mappers**: `mappers.js` - Implements cartridge memory mappers (0-180) using inheritance hierarchy. All mappers inherit from Mapper 0 and override specific banking/memory mapping behavior.

**ROM Loader**: `rom.js` - Parses iNES format ROM files, extracts PRG-ROM/CHR-ROM, and determines appropriate mapper.

**Tile Renderer**: `tile.js` - Handles 8x8 pixel tile rendering with sprite flipping (horizontal/vertical), alpha-blending priority, and scanline-based rendering.

**Controller**: `controller.js` - Button state management for 2 controllers with 8 buttons each (A, B, SELECT, START, UP, DOWN, LEFT, RIGHT) and serial strobe protocol.

**Utilities**: `utils.js` - Helper functions for state serialization (`toJSON()`/`fromJSON()`) used across CPU, PPU, PAPU, and mappers for save state support.

### Key Architectural Patterns

- **Event-driven design**: Main NES class uses callbacks (`onFrame`, `onAudioSample`, `onStatusUpdate`, `onBatteryRamWrite`)
- **Component separation**: Each NES subsystem is a separate class with clear interfaces
- **Inheritance for mappers**: Code reuse while supporting cartridge-specific behavior
- **Frame-based execution**: 60 FPS timing with proper CPU cycle counting

### Usage Pattern
```javascript
var nes = new jsnes.NES({
  onFrame: function(frameBuffer) { /* render 256x240 pixels */ },
  onAudioSample: function(left, right) { /* play audio */ }
});
nes.loadROM(romData);
nes.frame(); // Execute one frame
nes.buttonDown(1, jsnes.Controller.BUTTON_A); // Handle input
```

## Testing

Tests use `node:test` + `node:assert/strict` (Node's built-in test runner):
- `test/nes.spec.js` - Basic initialization, ROM loading, frame generation with regression testing using `croom.nes` test ROM, frame buffer validation, error handling
- `test/cpu.spec.js` - Comprehensive CPU instruction testing (based on upstream wedNESday and nintengo test suites)
- `test/mappers.spec.js` - Mapper functionality tests (ROM protection, SRAM writes, RAM mirroring)
- `test/nestest.spec.js` - Full nestest.nes ROM harness with detailed error code mappings for all CPU instructions and addressing modes
- `test/accuracycoin.spec.js` - AccuracyCoin test ROM harness (134 hardware accuracy tests)

Test ROMs:
- `roms/croom/` - Simple test ROM for automated testing
- `roms/AccuracyCoin/` - Comprehensive accuracy test ROM (134 tests covering CPU, PPU, APU behavior)
- `roms/nestest/` - CPU instruction test ROM (official + unofficial opcodes)
- `local-roms/` - Collection of ROMs for manual testing

Known AccuracyCoin failures are listed in the `KNOWN_FAILURES` object in `test/accuracycoin.spec.js`.

Remember that AccuracyCoin and nestest are DEFINITELY correct. They pass on a real NES. Don't blame the ROM for being wrong.

## Build Process

Webpack configuration creates UMD modules compatible with browsers and Node.js:
- Entry point: `src/index.js` (exports NES and Controller classes)
- Output: `dist/jsnes.js` (regular) and `dist/jsnes.min.js` (minified)
- Includes ESLint checking and source map generation
- Library name: `jsnes` (global variable in browsers)
- TypeScript type definitions: `src/nes.d.ts` and `src/controller.d.ts` provide public API types
- CI: GitHub Actions (`.github/workflows/ci.yaml`) runs build and tests on push/PR with Node.js 22.x

## Code Quality Requirements

- All code must be formatted with Prettier (enforced by test suite)
- ESLint rules are enforced during build
- Tests must pass before commits
- Frame buffer regression tests prevent rendering regressions

## Documentation and reference for the NES


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bfirsh/jsnes](https://github.com/bfirsh/jsnes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
