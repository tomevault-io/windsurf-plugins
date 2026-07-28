---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UntitledImGuiFramework is a cross-platform desktop application framework built on [dear imgui](https://github.com/ocornut/imgui). It provides an immediate-mode GUI system with a module architecture, plugin support, and both C++ and C APIs. Target platforms: Windows, macOS, Linux (X11 + Wayland), WASM.

The framework is maintained by [MadLadSquad](https://madladsquad.com). Comprehensive end-user documentation lives in `docs/` (mirrored from the project wiki) — consult it before guessing at framework conventions.

## Repository Structure

```
/
├── Framework/                  # Core framework source (shared across all projects)
│   ├── Core/                   # Instance, Global, Allocator, FrameworkMain, types, interfaces, components
│   │   ├── Components/         # InlineComponent, WindowComponent, TitlebarComponent, Instance
│   │   ├── Interfaces/         # Input, Layouts, Plugin, Renderer, Window interfaces
│   │   ├── Platform/           # Platform-specific code (e.g. WASM)
│   │   └── Utilities.{cpp,hpp} # Utility interface (loadContext, etc.)
│   ├── Renderer/               # Built-in renderers + abstractions
│   │   ├── GenericRenderer/    # Abstract base for custom renderers + textures
│   │   ├── OpenGL/             # Built-in OpenGL backend
│   │   ├── Vulkan/             # Built-in Vulkan backend
│   │   ├── WebGPU/             # Built-in WebGPU backend (used as the default on WASM in place of Vulkan)
│   │   ├── RendererUtils.{cpp,hpp}  # Helpers for writing custom renderers
│   │   └── Texture.{cpp,hpp}   # User-facing Texture class
│   ├── Window/                 # Window backends
│   │   ├── GenericWindow/      # Abstract base for custom window/monitor backends
│   │   ├── GLFW/               # Built-in GLFW backend (default)
│   │   ├── Window.{cpp,hpp}    # Static Window interface
│   │   └── WindowUtils.{cpp,hpp}
│   ├── ImGui/                  # Dear ImGui integration
│   │   ├── ImGui.{cpp,hpp}     # GUIRenderer — internal imgui lifecycle
│   │   ├── ClientSideBar.*     # Client-side title bar (.mm for macOS)
│   │   └── UImGuiExtensions.*  # Glue between imgui and framework types (FString, etc.)
│   ├── Modules/                # Optional feature modules (controlled by uvproj.yaml)
│   │   ├── Manager/            # ModulesManager + Modules interface
│   │   ├── i18n/, Knobs/, Plotting/, Spinners/, TextUtils/, Theming/, Toggles/, Undo/
│   │   ├── OS/                 # UntitledOpen, UntitledExec, plus Unix-only DBus + XDG submodules
│   │   └── CLIParser/
│   ├── C/                      # C API wrappers (~97% coverage of public C++ APIs)
│   ├── ThirdParty/             # imgui, glfw, freetype, rapidyaml, parallel-hashmap, utfcpp, etc.
│   ├── cmake/                  # CMake helpers
│   └── Framework.hpp           # Umbrella header included by all generated sources
├── UVKBuildTool/               # Build & code generation tool (git submodule)
├── Projects/                   # Generated projects live here (e.g., UImGuiDemo)
├── Config/                     # Template config files copied into new projects
│   ├── Core/                   # Window.yaml, Renderer.yaml, Modules.yaml, Keybindings.yaml, *.ini layouts
│   ├── Translation/            # translation-base.yaml + locale-specific files for i18n
│   ├── WASM/                   # pre.js / post.js for emscripten builds
│   └── cmake/                  # Per-project CMake customisations (`<project>.cmake`)
├── Content/                    # Template content directory copied into new projects
├── docs/                       # End-user docs (mirrored from the GitHub wiki)
├── install.sh                  # First-time setup: builds UVKBuildTool, creates Projects/
├── create-project.sh           # Scaffold a new application project
├── create-plugin.sh            # Scaffold a new plugin project
├── export.sh                   # Package a project for production release
└── update.sh                   # Pull latest framework + rebuild UVKBuildTool
```

Per-project layout (under `Projects/<Name>/`): `Source/`, `Generated/`, `Exported/`, `Config/` and `Content/` symlinks back to the framework, `CMakeLists.txt`, `uvproj.yaml`.

## Development Workflow

### Initial Setup
```bash
./install.sh            # Builds UVKBuildTool and creates Projects/ directory
```

### Creating a New Project
```bash
./create-project.sh MyApp                     # Interactive scaffold + initial build
./create-project.sh MyApp --skip-compilation  # Scaffold only
./create-project.sh MyApp                     # Pass name as arg for headless/CI mode
```

This creates `Projects/MyApp/` with symlinks back to `Framework/` and `UVKBuildTool/`, a `uvproj.yaml` config, and runs UVKBuildTool to generate `CMakeLists.txt` and `Generated/main.cpp`.

### Building a Project
Projects are built with CMake inside `Projects/<Name>/build/`:
```bash
cd Projects/UImGuiDemo/build
cmake .. -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=RELEASE
make -j$(nproc)
```

For WASM:
```bash
emcmake cmake .. -DCMAKE_BUILD_TYPE=RELEASE
emmake make -j$(nproc)
emrun <project name>.html
```

### Regenerating Build Files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MadLadSquad/UntitledImGuiFramework](https://github.com/MadLadSquad/UntitledImGuiFramework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
