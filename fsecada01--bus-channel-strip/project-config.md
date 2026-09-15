---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This document provides context and guidelines for AI assistance with the bus channel strip plugin development.

# Extended AI Session Context
@docs/SYSTEM_PROMPT.md

## Project Overview

A multi-module bus channel strip VST plugin built with NIH-Plug and Airwindows-based DSP modules in Rust. **Currently at v1.0.0** (see GitHub releases for v1.0.0 notes).

**Signal Flow**: `[API5500 EQ] → [ButterComp2] → [Pultec EQ] → [Dynamic EQ] → [Transformer] → [Haas] → [Punch] → [Sheen]`

The first seven modules occupy reorderable slots driven by the `module_order_*` params. **Sheen** is pinned to the master end of the chain (post-Punch, pre-master-gain) and is not a slot module — it's a chassis-level "polish coat" exposed only via the brushed-brass brand plate that flips into a hidden back view.

**Current Status (v1.0.0)**:
- ✅ ALL 7 SLOT MODULES + SHEEN POLISH STAGE IMPLEMENTED
- ✅ MULTI-FX RACK REDESIGN: native vizia drag-drop, swap-or-insert hit-test, live drop preview, floating ghost label, focus mode (1-7 / Esc), library sidebar as sole add path
- ✅ BRUSHED-BRASS BRAND PLATE → SHEEN BACK VIEW (mutually exclusive with DynEQ back view)
- ✅ ~86 AUTOMATION PARAMETERS
- ✅ LOCAL BUILD, BUNDLE, AND DEPLOY WORKING
- ✅ SUCCESSFUL VST3 AND CLAP BUNDLE CREATION
- 🔧 CI/CD pipeline needs bundle command fixes

## Workflow: Worktrees vs. Direct Branching

Default to working directly in the main repo checkout (`C:\dev\projects\vst3s\bus_channel_strip`), using plain `git branch` / `git checkout` to switch work — not a worktree.

Reserve worktrees for when work genuinely needs to run in parallel (e.g. multiple subagents/sessions active on different branches at once) or when deliberately A/B-ing two approaches side by side. Outside those cases, a worktree just adds a second checkout that can silently drift out of sync with `main` (see `xtask bundle`/`deploy` resolving to the wrong checkout entirely when invoked from a nested worktree) and needs its own manual `git pull`/cleanup.

## Development Guidelines

### Audio Processing Requirements
- All real-time audio processing must be **lock-free** and **allocation-free**
- Parameters must be automation-safe and uniquely identified
- Use `#[derive(Params)]` for parameter bindings

### DSP Implementation
- Implement math shaping functions in `src/shaping.rs` for reuse across modules
- Common shaping functions:
  - `sigmoid(x)` / `tanh(x)` for soft knees and saturation
  - `poly(x) + log(x)` for filter or tone control curves
  - `log2(x)`, `exp(x)` for perceptual/gain scaling

### FFI Integration
- Airwindows modules must be wrapped in FFI-safe C++ using `extern "C"` interface
- FFI wrappers go in `cpp/*.cpp`
- Use `build.rs` for FFI compilation

### GUI Development
- Built with `vizia` via `vizia-plug` for modern, performant GUI
- Follow vizia architecture patterns: Entity-Component-System (ECS) with reactive state management
- Use CSS-like styling with performant rendering via Skia graphics library
- Module color coding:
  - **EQ**: blue-gray background, cyan accents
  - **Compressor**: slate or black, orange knobs
  - **Pultec**: brass tones, gold highlights
  - **Dynamic EQ**: steel blue, green accents
  - **Console/Tape**: charcoal or oxide red tones
- Keep GUI interactions performant and audio-thread safe
- See [ADR-0009](docs/adr/0009-gui-framework-and-color-coding.md) for the GUI framework decision and full color-coding table, and [ADR-0007](docs/adr/0007-multi-fx-rack-native-drag-drop.md) for the rack/drag-drop design

**Key vizia Resources:**
- vizia-plug GitHub: https://github.com/vizia/vizia-plug
- vizia book: https://vizia.dev/
- vizia examples: https://github.com/vizia/vizia/tree/main/examples

## Build Commands

### Core Development
- **Development build**: `cargo build` (core modules)
- **Development build with GUI**: `cargo +nightly build --features "api5500,buttercomp2,pultec,transformer,gui"`
- **Release build**: `cargo build --release`
- **Run tests**: `cargo test`

### Plugin Bundle Creation (Production)
- **RECOMMENDED**: `just bundle` — uses the `FEATURES` var (api5500,buttercomp2,pultec,transformer,punch,haas,dynamic_eq,sheen,gui) and handles env vars automatically.
- **Manual full-feature command**:
  ```cmd
  set LLVM_HOME=C:\Program Files\LLVM
  set LIBCLANG_PATH=C:\Program Files\LLVM\bin
  cargo +nightly run --package xtask -- bundle bus_channel_strip --release --features "api5500,buttercomp2,pultec,transformer,punch,haas,dynamic_eq,sheen,gui"
  ```
- **Core modules only (no GUI, fast iteration)**: `just bundle-core` — same feature list minus `gui`

### Code Quality
- **Format code**: `cargo +nightly fmt` or `pre-commit run rustfmt-nightly --all-files`  
- **Lint**: `cargo clippy --all-targets --all-features`
- **Install pre-commit hooks**: `pre-commit install`

### Important Build Notes
- ✅ Build xtask first: `cargo +nightly build --package xtask`
- ✅ Use minimal environment variables to avoid Skia build conflicts
- ❌ Complex preflight script (`bin\preflight_build.bat`) causes Skia compilation issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fsecada01/bus_channel_strip](https://github.com/fsecada01/bus_channel_strip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
