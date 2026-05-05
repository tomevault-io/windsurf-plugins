---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Sandopolis is a portable and accurate multi-system Sega emulator for Genesis, Master System, Game Gear, and SG-1000, written in Zig (and C).
Priorities, in order:

1. Correct emulation behavior and compatibility.
2. Clear timing and subsystem interactions.
3. Maintainable boundaries between frontend and core emulation logic.
4. Performance, but only after correctness is covered by tests.

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over broad rewrites.
- Ensure the project is modular and components are decoupled with clean APIs and interfaces.
- Keep emulator state instance-bound inside the existing structs (`Bus`, `Vdp`, `Io`, `Z80`, `Cpu`, and `AudioOutput`).
- Avoid introducing a new global mutable state.
- Keep SDL/frontend logic in `src/main.zig` and `src/frontend/`; keep core emulation logic in `src/`.
- Keep SDL3 portable in the build graph: use the Zig-native SDL3 build (`castholm/SDL`) plus `zsdl` bindings, and do not hard-code platform binary
  artifacts such as `lib/libSDL3.so` in `build.zig`.
- Add comments only when they clarify non-obvious hardware behavior or timing.

Quick examples:

- Good: add a VDP timing helper inside `src/video/vdp.zig` with module-local tests.
- Good: add a regression ROM boot assertion in `tests/regression_tests.zig`.
- Bad: move emulator core behavior into `src/main.zig`.
- Bad: add a new debug-only behavior path without tests.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "TCP proxy" not "lightweight TCP proxy", "scoring components" not "transparent scoring components".
- Use noun phrases for checklist items, not imperative verbs. Write "redundant index detection" not "detect redundant indexes".
- Headings in Markdown files must be in the title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in,
  on, at, to, by, of) stay lowercase unless they are the first word.

## Repository Layout

- `src/main.zig`: SDL frontend, event loop, rendering, audio device setup.
- `src/api.zig`: API/doc entrypoint used for generated documentation.
- `src/public/`: deliberate public API facade type (`Machine`) exposed from `src/api.zig`.
- `src/testing/`: explicit testing facade used by non-unit suites that need lower-level control than `Machine` alone exposes.
- `src/testing_root.zig`: root module used for internal testing facades that need broader access than `src/testing/` alone.
- `src/bus/`: cartridge loading/persistence, memory map, open-bus behavior, Z80 arbitration, and VDP/audio timing coordination.
- `src/scheduler/`: frame/master-clock scheduling.
- `src/cpu/`: 68K/Z80 wrappers, runtime hooks, CPU-facing memory interface, and the local jgz80 bridge C code.
- `src/audio/`: sample-based YM2612 FM synthesizer (`ym2612_sample.zig`, runtime), cycle-accurate Nuked OPN2 core (`ym2612.zig`, validation), SN76489
  PSG with bipolar output, blip-buffer band-limited resampler, board analog LPF, 3-band parametric equalizer, and the output mixing pipeline.
- `src/input/`: controller I/O and configurable input mapping.
- `src/recording/`: GIF animation recording with LZW compression and crash-safe output, WAV audio recording, and BMP screenshot capture.
- `src/video/`: VDP and video timing/rendering logic.
- `src/sms/`: Sega Master System and Game Gear emulation: VDP Mode 4, memory bus with Sega mapper, I/O port dispatch, Z80-based machine, PSG-only
  audio, GG 12-bit CRAM, and GG viewport cropping.
- `src/frontend/`: SDL frontend helpers including config, UI state, save manager, menu, dialog, toast, and performance overlay logic.
- `src/unit_test_root.zig`: internal test root that aggregates module-local unit tests for `zig build test-unit`.
- `src/wasm.zig`: WebAssembly export layer wrapping the Machine API for browser deployment.
- `src/wasm_stubs.c`: minimal C stubs (setjmp/longjmp and main) needed for WASM builds.
- `src/libretro.zig`: Libretro core shared library wrapping the Machine API for RetroArch and other Libretro frontends.
- `src/system.zig`: system type detection (Genesis, SMS, or Game Gear) from ROM headers and cartridge region codes.
- `src/system_machine.zig`: `SystemMachine` tagged union abstracting Genesis and SMS/GG for the SDL frontend.
- `src/rom_loader.zig`: ROM file loading with ZIP archive extraction (deflate and stored).
- `src/`: remaining core emulator modules (`machine.zig`, `cli.zig`, `performance_profile.zig`, `rom_metadata.zig`, `state_file.zig`, etc.).
- `tests/`: non-unit suites only:
    - `integration_tests.zig`
    - `regression_tests.zig`
    - `property_tests.zig`
- `tests/testroms/`: local (public-domain and community) ROMs for testing and hardware verification; see `tests/testroms/README.md`.
- `roms/`: local ROMs for manual testing only; this directory may be absent.
- `tools/`: developer-only utilities that are not part of the shipped emulator runtime.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pixel-clover/sandopolis](https://github.com/pixel-clover/sandopolis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
