---
trigger: always_on
description: Draxul is a high-performance, cross-platform Neovim GUI frontend built with C++20. It also supports shell hosts (Bash, Zsh, PowerShell, WSL). It leverages native GPU rendering (Vulkan on Windows, Metal on macOS) for low-latency text and grid updates, with SDL3 for windowing/input.
---

# GEMINI.md - Draxul Project Context

## Project Overview

Draxul is a high-performance, cross-platform Neovim GUI frontend built with C++20. It also supports shell hosts (Bash, Zsh, PowerShell, WSL). It leverages native GPU rendering (Vulkan on Windows, Metal on macOS) for low-latency text and grid updates, with SDL3 for windowing/input.

- **Communication:** msgpack-RPC via `nvim --embed` over stdin/stdout pipes
- **Text Stack:** FreeType (loading), HarfBuzz (shaping), dynamic glyph atlas (shelf-packed, RGBA8)
- **Main Goal:** Provide a visually polished and responsive Neovim experience with robust Unicode/emoji/ligature support and native OS integration.

See `docs/features.md` for a complete list of implemented features, configuration options, CLI flags, and build infrastructure.

## Building and Running

### Prerequisites
- **Windows:** CMake 3.25+, Visual Studio 2022, Vulkan SDK (with `glslc`), `nvim` on PATH.
- **macOS:** CMake 3.25+, Xcode CLT, `nvim` on PATH.

### Build Commands

The project uses CMake Presets for configuration.

- **Windows (Debug):**
  ```powershell
  cmake --preset default
  cmake --build build --config Debug --parallel
  ```
- **Windows (Release):**
  ```powershell
  cmake --preset release
  cmake --build build --config Release --parallel
  ```
- **macOS (Debug):**
  ```bash
  cmake --preset mac-debug
  cmake --build build --parallel
  ```
- **macOS (Release):**
  ```bash
  cmake --preset mac-release
  cmake --build build --parallel
  ```
- **macOS (ASan):**
  ```bash
  cmake --preset mac-asan
  cmake --build build --target draxul-tests
  ctest --test-dir build -R draxul-tests
  ```

### Running the App
- **Windows:** `.\build\Release\draxul.exe` (or `.\build\Debug\draxul.exe`)
- **macOS:** `./build/draxul.app/Contents/MacOS/draxul` or `open ./build/draxul.app`
- **Flags:** `--console` (Windows only, opens log console), `--smoke-test` (brief startup check), `--host <type>` (nvim/bash/zsh/powershell/wsl/megacity), `--log-file <path>`, `--log-level <level>`

### Convenience Scripts
- `do run`: Configure, build, and run the application (supports `debug`/`release`, `--vs`/`--ninja`, `--reconfigure`).
- `t.bat` / `t.sh`: Build and run the test suite.

### Debugging / Logging

Use `--log-file` and `--log-level` CLI flags (reliable on all platforms; env vars don't propagate into macOS `.app` bundles).

```bash
./build/draxul.app/Contents/MacOS/draxul --host zsh --log-file /tmp/debug.log --log-level debug
```

- Log levels: `error`, `warn`, `info`, `debug`, `trace`
- Log categories: `App`, `Rpc`, `Nvim`, `Window`, `Font`, `Renderer`, `Input`, `Test`
- Macros: `DRAXUL_LOG_ERROR`, `DRAXUL_LOG_WARN`, `DRAXUL_LOG_INFO`, `DRAXUL_LOG_DEBUG`, `DRAXUL_LOG_TRACE`

## Testing

- **Framework:** Catch2 with CTest runner
- **Run Tests:** `t.bat` (Windows) or `./t.sh` (macOS), or `ctest --test-dir build --output-on-failure`
- **Smoke Test:** `py do.py smoke` (spawns Neovim, verifies flush, exits within timeout)
- **Render Tests:** TOML scenario files in `tests/render/`, reference BMP images in `tests/render/reference/`
- **Bless References:** `py do.py blessbasic`, `blesscmdline`, `blessunicode`, `blessligatures`, `blessall`
- **Replay Fixtures:** Use `tests/support/replay_fixture.h` for redraw-oriented tests without launching Neovim

## Architecture

### Library Structure

- `libs/draxul-types`: Shared POD types, events, logging (header-only)
- `libs/draxul-window`: `IWindow` abstraction and SDL3 implementation
- `libs/draxul-renderer`: Renderer hierarchy (`IBaseRenderer` -> `I3DRenderer` -> `IGridRenderer`), Vulkan and Metal backends
- `libs/draxul-font`: FreeType/HarfBuzz font loading, shaping, glyph atlas
- `libs/draxul-grid`: Cell-based grid model, dirty tracking, highlight table
- `libs/draxul-nvim`: Neovim process management, msgpack-RPC, redraw parsing, input translation
- `libs/draxul-host`: Host abstraction (`IHost` -> `I3DHost` -> `IGridHost` -> `GridHostBase`), HostManager, terminal emulation (VT parser, scrollback, selection, mouse protocols)
- `libs/draxul-app-support`: Config I/O, grid rendering pipeline, render test infrastructure
- `libs/draxul-ui`: ImGui-based diagnostics panel
- `modules/megacity/`: Optional megacity module (gated by `DRAXUL_ENABLE_MEGACITY`). Contains four internal libraries — `draxul-megacity`, `draxul-citydb`, `draxul-treesitter`, `draxul-geometry`. The terminal product has zero source-level dependency on this directory; megacity self-registers via `HostProviderRegistry` from `app/main.cpp` under `#ifdef DRAXUL_ENABLE_MEGACITY`.
- `app/`: Orchestration only

### Key Abstractions

- **Renderer hierarchy**: `IBaseRenderer` -> `I3DRenderer` -> `IGridRenderer`. `MetalRenderer` and `VkRenderer` implement `IGridRenderer`.
- **IRenderPass / IRenderContext**: Typed render pass abstraction. Subsystems register passes with `I3DRenderer::register_render_pass()`.
- **Host hierarchy**: `IHost` -> `I3DHost` -> `IGridHost` -> `GridHostBase`. Terminal/Neovim hosts inherit `GridHostBase`. `MegaCityHost` inherits `I3DHost` directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmaughan/Draxul](https://github.com/cmaughan/Draxul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
