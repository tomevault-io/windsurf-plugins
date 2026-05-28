---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Claude's goal

The goal for Claude is to help develop PicaSim across Windows, Linux, Android, macOS and iOS.

The original project has been ported from Marmalade SDK to SDL2 + OpenGL. Windows, Linux, Android, macOS and iOS ports are functional.

The original project is owned by me, so I have all rights to do this.

The intention is to not change any existing behaviour significantly.

## Building

Always use CMake presets:

**Windows:**
- Debug: `cmake --build --preset windows-x64-debug`
- Release: `cmake --build --preset windows-x64-release`

**macOS (arm64):**
- Configure: `VCPKG_ROOT=/Users/roberto/vcpkg cmake --preset macos-arm64`
- Debug: `VCPKG_ROOT=/Users/roberto/vcpkg cmake --build --preset macos-arm64-debug`
- Release: `VCPKG_ROOT=/Users/roberto/vcpkg cmake --build --preset macos-arm64-release`
- Run: `cd data && ../build/macos-arm64/Debug/PicaSim` (must run from `data/` directory)

**iOS:**
- Configure: `cmake --preset ios-device`
- Debug: `cmake --build --preset ios-device-debug`
- Deploy via Xcode: `open build/ios-device/PicaSim.xcodeproj`
- Archive for TestFlight: `./ios_archive.sh` (do NOT use Xcode's Product→Archive — dSYMs won't be included, see iOS notes below)

**Linux (x64):**
- Configure: `cmake --preset linux-x64`
- Debug: `cmake --build --preset linux-x64-debug`
- Release: `cmake --build --preset linux-x64-release`
- Run: `cd data && ../build/linux-x64/Debug/PicaSim` (must run from `data/` directory)
- AppImage: `./linux_create_appimage.sh build/linux-x64 dist`

**Linux (arm64):**
- Configure: `cmake --preset linux-arm64`
- Debug: `cmake --build --preset linux-arm64-debug`
- Release: `cmake --build --preset linux-arm64-release`
- Run: `cd data && ../build/linux-arm64/Debug/PicaSim` (must run from `data/` directory)
- AppImage: `./linux_create_appimage.sh build/linux-arm64 dist`

**Android:**
- `cd android && gradlew.bat assembleDebug`

Always build in debug when building automatically, if just asked to "build". Requests to build in release will always be explicit.

## Git Commits (instruction for Claude/AI)

Never volunteer to commit changes. Only create commits when explicitly requested by the user.

## Project Overview

PicaSim is a cross-platform R/C flight simulator built with C++. It simulates radio-controlled aircraft with realistic physics, multiple aircraft types (40+), and various environments. The project targets Windows, Linux, macOS, Android, and iOS.

**Current stack**: SDL2 (window/input), OpenGL/GLES2 (rendering), OpenAL-Soft (audio), GLM (math), Bullet Physics (physics), Dear ImGui (UI).

**Note**: The project was migrated from Marmalade SDK (no longer commercially available) to SDL2 + OpenGL. Most dependencies are built from git submodules in `third_party/`; vcpkg provides only glad and OpenXR on Windows desktop. Linux builds do not use vcpkg.

## Architecture

### Core Classes

**PicaSim** (`source/PicaSim/PicaSim.h`) - Singleton orchestrator managing the game loop, camera, aeroplanes, and UI overlays. Handles mode transitions (ground/aeroplane/chase/walk) and game status (flying/paused).

**Aeroplane** (`source/PicaSim/Aeroplane.h`) - Container class that intentionally separates graphics and physics:
- `AeroplaneGraphics` - 3D rendering (can exist without physics for networked/animated planes)
- `AeroplanePhysics` - Flight dynamics simulation (can exist without graphics for AI/headless)
- This separation enables AI pilots, network multiplayer, and non-graphical simulations

**Controller hierarchy**:
- `Controller` - Abstract base for all input
- `HumanController` - Player input from touch/joystick
- `AIController*` variants - AI pilots (Glider, Powered, Tug)

**Environment** - Wind simulation, thermals, and terrain interaction

**Challenge** - Race/duration/limbo game modes with gates and scoring

### Framework Layer

`source/Framework/` contains reusable game engine components:
- `Graphics`, `RenderManager` - Rendering pipeline
- `Camera` - Multiple camera modes
- `AudioManager` - 3D positional audio via OpenAL-Soft
- `EntityManager` - Entity lifecycle management
- `ParticleEngine` - Particle effects
- `ShaderManager` - GPU shader handling

`source/Platform/` contains platform abstraction:
- `S3ECompat.h` - Marmalade API compatibility shims (SDL2 implementations)
- `Input.cpp/h` - Unified input handling (keyboard, mouse, touch, gamepad)
- `imgui_impl_sdl2.cpp/h` - ImGui SDL2 backend
- `GLCompat.h` - Apple platform OpenGL header shim (pre-included via CMake `-include` on our targets only)
- `AndroidAssets.cpp/h` - APK asset extraction to internal storage
- `VRManager.cpp/h`, `OpenXRRuntime.cpp/h` - VR support via OpenXR (desktop only)
- `FontRenderer.cpp/h` - Bitmap font rendering for in-game overlay text

### Physics

Uses Bullet Physics (`source/bullet-2.81/`) for rigid body dynamics. Custom aerodynamics code in `AeroplanePhysics.cpp` calculates lift, drag, and control surface effects using aerofoil definitions.

### Data Organization

All data lives in `data/` directory. The application runs with `data/` as the working directory.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rowlhouse/PicaSim](https://github.com/Rowlhouse/PicaSim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
