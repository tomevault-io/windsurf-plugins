---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Methane Kit is a cross-platform C++20 3D graphics abstraction library and application framework. It wraps DirectX 12, Vulkan and Metal behind one object-oriented API (RHI), with a single HLSL 6 shader codebase compiled to all backends at build time.

CMake 3.24+ is required; the `VS2026-*` presets additionally need **CMake 4.2+**, which is where the `Visual Studio 18 2026` generator they use was added. External dependencies are **not** git submodules — they are fetched by [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) during CMake configure into `Build/Output/ExternalsCache/` (override with `-DCPM_SOURCE_CACHE=<path>`). The first configure is slow because of this.

## Build & Test

Preferred path is CMake presets. Preset names follow `[VS2022|VS2026|Xcode|Make|Ninja]-[Win64|Win32|Win|Lin|Mac|iOS|tvOS]-[Sim]-[DX|VK|MTL]-[Default|Profile|Scan]`; the matching *build* preset replaces `Default` with `Debug` or `Release`. List them with `cmake --list-presets` and `cmake --list-presets build`.

```bash
cmake --preset VS2026-Win64-DX-Default
cmake --build --preset VS2026-Win64-DX-Debug
```

Build output goes to `Build/Output/<ConfigPresetName>/Build`, installs to `.../Install`.

Build a single target (much faster than the whole solution):

```bash
cmake --build --preset VS2026-Win64-VK-Debug --target MethaneHelloTriangle
```

Auxiliary scripts wrap the raw CMake invocation: `Build/Windows/Build.bat [--vs2022] [--win32] [--debug] [--vulkan] [--tracy] [--itt] [--logs] ...` and `Build/Unix/Build.sh [--debug] [--vulkan VULKAN_SDK_PATH] [--apple-platform ...] ...`.

### Tests

Tests use Catch2 and are registered with CTest via `catch_discover_tests` ([Tests/CMake/CatchDiscoverAndRunTests.cmake](Tests/CMake/CatchDiscoverAndRunTests.cmake)). Run all from the build directory:

```bash
ctest --test-dir Build/Output/VS2026-Win64-VK-Default/Build -C Debug --output-on-failure
```

Filter by test name with CTest:

```bash
ctest --test-dir Build/Output/VS2026-Win64-VK-Default/Build -C Debug -R "Render Context" --output-on-failure
```

Or invoke the Catch2 binary directly, selecting a single case by name or a group by tag (`--list-tests` shows both; names are prefixed `RHI `, tags look like `[device][rhi]`):

```bash
./Build/Output/VS2026-Win64-VK-Default/Build/Tests/Graphics/RHI/Debug/MethaneGraphicsRhiTest.exe "RHI Render Context Basic Functions"
```

```bash
./Build/Output/VS2026-Win64-VK-Default/Build/Tests/Graphics/RHI/Debug/MethaneGraphicsRhiTest.exe "[device]"
```

`METHANE_RUN_TESTS_DURING_BUILD` (initially `ON`, but `OFF` in all presets) runs CTest as a post-build step.

Tests never touch a real GPU: they link `MethaneGraphicsRhiNull`/`MethaneGraphicsRhiNullImpl`, the Null RHI backend that exists solely for unit testing.

### Key CMake options

`METHANE_GFX_VULKAN_ENABLED` (default `OFF`) switches Windows/macOS from their native API (DirectX 12 / Metal) to Vulkan; Linux is always Vulkan. Others worth knowing: `METHANE_CHECKS_ENABLED`, `METHANE_LOGGING_ENABLED` (off even in Debug presets — `META_LOG` is compiled out), `METHANE_RHI_PIMPL_INLINE_ENABLED`, `METHANE_TRACY_PROFILING_ENABLED`, `METHANE_ITT_INSTRUMENTATION_ENABLED`, `METHANE_GPU_INSTRUMENTATION_ENABLED`. Full table in [Build/README.md](Build/README.md#cmake-options).

## Architecture

### Module layers

Five layers of increasing abstraction, each a directory under [Modules/](Modules): `Common` (primitives, checks, instrumentation) → `Data` (types, events, ranges, animation, providers) → `Platform` (app window, input, app view) → `Graphics` (RHI, camera, mesh, primitives, app base) → `UserInterface` (typography, widgets, UI app). [Modules/Kit](Modules/Kit) is the umbrella target pulling everything together.

### Graphics RHI — the core abstraction

[Modules/Graphics/RHI](Modules/Graphics/RHI) is where most work happens. It is deliberately split, and the dependency direction matters:

```
Interface ──> Base ──> {DirectX | Vulkan | Metal | Null} ──> Impl
```

- **[Interface](Modules/Graphics/RHI/Interface)** — public virtual interfaces (`IDevice`, `IRenderContext`, `ICommandList`, …) and shared settings structs. No implementation.
- **[Base](Modules/Graphics/RHI/Base)** — API-agnostic logic shared by every backend (command list state tracking, resource lifetime retention, object registry). Backend classes derive from these.
- **[DirectX](Modules/Graphics/RHI/DirectX) / [Vulkan](Modules/Graphics/RHI/Vulkan) / [Metal](Modules/Graphics/RHI/Metal) / [Null](Modules/Graphics/RHI/Null)** — native implementations. Exactly one is compiled, chosen by CMake.
- **[Impl](Modules/Graphics/RHI/Impl)** — PIMPL value-type wrappers (`Rhi::Device`, `Rhi::RenderContext`, …) that applications actually use. These are what tutorials are written against, not the virtual interfaces.

A change to RHI behaviour usually means editing `Base` (if it is API-independent) plus each native backend. Because only one backend compiles per configuration, **a change to Vulkan code is not compile-checked by a DirectX build** — build the relevant preset.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MethanePowered/MethaneKit](https://github.com/MethanePowered/MethaneKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
