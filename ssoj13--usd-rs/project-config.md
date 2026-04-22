---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**usd-rs — Pure Rust OpenUSD Port**

A ground-up pure Rust port of Pixar's OpenUSD (Universal Scene Description) — the industry standard for 3D scene interchange in VFX, animation, and real-time graphics. Not bindings — a complete rewrite preserving the same architecture, APIs, composition semantics, and file format compatibility (USDA/USDC/USDZ). Includes a full egui-based viewer (usdview equivalent) with wgpu rendering backend.

**Core Value:** Pixel-perfect behavioral parity with C++ OpenUSD — same composition results, same file compatibility, same rendering correctness, same interactive performance. The Rust port must produce identical results for any USD file that the C++ reference handles.

### Constraints

- **Tech stack:** Pure Rust, no C/C++ dependencies, nightly toolchain, MSRV 1.85
- **Platform:** Windows 11 primary, cross-platform CI (Linux, macOS)
- **Parity standard:** Every change must be traceable to a specific C++ file:function in the reference
- **No logic changes:** Optimizations allowed only when they preserve identical behavior
- **Build:** Must compile with 0 errors, 0 warnings after every change
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Language & Runtime
- **Language:** Rust (Edition 2024, MSRV 1.85, Nightly toolchain)
- **Platform:** Windows 11 primary, cross-platform CI (Ubuntu, macOS, Windows)
- **Build:** Cargo workspace (72 crates)
- **Total LOC:** ~1.1M+ Rust across 2480+ files
## Core Dependencies
### Graphics & GPU
- `wgpu` 27 — GPU abstraction (dx12 + vulkan backends)
- `egui` 0.33 + `eframe` — immediate-mode UI
- `egui_dock` — dockable tabbed panels
- `egui-wgpu` 0.33.3 (vendored patch) — wgpu integration for egui
- `bytemuck` 1 — zero-copy GPU data casting
- `pollster` 0.4 — block on async for wgpu init
### Math & Geometry
- `glam` 0.31 — SIMD math (Vec2/3/4, Mat3/4, Quat)
- `half` 2.7 — IEEE 754 half-precision floats
- `wide` 0.7 — SIMD wide types
### Serialization & Data
- `serde` 1 + `serde_json` 1 — JSON serialization
- `ron` 0.8 — Rusty Object Notation (persistence)
- `quick-xml` 0.39 — XML parsing (MaterialX, plugInfo)
- `indexmap` 2.13 — ordered hash maps
- `smallvec` 1 — small-buffer-optimized vectors
### Error Handling
- Custom error enums per crate (no thiserror in USD core crates)
- `thiserror` 2 in viewer + ext crates (mtlx-rs, opensubdiv-rs)
- `anyhow` 1 in viewer (usd-view) only
### Logging & Tracing
- `tracing` 0.1 + `tracing-subscriber` 0.3
- `log` 0.4 (compatibility layer)
- `tracing-log` 0.2 (bridge)
### Concurrency
- `rayon` 1.11 — data parallelism
- `regex` 1.12 — pattern matching
- `notify` 7 — file system watcher (hot-reload)
### External Libraries (ext/)
- `opensubdiv-rs` — Pure Rust OpenSubdiv 3.7.0 port (subdivision surfaces)
- `mtlx-rs` — Pure Rust MaterialX port (material definitions, WGSL gen via naga)
- `osl-rs` — Pure Rust OSL port (Open Shading Language)
- `draco-rs` — Pure Rust Draco mesh compression
- `gltf-rs` — glTF 2.0 loader
- `pxr-lz4` — LZ4 decompression for USDC crate files
### VFX Ecosystem (from vfx-rs git dep)
- `vfx-core`, `vfx-math`, `vfx-lut`, `vfx-transfer`, `vfx-primaries`
- `vfx-icc`, `vfx-io`, `vfx-ocio`, `vfx-exr`, `vfx-compute`
- `murmur3` — hash function
- `vcv-rs` — (private, ssh)
### Image I/O
- `image` 0.25 — PNG/JPEG screenshot export
- `vfx-exr` — EXR image I/O
### Python Bindings
- `pyo3` 0.28 — Rust↔Python FFI (extension-module mode)
- `maturin` 1.x — build backend for Python wheels from Rust cdylib
- Python package: `pxr` — drop-in replacement for Pixar's `pxr` (`import pxr as pxr`)
### Miscellaneous
- `rfd` 0.17 — native file dialogs
- `dirs` 6 — platform config directories
- `glob` 0.3 — file pattern matching
- `naga` 24 — shader transpilation (GLSL→WGSL in mtlx-rs)
## Build Configuration
- `Cargo.lock` committed (binary project)
- Custom `build.rs` — minimal (cfg for sanitizer support)
- Vendored `egui-wgpu` patch at `vendor/egui-wgpu-0.33.3`
- vcpkg at `$VCPKG_ROOT` (c:/vcpkg) for native deps
- Feature flags: `wgpu` (default), `mtlx-rs`, `nightly`, `dev_build`, `jemalloc`
- Release profile: `lto = false`, `codegen-units = 1`
- `bootstrap.py` — unified build script: `b` (build all), `b p` (build Python wheel via maturin), `t` (test), `t p` (Python tests via pytest), `ch` (clippy+fmt)
- Python bindings: `maturin build -m crates/usd-pyo3/Cargo.toml --release` produces `pxr` wheel; `pyproject.toml` in `crates/usd-pyo3/`
## CI/CD
- GitHub Actions (`.github/workflows/`)
- Matrix: ubuntu-latest, macos-latest, windows-latest
- Toolchain: nightly
- Steps: checkout, rust-toolchain, build & test
- Submodules + LFS enabled
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Code Style
- **Edition:** 2024
- **Formatting:** `cargo fmt` (standard rustfmt)
- **Lints:** Per-crate `[lints.clippy]` sections — typically `all = "warn"`, `pedantic = "warn"`
- **Unsafe:** `unsafe_code = "warn"` (not deny — FFI and GPU code requires unsafe)
## Naming
- **RFC 430** naming: `UpperCamelCase` types, `snake_case` functions, `UPPER_SNAKE` constants
- **Concise domain names:** `get_tr()` not `get_translation()`, `calc_bbox()` not `calculate_bounding_box()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ssoj13) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
