---
trigger: always_on
description: This file provides guidance to Claude Code when working with the Bloom Engine codebase.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the Bloom Engine codebase.

## Project Overview

Bloom is a native TypeScript game engine compiled by [Perry](../../perry/perry) (a TypeScript AOT compiler). It provides a simple, function-based API for 2D/3D games that compiles to Metal, DirectX 12, Vulkan, OpenGL, and WebGPU.

## Build Commands

```bash
# Native (macOS)
cd native/macos && cargo build --release

# Native (Windows — set INTERPROCEDURAL_OPTIMIZATION=OFF so the Jolt
# cmake build doesn't trip over LTO with the MSVC toolchain)
INTERPROCEDURAL_OPTIMIZATION=OFF cargo build --release --manifest-path native/windows/Cargo.toml

# Tests: unit + golden-image suite (run before any renderer PR)
cd native/shared && cargo test --release
BLOOM_UPDATE_GOLDEN=1 cargo test --release   # regenerate goldens — commit ONLY the ones your change intentionally moved (mean tol 2, outliers 1%)

# Web/WASM
cd native/web && cargo check --target wasm32-unknown-unknown
./native/web/build.sh [game.ts]              # Full web build pipeline

# Check shared code compiles for all targets
cd native/shared && cargo check                                          # native (default features)
cd native/shared && cargo check --target wasm32-unknown-unknown --no-default-features --features web  # WASM
```

After an engine-only rebuild, a consuming game does NOT relink by
itself: `perry compile` skips the link if the game's `main.ts` is
untouched — touch it first. After ANY change to `package.json`'s
native-function manifest, also delete the game's `.perry-cache/`.
Games should link with `perry compile … --debug-symbols` so `main.pdb`
lands next to the exe (see `docs/crash-triage-windows.md`).

## Architecture

```
src/                  TypeScript API (compiled by Perry)
  core/               Window, input, game loop, runGame()
  shapes/             2D shapes + collision
  textures/           Image loading, sprites
  text/               Font rendering
  audio/              Sound + music
  models/             3D models, skeletal animation
  math/               Vectors, matrices, easing
  scene/              Scene graph, frame callbacks, lighting
  physics/            Jolt-backed rigid + soft bodies, character, vehicles

native/               Rust implementations (one crate per platform)
  shared/             Cross-platform core
                      - renderer/: wgpu 29 renderer — deferred MRT
                        (hdr/material/velocity/albedo), TSR upscaling,
                        cascaded shadows, SSR, Lumen-class GI (screen
                        probes; HW ray-query / SDF-clipmap / Hi-Z tiers,
                        mesh cards, WSRC), material system with a
                        5-bind-group ABI (shaders/material_abi.wgsl),
                        transient texture pool, 2D draw layer.
                        WGSL lives as strings in renderer/shaders/*.rs
                      - profiler.rs: CPU+GPU pass profiler (enabling it
                        inserts a blocking per-frame GPU sync — never
                        benchmark with it on)
                      - audio/: control/render split over a lock-free
                        SPSC ring (mod/render/stream/decode)
                      - text_renderer.rs: fontdue text, rasterized at
                        physical resolution
                      - string_header.rs: Perry string ABI (read its
                        header comment before touching FFI strings)
                      - ffi_core/: define_core_ffi! macro — the shared
                        FFI surface each platform crate instantiates
                      - physics_jolt.rs: JoltPhysics wrapper (native only)
                      - jolt_sys.rs: C ABI bindings to bloom_jolt shim
                      - textures.rs, models.rs, scene.rs, etc.
  third_party/
    JoltPhysics/      Jolt 5.5.0 submodule (built via cmake crate)
    bloom_jolt/       C++ shim exposing Jolt behind a C ABI
                      - include/bloom_jolt.h, src/bloom_jolt.cpp
  macos/              Metal + AppKit + Core Audio
  ios/                Metal + UIKit + Core Audio
  tvos/               Metal + UIKit + GCController
  windows/            DirectX 12 + Win32 + WASAPI
  linux/              Vulkan/OpenGL + X11 + PulseAudio
  android/            Vulkan/OpenGL ES + NativeActivity + AAudio
  web/                WebGPU/WebGL + Canvas + Web Audio API (WASM via wasm-pack)
  visionos/           Metal + UIKit-style shell (wgpu; iOS/tvOS-family port)
  watchos/            SwiftUI Canvas (2D) + SceneKit (3D) — no wgpu/Jolt on
                      the watch; own .glb loader; built via Perry's
                      watchos-swift-app feature (see docs/watchos-target.md)
```

## FFI Pattern

Each platform implements ~470 `bloom_*` FFI functions declared in `package.json` under `perry.nativeLibrary.functions`. Native platforms use `#[no_mangle] extern "C"`, web uses `#[wasm_bindgen]`.

String parameters are `i64` on native (Perry StringHeader pointers) and NaN-boxed string IDs on web (converted by JS glue layer).

### Hard-won FFI rules (violating these produced real shipped bugs)

- **Every new native MUST be declared in `package.json`'s manifest.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bloom-Engine/engine](https://github.com/Bloom-Engine/engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
