---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

PoseStudio is an early-stage, open-source Qt6/C++ desktop application for 3D character creation, posing, and scene building. The codebase is currently a small skeleton: a main window shell, an asset manager side panel backed by SQLite, a Preferences dialog, a menu bar, and the foundation of a Vulkan-backed 3D viewport (`src/viewport/` — device/swapchain/renderer scaffolding that draws a Blender-style infinite floor grid through a real camera and can import + render lit Wavefront `.OBJ` meshes; see the 3D Viewport section below). The rigging, animation, and final-frame rendering systems described in README.md are still aspirational/roadmap — they don't exist in code yet.

## Build

Requires Qt 6 (developed against 6.11.1, MSVC 2022 64-bit), CMake 3.21+, and the **LunarG Vulkan SDK** (https://vulkan.lunarg.com — supplies the Vulkan headers, loader, validation layers, and the `glslc` shader compiler). `find_package(Vulkan REQUIRED)` aborts configuration with a clear message if the SDK is absent, so the viewport can't be built without it.

```
cmake -B build -DCMAKE_PREFIX_PATH="C:/Qt/6.11.1/msvc2022_64"
cmake --build build
```

If `CMAKE_PREFIX_PATH` isn't passed, `CMakeLists.txt` falls back to the hardcoded path `C:/Qt/6.11.1/msvc2022_64` — never commit a change to that fallback to point at your personal Qt install.

GLM (math), AMD's Vulkan Memory Allocator (VMA), and tinyobjloader (the OBJ importer's parser) are pulled in header-only via CMake `FetchContent` at pinned tags (see the "3D VIEWPORT DEPENDENCIES" block) — a fresh clone needs only Qt + the Vulkan SDK, nothing vendored. tinyobjloader, like VMA, is a single-header lib whose implementation is compiled in exactly one TU (`scene/objloader.cpp` via `#define TINYOBJLOADER_IMPLEMENTATION`). They're *populated* (downloaded) but their own CMake isn't run; we consume their headers directly via `target_include_directories`. GLM's Vulkan-convention flags (`GLM_FORCE_RADIANS`, `GLM_FORCE_DEPTH_ZERO_TO_ONE`) are set once project-wide as `target_compile_definitions` — never per-file, or translation units disagree on matrix layout (a silent-corruption bug).

The build links `Qt6::Widgets`, `Qt6::Gui`, `Qt6::Sql`, and `Vulkan::Vulkan` (the loader; `QVulkanInstance`/`QWindow` live in Qt6::Gui). No Qt Network/3D/Quick modules. `CMAKE_AUTOMOC`/`AUTOUIC`/`AUTORCC` are all on, so new `Q_OBJECT` classes, `.ui` files, and resources just need to be added to the `add_executable` source list / `resources.qrc` — no manual moc/rcc invocation.

Two post-build mirror steps copy on-disk content next to the executable on every build (both true mirrors — `rm -rf` then `copy_directory`, not overlays, so renamed/removed source files don't linger): (1) `resources/Maquettes/` (the built-in asset library, real on-disk content — not embedded via `resources.qrc`, since the Asset Manager browses libraries through `QDir`, not Qt's resource filesystem); and (2) the compiled SPIR-V shaders. Shaders are GLSL under `src/viewport/shaders/`, compiled to `*.spv` by `glslc` via a per-shader `add_custom_command` (gated through the `PoseStudioShaders` custom target that `PoseStudio` depends on), then mirrored to `<applicationDirPath>/shaders` where `VulkanRenderer` loads them at runtime. Add a new shader by dropping the `.vert`/`.frag` in that folder and appending its name to the `SHADER_SOURCES` list in `CMakeLists.txt`.

**Cross-platform posture**: PoseStudio is meant to be fully cross-platform (Windows, macOS, Linux). The stack is portable by construction — Qt6, C++17, CMake, SQLite, Vulkan — and the codebase carries no `#ifdef` platform branches; platform differences are absorbed by Qt (window/surface creation via `QVulkanInstance`/`QWindow`) and CMake rather than hand-written per-OS code. Accommodations already in place: writable data lives in `QStandardPaths::AppDataLocation` (an installed app's own directory is read-only on macOS/Linux — see the Database section); the Windows icon resource (`resources/PoseStudio.rc`) is added to the target only `if(WIN32)`; and because macOS runs Vulkan through **MoltenVK** (a Vulkan-on-Metal translation layer), `VulkanContext` enables the spec-mandated `VK_KHR_portability_subset` device extension whenever a device advertises it (a no-op on Windows/Linux). **Caveat:** only the Windows/MSVC build has been exercised so far — the macOS/Linux paths are correct by construction but unbuilt/unrun, and the one known runtime unknown on macOS is whether Qt's `QVulkanInstance` auto-adds the *instance*-level portability-enumeration extension (verify on real Apple hardware before claiming macOS support).

There is no test suite, linter, or CI configuration in this repo yet.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PoseStudio/PoseStudio](https://github.com/PoseStudio/PoseStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
