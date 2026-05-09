---
trigger: always_on
description: Chroma is a Rust terminal audio visualizer. It renders GPU-generated WGSL shader output as colored ASCII and can react to system audio. The source code is the authority for behavior; treat docs and examples as helpful context that may lag behind implementation.
---

# Chroma Agent and Contributor Guide

Chroma is a Rust terminal audio visualizer. It renders GPU-generated WGSL shader output as colored ASCII and can react to system audio. The source code is the authority for behavior; treat docs and examples as helpful context that may lag behind implementation.

`CONTRIBUTING.md` and `CLAUDE.md` are symlinks to this file. Keep this file as the single source of truth for contributor and agent guidance.

## First Places To Check

- `src/main.rs` and `src/app/` for application startup, the render loop, input handling, config reload, status bar behavior, and audio-reactive updates
- `src/params/` for user-facing parameters, enum mappings, defaults, clamping, serialization, and CLI/config conversion behavior
- `src/shader_common/` and `src/shader_patterns/` for WGSL uniforms, effects, pattern dispatch, and shader pattern implementations
- `src/audio/` for PulseAudio/PipeWire and CPAL capture, device selection, buffering, and FFT analysis
- `src/ascii/` and `src/render/` for RGBA-to-ASCII conversion and terminal frame construction
- `notes/` and `README.md` for user-facing docs that should be updated when behavior changes

## Current Facts

- The default target frame rate is `60` FPS (`DEFAULT_FPS` in `src/constants.rs`) and can be overridden with `--fps`.
- Audio support is built in. Linux uses PulseAudio/PipeWire monitor capture through libpulse when available, then falls back to CPAL.
- Linux runtime/package dependencies include Vulkan loader support, ALSA, and PulseAudio libraries.
- Stream mode is selected with `--stream WIDTHxHEIGHT`; it disables terminal setup, status bar, and interactive input, and emits full frames to stdout.
- Built-in presets are numbered `0..24` and live in `src/presets/`.
- `CLAUDE.md` and `CONTRIBUTING.md` should remain symlinks to `AGENTS.md`.

## Runtime Output Rule

Do not add `println!`, `eprintln!`, `dbg!`, or direct writes to stdout/stderr in runtime paths that can execute while the visualizer is rendering. Stdout is reserved for rendered frames and explicit list/stream commands. Extra output can corrupt the terminal UI and reduce FPS.

Use file-backed logging instead:

- `debug_logln!(debug_log, ...)` when an app-owned `DebugLog` is available
- `append_debug_line(component, message)` from library or background-thread code that cannot access `DebugLog`
- `CHROMA_TRACE_FRAMES=1` only when intentionally enabling per-frame debug logs guarded by `frame_logging_enabled()`

User-facing terminal output is acceptable only for commands that exit before rendering starts, such as `--list-audio-devices`, `--list-patterns`, `--list-color-modes`, and `--list-palettes`.

## Validation Commands

Prefer the Nix dev shell because it supplies the native audio libraries and the nightly rustfmt required by this repo's rustfmt options:

```bash
nix --extra-experimental-features 'nix-command flakes' flake check
nix --extra-experimental-features 'nix-command flakes' develop -c cargo fmt --all -- --check
nix --extra-experimental-features 'nix-command flakes' develop -c cargo test
nix --extra-experimental-features 'nix-command flakes' develop -c cargo clippy --all-targets -- -D warnings
nix --extra-experimental-features 'nix-command flakes' develop -c actionlint -color
```

Useful focused checks:

```bash
nix --extra-experimental-features 'nix-command flakes' develop -c cargo test --test shader_pattern_position_test
nix --extra-experimental-features 'nix-command flakes' develop -c cargo test --test render_test test_shader_produces_non_zero_output -- --ignored --exact
```

Running `cargo test` outside the dev shell may fail to link Linux PulseAudio libraries even when the Rust code is correct.

## Implementation Guidelines

- Keep changes scoped to the behavior being fixed. Avoid unrelated refactors, formatting churn, or doc rewrites unless they directly support the change.
- Prefer existing local patterns over new abstractions. This project values direct, simple code in hot paths.
- Keep every source code file at or below 700 lines of code. Removing empty lines or useful comments does not count as satisfying this rule; split files into idiomatic modules for organization before they grow past that limit.
- Preserve terminal rendering performance. Avoid per-cell allocations, excessive string formatting, blocking I/O, or unguarded logging inside frame rendering.
- Keep parameter changes synchronized across Rust params, CLI/config handling, `ShaderUniforms`, WGSL `Uniforms`, docs, and tests.
- Keep pattern enum ordering synchronized with WGSL dispatch IDs. If `PatternType` changes, inspect `src/shader_common/main.wgsl` and tests that assert numeric IDs.
- Register new patterns and randomly selectable visual/effect modes in the randomization pools in `src/params/randomizer.rs`, so `--random` and the `R` key can discover them. Add or update tests that fail when a new enum variant is missing from randomization.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuri-xyz/chroma](https://github.com/yuri-xyz/chroma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
