---
trigger: always_on
description: Source port of Unreal Tournament 1999 (v1.40) to **original Xbox hardware** using **VS2005 + XDK 5849**. Static lib build (no DLLs). Steve is the project manager, compiler, and hardware tester. Codex is sole programmer.
---

# AGENTS.md — UT99 Xbox OG Source Port

## Project Overview

Source port of Unreal Tournament 1999 (v1.40) to **original Xbox hardware** using **VS2005 + XDK 5849**. Static lib build (no DLLs). Steve is the project manager, compiler, and hardware tester. Codex is sole programmer.

## Repository Layout

```
C:\Programming\GitHub\UT99-Xbox-Releases\
  Core/                  ← UT99 source (upstream, minor Xbox patches)
  Engine/                ← UT99 source (upstream, minor Xbox patches)
  UT99-Xbox/             ← This project (all Xbox-specific code)
    XboxLaunch/          ← Entry point, platform objects (replaces Launch/)
      Inc/               ← FFileManagerXbox.h, FMallocXbox.h, FFeedbackContextXbox.h,
                            FOutputDeviceXboxError.h, FXboxLogger.h
      Src/               ← XboxLaunch.cpp, XboxEngine.cpp, XboxLaunchPrivate.h
    XboxDrv/             ← Viewport, controller input (replaces WinDrv/)
    XboxRender/          ← D3D8 render device (replaces D3DDrv/) — stub
    XboxAudio/           ← Audio — stub
    XboxNet/             ← Networking — stub
    XboxStubs/           ← CRT intrinsics (_ftol2_sse, _alloca_probe_16, __CxxFrameHandler3)
    Tools/
      patchxbe.py        ← PE→XBE converter (subsystem patch + imagebld + D3D8/XGRAPHC injection)
    UT99-Xbox.sln
```

## Build System

- **Toolchain:** Visual Studio 2005 + Xbox XDK 5849 (installed at `C:\XDK`, non-standard)
- **Architecture:** All modules compile as static libraries, linked into a single EXE by XboxLaunch
- **XBE creation:** Post-build step runs `patchxbe.py` which patches PE subsystem 1→14, runs `C:\XDK\xbox\bin\imagebld.exe`, then injects D3D8/XGRAPHC library version entries
- **Forced includes:** `CoreXboxCompat.h` for Core/Engine libs, `XboxLaunchPrivate.h` for XboxLaunch — these include `<xtl.h>` first, kill XDK macro collisions (`Top`, `MAKEFOURCC`), empty `DLL_EXPORT`/`CORE_API`/`ENGINE_API`, set `#pragma conform(forScope, off)` for VC6 compat
- **Key preprocessor defines:** `TARGET_XBOX=1`, `ASM=0`, `ASM3DNOW=0`, `ASMKNI=0`
- **Linker (Release):** `SubSystem="1"` (Console), no `EntryPointSymbol` — lets xapilib CRT startup call `main()`
- **Release libs:** `d3d8-xbox.lib xboxkrnl.lib xgraphics.lib xonline.lib xacteng.lib xnet.lib xapilib.lib s3tc.lib`
- **Manifest embedding:** Disabled or fails harmlessly (mt.exe error 31) — run `patchxbe.py` manually if needed

## Key Modified UT99 Source Files

- `Core/Src/UnXboxWin32.cpp` — Replaces UnVcWin32.cpp. Contains: `IMPLEMENT_CLASS(USystem)`, `USystem::StaticConstructor()`, `appPlatformInit()` (creates GSys + LoadConfig), `appBaseDir()`, `appSeconds()`, and all platform function stubs
- `Core/Src/CoreXbox.cpp` — `IMPLEMENT_PACKAGE(Core)` only
- `Engine/Src/EngineXbox.cpp` — `IMPLEMENT_PACKAGE(Engine)` + `GCache`/`GEngineMem` globals
- `Core/Inc/CoreXboxCompat.h` — Forced include for Core/Engine, kills XDK macro collisions
- `Core/Inc/UnVcWin32.h` — Needs `#ifndef` guard around `IMPLEMENT_PACKAGE_PLATFORM` (patched)

## Current State (as of March 29, 2026)

### What Works
- CRT init → `main()` entry point runs
- `FXboxLogger` writes to `D:\ut99.log` (opened before any Unreal code)
- `appInit()` completes: names init, config loaded from `D:\System\UnrealTournament.ini`, UObject subsystem initialized
- `InitEngine()` starts, reads `GameEngine=Engine.GameEngine` from config
- File manager (`FFileManagerXbox`) resolves paths using internal BaseDir tracking, `../` resolution works
- `D:\` drive mapping works on both CXBX-R and real Xbox (maps to XBE parent directory)

### Active Blocker — PackageNotFound for Engine.u

`StaticLoadClass("Engine.GameEngine")` calls `appFindPackageFile("Engine")` which searches `GSys->Paths`. **`GSys->Paths` may still be empty** despite adding `GSys->LoadConfig(1)` in `appPlatformInit()`.

The last build added `LoadConfig(1)` and diagnostic logging (`appPlatformInit: GSys->Paths.Num()=X`) but Steve hasn't tested it yet. **The diagnostic output needs to be checked in CXBX-R's console.**

The config loading chain: `LoadConfig` → `StaticConfigName()` returns `"System"` → `FConfigCacheIni::Find` appends `.ini` → `"System.ini"` → translated to `SystemIni` = `"UnrealTournament.ini"` → reads `[Core.System]` section → should populate `Paths` array.

If `Paths.Num()` is still 0 after `LoadConfig(1)`, the problem is in the property registration. Check that `USystem::StaticConstructor` registers the `"Path"` property (note: property name is `"Path"`, config key is `"Paths"` — this mismatch might be the bug).

### If Paths Load Successfully
The next issue will be that `appFindPackageFile` concatenates `appBaseDir()` + `Paths(i)` giving e.g. `D:\System\../System/*.u`. This raw path with embedded `../` must be resolved by the OS or by `FFileManagerXbox::ResolvePath`. On CXBX-R (Windows host), `FindFirstFileA` handles `../` natively. On real Xbox hardware, it may not — `ResolvePath` handles it.

## File Manager (FFileManagerXbox)

- Tracks `BaseDir` internally as a `TCHAR[1024]` member
- `SetDefaultDirectory(path)` stores path in BaseDir with trailing backslash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GTTeancum/UT99-Xbox-Releases](https://github.com/GTTeancum/UT99-Xbox-Releases) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
