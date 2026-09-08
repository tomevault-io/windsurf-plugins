---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Overview

FlashIE is a standalone Windows application that hosts an embedded IE WebBrowser control with a locally-loaded Adobe Flash Player ActiveX control (Flash.ocx). It uses a pre-patched Flash.ocx (no time bomb or region restrictions) and applies process-local, Flash-scoped compatibility exceptions to run content without registering Flash system-wide. Flash activation is forced without user clicks, with bounded retries for hidden iframes. Top-level HTTP(S) and local `.swf` navigations are converted in-process into full-window Flash documents while preserving the original navigation URL.

## Build

CMake project targeting MSVC with C++17 and static CRT (`/MT` for Release, `/MTd` for Debug).

```bash
# Generate build files (from repo root)
cmake -S . -B build-x86 -A Win32    # 32-bit
cmake -S . -B build-x64 -A x64      # 64-bit

# Windows 7 target (selects the OCX for Windows 7 and earlier)
cmake -S . -B build-win7-x86 -A Win32 -DFLASHIE_WINDOWS_TARGET=WIN7
cmake -S . -B build-win7-x64 -A x64 -DFLASHIE_WINDOWS_TARGET=WIN7

# Build (replace Release with Debug when needed)
cmake --build build-x86 --config Release
cmake --build build-x64 --config Release
cmake --build build-win7-x86 --config Release
cmake --build build-win7-x64 --config Release
```

Output binary: `<build-directory>/<Config>/FlashIE.exe`. Post-build steps automatically copy the selected architecture- and OS-matched `Flash.ocx` to the output directory.

## Architecture

### Source Files

Application sources and headers, plus three submodule dependencies:

- **`Detours/`** — [Microsoft Detours](https://github.com/microsoft/Detours.git) submodule. A library for intercepting Win32 API function calls. Built as a static library (`detours`) in CMake and linked into flashie. Used for all API-level inline hooks (COM, registry, file, host identity, WLDP, TypeLib).
- **`JScriptCC/`** — [JScriptCC](https://github.com/Mzying2001/JScriptCC.git) submodule. A C++ library for JScript Conditional Compilation preprocessing (`@cc_on`, `@if`, `@set`, `@end`). Built as a static library (`jscriptcc`) and linked into flashie. Used by the `ParseScriptText` hook to expand CC blocks before script execution.
- **`swc-es5-c-api/`** — [swc-es5-c-api](https://github.com/Mzying2001/swc-es5-c-api.git) submodule. A Rust static library exposing a C ABI for transpiling classic JavaScript to ES5. CMake builds its architecture-matched Win7 MSVC target through Cargo and links it as `swc_es5`. The `ParseScriptText` hook invokes it after JScriptCC for both classic JScript and JScript9 engines; expression-mode calls bypass SWC because its input contract is a complete script.
- **`source/flash_loader.h/.cpp`** — Process-wide `FlashLoader` namespace. The core hooking engine is exposed through `Activate`, `InstallHooks`, and `Deactivate`, and is organized into 8 sections:
  - **Section 1-2**: Includes, constants, function pointer typedefs for all hooked APIs.
  - **Section 3**: Process-wide state — one `LoaderState` owns the lifecycle phase, activation STA, module/factory/COM cookie, API Detours pointers, fake keys, Flash hooks, script hooks, windowless-rendering fallback mode, and activation timers. Independent SRW locks protect lifecycle transitions and the factory, fake-key, Flash-hook, and script-hook domains.
  - **Section 4**: API hook target resolution — `ResolveTarget` resolves API exports with module fallbacks, while `ResetApiHookPointers` clears the `LoaderState::api` table after installation failure or successful detachment. `InstallHooks()` consumes these persistent pointer slots for its atomic Detours transaction.
  - **Section 5**: Fake registry key system — an SRW-lock-protected table tracks real `HKEY` handles opened read-only as sentinels for fake Flash entries, plus real `FEATURE_BROWSER_EMULATION` handles whose executable value is overlaid in memory. Query buffer sizes follow Win32 registry contracts, and every tracked handle is closed on normal close or shutdown.
  - **Section 6**: COM wrapper classes:
    - `LoggingClassFactory` — wraps Flash's `IClassFactory` and hooks new Flash objects on `CreateInstance`: a Detours hook on Flash `QueryInterface`, plus direct vtable hooks on SetClientSite, QuickActivate, and the `IViewObject` drawing interfaces.
    - `FlashSafetyTearoff` — `IObjectSafety` tearoff with an independent interlocked lifetime that delegates COM identity to Flash (required by MSHTML for scripting). Page-provided property-bag values such as `allowScriptAccess` are not overridden.
  - **Section 7a**: COM hooks — `CoGetClassObject`, `CoCreateInstance`, `CoGetClassObjectFromURL`, `CLSIDFromProgID`. Intercept Flash CLSID requests and redirect to our local factory.
  - **Section 7b**: Registry hooks — `RegOpenKeyExW`, `RegQueryValueExW`, `RegCloseKey`. Provide fake Flash CLSID registration (InprocServer32, TypeLib, ProgID), MIME type mapping, scoped ActiveX kill-bit bypass, and a `FEATURE_BROWSER_EMULATION` value. Flash CLSID and ProgID paths use precise, case-insensitive element matching; unrelated `Compatibility Flags` values are never rewritten.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mzying2001/flashie](https://github.com/Mzying2001/flashie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
