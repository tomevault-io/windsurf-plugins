---
trigger: always_on
description: EGL is a portable EGL implementation: a thin C API surface in `src/egl.c`
---

# AGENTS.md - EGL working contract

EGL is a portable EGL implementation: a thin C API surface in `src/egl.c`
forwarding to `_egl*` entry points, with the real implementation in
`src/egl_*.cpp`. Display/config/context/surface/sync/image object model with
deferred destruction and reference counting, over four native backends:

| Backend | Where | Notes |
|---|---|---|
| Windows WGL + Vulkan HDR | `src/egl_windows.cpp`, `src/egl_windows_vk.cpp` | fully implemented |
| Windows ANGLE | `src/egl_windows_angle.cpp` | OpenGL ES delegated to ANGLE's libEGL |
| X11 / GLX + optional Vulkan HDR | `src/egl_x11_glx.cpp`, `src/egl_linux_vk.cpp` | `LINUX_VK` option |
| Wayland (GLX via XWayland + Vulkan present) | `src/egl_wayland.cpp` | all presentation goes through Vulkan |
| Linux/Wayland system GLES | `src/egl_linux_gles.cpp` | delegated to the system libEGL |

**Stability mandate.** This implements a Khronos-specified API and is a drop-in
for other EGL implementations, so conformance is the contract.

- Never edit the upstream API headers. `include/` holds **only** the
  Khronos-generated/owned headers (`EGL/egl.h`, `EGL/eglext.h`,
  `EGL/eglplatform.h`, `KHR/khrplatform.h`) and `src/wglext.h` is likewise
  upstream-generated. If one needs a change, that is a discussion, not an edit.
- Every first-party header lives in `src/`. This includes `src/eglctxinternals.h`;
  keeping first-party headers out of `include/` is deliberate so the public
  include directory reads as exactly the standard EGL API surface.
- Match the EGL 1.5 error contract: the documented `EGL_BAD_*` code must be set
  on **every** failure path. Returning `EGL_FALSE` / `EGL_NO_*` without setting
  an error leaves `eglGetError()` reporting `EGL_SUCCESS` and is a defect.
- Behaviour must be identical across backends. A platform-divergent outcome
  (something accepted on Windows and rejected on GLX, or vice versa) is a bug
  even when each half is locally defensible.
- Never reformat. Formatting is authored, not derived; tooling runs with
  `FormatStyle: none`. `src/egl.c` and the platform files have distinct,
  deliberate styles.

## Build (standalone)

```
cmake -S . -B build/ninja -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build/ninja
```

Requires the Vulkan SDK on Windows (`find_package(Vulkan REQUIRED)`); ANGLE
comes from the vcpkg manifest (`vcpkg.json`). Options: `EGL_WIN_ENABLE_ANGLE`,
`EGL_WAYLAND_ENABLE_GLES`, `EGL_LINUX_ENABLE_GLES`, `LINUX_VK`. Examples build
via `add_subdirectory(examples)`.

The build policy in `cmake/` is registered **only when EGL is the top-level
project**, so being vendored never disturbs a host build.

## Finding and eliminating bugs

All tooling is optional at build time - a clean checkout with none of it
installed builds exactly as before.

| Layer | Command | Gate |
|---|---|---|
| Compiler warnings | part of every build | `ENABLE_WERROR=ON` makes them fatal |
| cppcheck | `cmake --build build/ninja --target cppcheck` | exits non-zero on findings |
| cppcheck (exhaustive) | `... --target cppcheck-strict` | opt-in |
| clang-tidy | `python tools/check_tidy.py --build-dir build/ninja` | `WarningsAsErrors` in `.clang-tidy` |
| Sanitizers | `-DENABLE_SANITIZER=address,undefined` then run the examples | runtime faults |

`ENABLE_WERROR` is **OFF** by default so a clean checkout keeps building; turn
it on once the baseline is clean. `ENABLE_SANITIZER` (`address`, `undefined`,
`address,undefined`, `thread`) is the highest-yield lane here because the
object graphs are hand-rolled linked lists with reference counts.

**Suggested bug-elimination cycle.** Build with
`ENABLE_SANITIZER=address,undefined`, run the examples (especially the
`display_p3*` set and anything exercising `eglMakeCurrent`/`eglReleaseThread`),
run `cppcheck`, run `check_tidy.py`. Fix in order of confidence: memory
corruption and use-after-free first, then conformance/error-contract defects,
then platform divergences. Re-run the full set after each change. When anything
broken is found at any point, drop back to bugs immediately.

Suppression policy: `cppcheck.supp` stays short and every entry carries a
justification. Prefer inline `// cppcheck-suppress <id>` for one-off findings.
Never suppress to make a gate go green.

## Layout

| Path | Contents |
|---|---|
| `src/egl.c` | the public `egl*` C entry points, forwarding to `_egl*` |
| `src/egl_api.cpp`, `egl_globals.cpp`, `egl_display.cpp`, `egl_config.cpp`, `egl_context.cpp`, `egl_surface.cpp`, `egl_sync.cpp`, `egl_image.cpp` | the portable object model |
| `src/egl_internal.h`, `egl_common.h` | internal contracts shared by core and backends |
| `src/eglctxinternals.h` | first-party handle struct exposed to consumers (`eglGetPlatformDependentHandles`) |
| `src/egl_*.cpp`, `egl_*_vk.h` | the native backends listed above |
| `src/wglext.h` | upstream WGL extension declarations |
| `include/` | **upstream Khronos headers only** |
| `examples/` | one sample app per feature, incl. the Display-P3 colour-space trio |
| `cmake/` | build policy: `warnings.cmake`, `cppcheck.cmake` |
| `tools/` | `check_tidy.py` |

## Defect classes this code is prone to

- **Missing error codes.** Every failure return needs its `EGL_BAD_*`. This is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [McNopper/EGL](https://github.com/McNopper/EGL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
