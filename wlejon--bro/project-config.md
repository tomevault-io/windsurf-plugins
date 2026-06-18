---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

**Apps live in a sibling repo** — `../broworkshop/` holds the launcher and starter apps (`games/`, `tools/`, `demos/`, `ai/`). bro is the runtime only; no apps are bundled here. Run any app by passing its directory to `bro` or `bro-headless`.

**Naked `bro` opens the built-in project manager** at `system/projects/` (the no-args fallback in `src/main.cpp`). New projects are seeded from `system/skeletons/<name>/`. Registry persists at the OS user-data dir (`%APPDATA%/bro/projects.json` etc.). See [docs/projects.md](docs/projects.md). System-panel scanning (`src/engine/system_panels.cpp`) skips any `system/<dir>/` containing a `bro.json` so these self-contained apps aren't double-loaded as overlay panels.

**Windows** uses the Visual Studio multi-config generator — one build dir, pick the config at build time:
```bash
cmake -B build                                 # configure (do not use MinGW)
cmake --build build --config Debug
cmake --build build --config Release
./build/Debug/bro.exe ../broworkshop/demos/example
./build/Release/bro.exe ../broworkshop/demos/example          # bro-headless.exe lives alongside
```

**Linux / macOS** use Ninja (single-config) — `--config` is ignored, so use a separate build dir per config:
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug        # debug
cmake --build build
./build/bro ../broworkshop/demos/example

cmake -B build-release -DCMAKE_BUILD_TYPE=Release   # release
cmake --build build-release
./build-release/bro ../broworkshop/demos/example
```

For `scripts/package-release.sh` on Linux/macOS, pass `--build-dir build-release` so it picks up the Release binaries (the script's `--config Release` default is the Windows-style config selector and is a no-op for Ninja).

**Common headless invocations** (paths differ per platform as above):
```bash
# Interactive JS REPL (GPU — default)
bro-headless ../broworkshop/demos/example

# JS script file
bro-headless ../broworkshop/demos/example test.js

# Inline JS expression
bro-headless ../broworkshop/demos/example -e "document.querySelector('#btn').click()" -e "screenshot('out.png')"

# CPU-only, no GPU/WebGL
bro-headless --no-gpu ../broworkshop/demos/example
```

Submodules must be initialized: `git submodule update --init`

Skia is a pre-built dependency. On Linux, run `third_party/skia/build_skia_linux.sh`; on macOS, run `third_party/skia/build_skia_mac.sh` (uses CoreText, freetype/fontconfig off). On Windows, build Skia separately and place `skia.lib` in `third_party/skia/lib/{Debug,Release}/`.

On macOS, `tests/run_tests.sh` uses `mapfile` and needs bash 4+ (`brew install bash` → `/opt/homebrew/bin/bash tests/run_tests.sh`); the system `/bin/bash` is 3.2.

## Architecture

Bro is a lightweight app runtime: HTML/CSS/JS apps rendered with GPU acceleration. ~103K LOC of C++20.

**Stack:** QuickJS (JS engine) + qjsbind (C++/JS bindings) + brokit (web/system APIs) + htmlayout (HTML parsing + CSS + layout) + broaudio (audio engine) + bromesh (mesh generation/manipulation) + Jolt (physics) + Skia (raster rendering) + SDL3 (windowing + GPU display)

**Two executables, one Engine:**
- `bro` — windowed app runner (DisplayMode::Windowed)
- `bro-headless` — headless tool with JS scripting (DisplayMode::Headless)

Both share the same `Engine` class configured via `EngineConfig`. Headless defaults to GPU rendering via a hidden SDL window (same pipeline as windowed, including WebGL). Use `--no-gpu` to fall back to `RasterRenderer` (CPU Skia) for environments without a GPU. Headless exposes JS globals (`screenshot()`, `advanceTime()`, `flush()`, `sleep()`, `assert()`) for scripted testing.

### Module dependency graph

```
util  (logging, string helpers — standalone)
  ↑
platform  (SDL3 window, event loop)
  ↑
render  (abstract Renderer interface, SkiaRenderer, RasterRenderer)
  ↑
svg  (SVG rendering via Skia SVG module)
  ↑
layout  (htmlayout adapters, draw traversal, replaced elements)
  ↑
dom  (Document/Element/TextNode tree, events, style proxy)
  ↑
canvas  (Canvas 2D API)    webgl  (WebGL 2.0 context)    scene  (3D scene graph, meshes, terrain, sprites)    physics  (Jolt physics world)
  ↑
js  (QuickJS + qjsbind bindings: DOM, canvas, WebGL, audio, mesh, physics, scene)
  ↑
engine  (orchestrates all subsystems, main loop)
```

### Key design patterns

- **Single DOM:** HTML is parsed with gumbo into a `bro::dom` tree. CSS is resolved by `htmlayout::css::Cascade`, layout by `htmlayout::layout::layoutTree()`, and rendering by `DrawTraversal` which walks the tree and issues Skia draw calls.
- **GPU rendering:** `GPUContext` owns the `SDL_GPUDevice`, shader pipelines (color + texture), and manages the D3D12 render passes. `SkiaRenderer` rasterizes HTML/CSS to a Skia surface, uploads to a `SDL_GPUTexture` via transfer buffers, and composites as a fullscreen textured quad. Canvas 2D commands are batched into vertex buffers and drawn via the color pipeline.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wlejon/bro](https://github.com/wlejon/bro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
