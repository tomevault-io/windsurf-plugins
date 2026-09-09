---
trigger: always_on
description: LLGL (Low Level Graphics Library) is a thin C++11 abstraction layer over modern and legacy
---

# AGENTS.md — LLGL

## Project context

LLGL (Low Level Graphics Library) is a thin C++11 abstraction layer over modern and legacy
rendering APIs (Direct3D 11/12, Vulkan, OpenGL, OpenGLES 3, WebGL, Metal) across desktop and
mobile platforms. C99, C#, and Go wrappers are generated on top of the C++ API.

## Primary goals when editing

When making changes, prefer solutions that are:
1. Correct across supported backends and platforms
2. Consistent with existing LLGL API and naming conventions
3. Easy to validate via examples and tests
4. Minimal and localized unless a broader refactor is clearly needed

## General coding rules

- Preserve the existing public API unless the task explicitly requires an API change.
- Avoid unnecessary stylistic churn in unrelated files.
- Keep platform-specific code isolated where possible.
- Prefer explicit, readable code over clever or condensed code.
- Maintain compatibility with the project’s existing C++ style and header organization.
- Do not introduce dependencies unless they are clearly needed and justified.

## C++ language standard policy

- The C++ codebase should be treated as **strictly C++11 compliant by default**.
- You may use compiler extensions or newer C++ features (C++14, C++17, and later)
  **only when they are explicitly guarded by preprocessor conditions** so they are excluded
  on compilers that do not support them.
- Do not introduce unguarded syntax or library usage that would break C++11 compilation.
- If a feature depends on a newer language version or compiler-specific extension,
  isolate it clearly and document the requirement near the use site.
- Prefer C++11-compatible alternatives unless the newer feature is clearly necessary and
  safely conditional.

## Build and configuration model

CMake is the source of truth; the platform scripts are convenience wrappers around it. The
top-level configuration uses C++11 for normal desktop/mobile builds, but C++17 for UWP and
Emscripten. `CMAKE_CXX_STANDARD_REQUIRED` is enabled, so do not infer the language standard from
the compiler selected by the IDE.

Most optional components are OFF unless the platform or a convenience script enables them:

- `LLGL_BUILD_TESTS` and `LLGL_BUILD_EXAMPLES` are OFF in a bare CMake configure.
- `LLGL_BUILD_STATIC_LIB` is OFF. The default is separate shared libraries: the frontend
  `LLGL` library plus one module per enabled renderer. Runtime loading uses names such as
  `LLGL_Vulkan`; static builds link the enabled modules into the application and have different
  link-order requirements (see `docu/README.md`).
- Renderer defaults are platform-dependent: Null is ON, desktop Windows enables D3D11/D3D12 and
  OpenGL, desktop Apple enables Metal and OpenGL, Linux enables OpenGL, mobile enables OpenGLES3,
  and Emscripten enables WebGL. Vulkan and OpenXR are opt-in.
- `LLGL_BUILD_WRAPPER_CSHARP` and `LLGL_BUILD_WRAPPER_GO` require
  `LLGL_BUILD_WRAPPER_C99`. `LLGL_BUILD_XR_OPENXR` currently requires a compatible Vulkan,
  D3D11, or D3D12 renderer and an OpenXR loader/header setup.

For a clean local configure, initialize submodules first, then use an out-of-source build:

```text
git submodule update --init --recursive
cmake -S . -B build_local -A x64 -DLLGL_BUILD_EXAMPLES=ON -DLLGL_BUILD_TESTS=ON
cmake --build build_local --config RelWithDebInfo
```

Use a separate build directory for materially different configurations (static/shared,
Debug/Release, renderer options, or architecture). CMake caches options, and rebuilding only an
executable does not reconfigure or rebuild a renderer whose compile-time option changed.

The checked-in platform entry points encode extra assumptions: `BuildWin64.bat` enables examples,
tests, C99, OpenGL, D3D11, and D3D12 and may obtain `GaussianLib`; Android requires
`ANDROID_NDK_ROOT` and API level 21 or later; Linux needs X11 and Xrandr development libraries;
Wasm uses Emscripten; Apple builds use Xcode. Check the corresponding `Build*.sh/.bat/.command`
script before inventing a new invocation.

## Dependencies and generated files

The repository has two git submodules: `external/SPIRV-Headers` and `external/GaussianLib`.
SPIRV-Headers is required when Vulkan SPIR-V reflection is enabled. GaussianLib is used by some
builds/examples and the Windows convenience script can fetch it into the build area. Keep these
requirements optional where the existing CMake option makes them optional; do not include generated
or downloaded dependency trees in a focused source change.

The wrapper files are generated from public headers and C99 declarations. Run
`scripts/GenerateWrappers.bat` from the `scripts/` directory (with Python available) after changing
an input declaration; it regenerates `include/LLGL-C/LLGLWrapper.h`, `wrapper/CSharp/LLGLWrapper.cs`,
and `wrapper/Go/LLGLWrapper.go`. Never hand-edit those generated outputs. Public struct layout is
part of the wrapper ABI: update the generator inputs and check the C99 assertions when adding,
removing, or reordering fields.

A high priority goal of LLGL is to keep external dependencies to an absolute minimum.

- Avoid adding new third-party dependencies unless there is a strong, explicit justification.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LukasBanana/LLGL](https://github.com/LukasBanana/LLGL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
