---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`Py4GW_Reforged` is a Windows-only, **32-bit** injected DLL for Guild Wars. It is a rework of the legacy `Py4GW` library, focused on a clean C++/Python interface and supporting a proxy library on top. Inside the game process it:

- spawns a runtime thread from `DllMain` (work is kept out of `DllMain` itself),
- hooks the Guild Wars Direct3D 9 render/reset pipeline via MinHook,
- hosts an embedded CPython interpreter through pybind11,
- renders a Dear ImGui overlay and runs user Python scripts.

The build output `Py4GW.dll` is written to the **repo root** (not a `bin/` dir) so it sits beside the `fonts/`, `scripts/`, and `offsets/` payload directories it loads at runtime.

## Build

32-bit is mandatory — CMake hard-fails on non-Windows or non-Win32 architecture.

```powershell
# Preset-based (writes to ./vs2022-win32)
cmake --preset vs2022-win32
cmake --build --preset vs2022-win32-relwithdebinfo   # or vs2022-win32-debug

# Or plain
cmake -S . -B build -A Win32
cmake --build build --config RelWithDebInfo
```

The DLL is emitted directly into the repo root, next to the `fonts/`, `scripts/`, and `offsets/` payload directories it loads at runtime (`Patterns::Initialize()` defaults to `<module_dir>/offsets`), so no copy step is needed. There are no automated tests; validation is "it builds + a forced startup/shutdown path works" (see migration docs).

Source files are globbed (`GLOB_RECURSE` with `CONFIGURE_DEPENDS`) from `src/**/*.cpp`, so **adding a new `.cpp` requires re-running CMake configure**. ImGui addon sources (`src/imgui/addons/`) are compiled into a separate `imgui_addons` static lib, not the main target. `base/error_handling.h` is force-included (`/FI`) into every translation unit of `Py4GW` and `imgui_addons`.

## Runtime lifecycle

- C API in `include/Py4GW.h`: `Py4GW_Initialize()`, `Py4GW_Shutdown()`, `Py4GW_RequestShutdown()`; runtime thread `py4gw::RuntimeThread`.
- `src/Py4GW.cpp` owns the coarse state machine (mutex + `g_running` / `g_shutdown_requested` booleans). Bootstrap order: logger → `Scanner` → `Patterns` → Python → GW layer → crash handler → register ImGui + render/reset callbacks.
- `src/GW/GuildWars.cpp` drives the GW subsystem layer: `GW::Initialize()` runs an ordered `kInitSteps` table of per-module `Initialize`/`Shutdown` pairs, then enables the `MemoryPatcher` hooks. Shutdown disables patches and tears down steps in reverse. **Add a migrated module by inserting it into `kInitSteps` here** (and update the array size). Each step stamps `CrashHandler::SetContext(...)` for crash attribution.
- Render callbacks `DrawLoop`/`OnReset` only run while `g_running`; the update loop calls the Python update path every ~10 ms.

Shutdown is a strict state transition, not best-effort: mark intent → stop new work → disable hooks/restore patches → drain in-flight detours → remove hooks → delete sync primitives. Never delete a critical section or null a trampoline while a detour may still enter it; a short sleep is not a correctness boundary. Crash capture is torn down last.

## GW module anatomy

Each Guild Wars subsystem lives under `include/GW/<module>/` + `src/GW/<module>/` in namespace `GW::<module>`. Observed file split:

- `<module>.h` — all module-owned declarations: lifecycle, public callable surface, structs/typedefs/aliases, globals, and **named ownership of every resolved symbol** (functions, offsets, anchors, pointers, callsites, patch sites).
- `<module>.cpp` — discovery/setup, hook install/trampolines, lifecycle orchestration, private helpers.
- `<module>_methods.cpp` — bodies of public callable accessors/operations (omit if the module is lifecycle-only).
- `<module>_patterns.cpp` — `Resolve*` functions that call `PY4GW::Patterns::Resolve("<ns>.<name>", &g_ptr)`.
- `<module>_bindings.cpp` — pybind11 module (see below).

Shared GW data (entities, context layouts, helpers on GW types — anything without its own lifecycle) goes in `GW/context/`, **not** inside whichever manager first needed it. Shared protocol/packet/opcode declarations go in `GW/common/`. Project-wide infrastructure with no GW specificity goes in `base/`.

## Pattern / scanner system

Addresses are **not** hardcoded — they are resolved at runtime from `offsets/<module>.json`. Each file declares a `namespace`, raw `patterns` (byte `pattern`+`mask`+`offset`+`section`, or assertion anchors via `assertion_file`/`assertion_message`), and step-based `resolvers` (ops like `scan`, `dereference`, `to_function_start`, `function_from_near_call`, `add`, `validate_section`). Code resolves a final symbol with:

```cpp
PY4GW::Patterns::Resolve("agent.change_target_func", &g_change_target_func);
```

Move byte patterns, masks, sections, offsets, assertion files/messages into the JSON — do not leave them as string literals in code. The `.cpp` may execute the scan, but the module **header** must declare which symbols the module owns. Validate resolved addresses before installing hooks.

## Python bindings


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apoguita/Py4GW_Reforged_Native](https://github.com/apoguita/Py4GW_Reforged_Native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
