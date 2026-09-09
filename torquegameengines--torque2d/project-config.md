---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Torque2D 4.0 ("Rocket Edition", Early Access) is a cross-platform 2D game engine. The C++ engine lives in `engine/source`; games are written in **TorqueScript** (`.cs` files) and structured as **modules**. The same engine binary runs the in-engine editors (Project Manager, Asset Manager, GUI Editor) and any game built on top of it.

**When writing or refactoring TorqueScript, follow the conventions in [`TORQUE_SCRIPT.md`](TORQUE_SCRIPT.md)** — the prescriptive style guide for script code (one class per file, `onAdd`/`onRemove` lifecycle, ownership/teardown chains, `class`/`superclass` inheritance).

Target platforms: Windows, macOS, Linux, iOS, Android, and Web (Emscripten).

## Building

**CMake is the single source of truth.** The engine is built from the root
`CMakeLists.txt`; you generate a project for your platform/toolchain and build it.
The executable is dropped at the **repository root** (`Torque2D.exe` /
`Torque2D_DEBUG.exe` on Windows).

- **Configure + build:** e.g. `cmake -S . -B build -G "Visual Studio 17 2022" -A x64` then `cmake --build build --config Debug` (also `Release`/`Shipping`). Single-config generators (Make/Ninja) use `-DCMAKE_BUILD_TYPE=` instead of `--config`. Convenience generator scripts live at the repo root (`generate-vs2022.bat`, `generate-vs2026.bat`, `generate-xcode.command`, `generate-make.sh`, `build-linux.sh`).
- **Per-platform recipes & status** (configure flags for macOS/iOS/Linux 32-bit/Android, runtime-verification state) are documented in `cmake/BUILD-PLATFORM-NOTES.md`.
- Engine sources are listed **explicitly** in `cmake/EngineSources.cmake` (cross-platform) and `cmake/PlatformSources.cmake` (per-platform back-ends: Windows, macOS, Linux, iOS, Android wired; Emscripten stubbed) — these are the authoritative file lists, **not** globs. (All six back-ends — Windows, macOS, Linux, iOS, Android, and Emscripten — are wired and runtime-verified.)
- Third-party libs (libogg, libvorbis, lpng, ljpeg, zlib) are built as static targets from `engine/lib/CMakeLists.txt`; GoogleTest is built via `add_subdirectory` and linked for the in-engine unit tests (desktop only).
- **Windows specifics that are load-bearing:** static non-debug runtime `/MT` for all configs (avoids `_DEBUG`, which would make tinyXML `#define DEBUG` and break Box2D), `/Zc:wchar_t-` (so `wchar_t` == the engine's `UTF16`), C++17, and `_HAS_STD_BYTE=0`.

The only remaining item under `engine/compilers/` is **not** a standalone build
system: `android-studio` is the Android app shell whose Gradle native step *invokes*
the root CMake via the NDK. The legacy hand-maintained projects — the VS solutions,
the macOS and iOS Xcode projects, the Linux Makefiles, and the Emscripten reference
recipe — have all been **retired** (the Web target is now CMake-runtime-verified);
CMake replaces them.

The built executable must run from the repo root because it loads `main.cs` and the script/asset trees (`editor/`, `library/`, `toybox/`, `tools/`) relative to the working directory.

## Running

The engine's entry point is the **`main.cs`** script next to the executable. On launch it calls `setCompanyAndProduct(...)` then `exec("./editor/main.cs")`, which starts the Project Manager UI. To boot directly into a game instead, scan and load a module (see the commented `ModuleDatabase.scanModules` / `ModuleDatabase.LoadExplicit` lines in `main.cs`).

The in-engine **console** (and the editor tabs: Asset Manager, Project Manager, GUI Editor) is opened with **Ctrl + Tilde (~)**.

## Tests

There are two suites, and they test different things.

### C++ unit tests (GoogleTest)

Vendored at `engine/source/testing/googleTest`. **Every test lives in `engine/source/testing/tests/`** (e.g. `guiTreeRowLayoutTests.cc`, `platformStringTests.cc`); each file is listed explicitly in `cmake/EngineSources.cmake`, so a new one needs a CMake edit and a re-configure to be compiled at all.

```
tests\run-unit.ps1                        all of them
tests\run-unit.ps1 GuiTreeRowLayoutTests.*  one suite (a GoogleTest filter)
```

- Under the hood that launches the engine with the alternate boot script `main.runAllUnitTests.cs`, which calls `runAllUnitTests()` and quits. You can also invoke `runAllUnitTests()` from the in-engine console.
- **`runAllUnitTests()` takes no arguments.** It hands `InitGoogleTest` an empty argv, so a subset is selected with the `GTEST_FILTER` environment variable — which is what `run-unit.ps1`'s parameter sets.
- **What a unit test can reach.** The engine boots far enough to give it `Con`, `Sim`, the string table, the resource manager and `GuiDefaultProfile`, so it can `new` and `registerObject()` a control, read and write fields, run script via `Con::evaluate`, and round-trip TAML. It has **no canvas and no GL context**, so it must never wake a control or measure text: a font registers a texture and `TextureManager::refresh` asserts — which in a debug build is a modal box, so the failure arrives as a *hang*. Note this rules out adding rows to a list box or tree, since that calls `updateSize()` → `getFont()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TorqueGameEngines/Torque2D](https://github.com/TorqueGameEngines/Torque2D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
