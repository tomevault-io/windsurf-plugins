---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OASIS_OS is an embeddable operating system framework in Rust (edition 2024). It provides a skinnable shell with a scene-graph UI, command interpreter, virtual file system, browser engine (HTML/CSS/Gemini), plugin system, and remote terminal. It renders to any pixel buffer + input stream. Built from scratch in Rust starting early 2026, inspired by PSP homebrew shells like PSIX. Eighteen skins are implemented (14 external TOML skins, 18 built-in; external skins also have built-in equivalents).

Default virtual resolution is 480x272 (PSP native). Skins may override this (e.g. modern=800x600, xp=1024x768); the backend canvas/window scales to match.

## Build Commands

All CI commands run inside Docker containers. For local development you can run cargo directly (SDL3 is compiled from source automatically via the `build-from-source` feature), or use the Docker wrapper.

```bash
# Build (desktop)
cargo build --release -p oasis-app

# Build via Docker (matches CI exactly)
docker compose --profile ci run --rm rust-ci cargo build --workspace --release

# Run tests
cargo test --workspace

# Run a single test
cargo test --workspace -- test_name

# Run tests in a specific crate
cargo test -p oasis-core

# Format check
cargo fmt --all -- --check

# Apply formatting
cargo fmt --all

# Lint (CI treats warnings as errors)
cargo clippy --workspace -- -D warnings

# License/advisory audit
cargo deny check

# Build PSP backend (excluded from workspace, requires nightly + cargo-psp)
cd crates/oasis-backend-psp && RUST_PSP_BUILD_STD=1 cargo +nightly psp --release

# Build PSP overlay plugin PRX (excluded from workspace, kernel mode)
cd crates/oasis-plugin-psp && RUST_PSP_BUILD_STD=1 cargo +nightly psp --release

# Build UE5 FFI shared library
cargo build --release -p oasis-ffi

# Build WASM backend (requires wasm-pack)
./scripts/build-wasm.sh          # debug build
./scripts/build-wasm.sh --release # release (smaller + faster)
# Serve: python3 -m http.server 8080 → http://localhost:8080/www/

# Take screenshots
cargo run -p oasis-app --bin oasis-screenshot
```

## CI Pipeline Order

format check -> clippy -> nightly clippy (advisory) -> doc build -> markdown link check -> test -> release build -> screenshot regression -> cargo-deny -> benchmarks -> PSP EBOOT build -> PPSSPP headless test -> code coverage -> GitHub Pages deploy (WASM)

All steps run via `docker compose --profile ci run --rm rust-ci`.

**Memory analysis** (`memory-ci.yml`, separate non-blocking workflow):
- **ASAN** (AddressSanitizer) -- catches use-after-free, buffer overflow, leaks (~2x slowdown)
- **Valgrind massif** -- heap profiling with peak memory assertions for video decode

Both jobs use `continue-on-error: true` and run on pushes to main, PRs touching `crates/oasis-video/**` or `crates/oasis-core/**`, and `workflow_dispatch`.

**Nightly streaming** (`nightly-streaming.yml`, separate workflow):
- End-to-end streaming validation for TV Guide video playback

## Architecture

### Crate Dependency Graph

```
oasis-types     (foundation: Color, Button, InputEvent, backend traits, error types, geometry)
├── oasis-vfs        (virtual file system: MemoryVfs, RealVfs, GameAssetVfs)
├── oasis-platform   (platform service traits: Power, Time, USB, Network, OSK)
├── oasis-sdi        (scene display interface: named object registry, z-order)
├── oasis-net        (TCP networking, PSK auth, remote terminal, FTP)
├── oasis-audio      (audio manager, playlist, MP3 ID3 parsing)
├── oasis-ui         (32 widgets: Button, Card, TabBar, ListView, flex layout, etc.)
├── oasis-wm         (window manager: drag/resize, hit testing, decorations)
├── oasis-skin       (TOML skin engine, 18 skins, theme derivation)
├── oasis-terminal   (90+ commands across 17+ modules, shell features)
├── oasis-browser    (HTML/CSS/Gemini: DOM, CSS cascade+@media, layout engine, full 2D CSS transforms, Canvas 2D path API, SVG paths/groups, light compositor, z-index stacking contexts, nested scroll containers, form elements with select dropdown + label association, hover-triggered CSS transitions, soft hyphens, bidi text, JS DOM bindings)
├── oasis-js         (JavaScript engine: QuickJS-NG runtime, console API)
├── oasis-video      (MP4/H.264+AAC decode; StreamingBuffer sliding-window; features: h264, no-std-demux, video-decode)
├── oasis-vector     (vector graphics: scene graph, path ops, icons, frame-driven animations)
├── oasis-shader     (animated shader wallpapers: Shadertoy-style fragment shaders)
├── oasis-rasterize  (software rasterizer for CPU-side rendering)
├── oasis-i18n       (internationalization support)
├── oasis-test-backend (mock backend for testing)
├── oasis-app-core   (shared app framework: AppTrait, common utilities)
├── oasis-app-games  (Games app)
├── oasis-app-paint  (Paint app)
├── oasis-app-clock  (Clock app)
├── oasis-app-text-editor (Text Editor app)
├── oasis-app-calculator  (Calculator app)
├── oasis-app-media       (Music Player + Photo Viewer apps)
├── oasis-app-tv-guide    (TV Guide app)
├── oasis-app-radio       (Internet Radio app)
├── oasis-app-settings    (Settings app)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndrewAltimit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
