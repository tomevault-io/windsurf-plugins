---
trigger: always_on
description: This is the **ClamWin** antivirus GUI — a **native C++ Win32 application** with a Google Drive-inspired visual design, rewritten from the legacy Python/wxPython codebase. This is a standalone repository; the ClamAV engine and command-line tools are built separately from [clamav-win32](https://github.com/clamwin/clamav-win32).
---

# Copilot Instructions — ClamWin GUI

## Project Overview

This is the **ClamWin** antivirus GUI — a **native C++ Win32 application** with a Google Drive-inspired visual design, rewritten from the legacy Python/wxPython codebase. This is a standalone repository; the ClamAV engine and command-line tools are built separately from [clamav-win32](https://github.com/clamwin/clamav-win32).

The application does **not** link against libclamav. It spawns `clamscan.exe` and `freshclam.exe` as child processes and captures their output.

## Architecture

### Class Hierarchy

| Class | Role | Base |
|---|---|---|
| `CWApplication` | Singleton. Owns config, tray, scheduler, dashboard. Entry point via `run()`. | — |
| `CWWindow` | Base for top-level windows. Static `WndProc` → `GWLP_USERDATA` → virtual dispatch. | — |
| `CWDialog` | Base for modal dialogs created from in-memory `DLGTEMPLATE`. | — |
| `CWDashboard` | Main card-based dashboard. | `CWWindow` |
| `CWScanDialog` | Scan/update progress with RichEdit log. | `CWDialog` |
| `CWPrefsDialog` | Preferences (stub — needs full implementation). | `CWDialog` |
| `CWScheduleDialog` | Schedule settings (stub — needs full implementation). | `CWDialog` |
| `CWAboutDialog` | Version/credits display. | `CWDialog` |
| `CWLogViewDialog` | Log file viewer. | `CWDialog` |
| `CWProcess` | RAII wrapper for `CreateProcess` + pipe capture. | — |
| `CWConfig` | INI-based config via `GetPrivateProfileString`. | — |
| `CWTray` | System tray icon management. | — |
| `CWScheduler` | Timer-based job triggering. | — |
| `CWTheme` | Light/Dark palette with cached GDI brushes. | — |

### Key Files

- Entry point: `cw_main.cpp` → `CWApplication::run()`
- CMake: `CMakeLists.txt` (target name: `clamwin`)
- Shared defs: `cw_gui_shared.h` (resource IDs, message constants, shared GUI types)
- Platform defs: `cwdefs.h` (`_WIN32_WINNT 0x0410`)
- DPI helper: `cw_dpi.h` (`CW_Scale()`)
- Theme: `cw_theme.h`/`.cpp` (Google Drive palette, dark/light detection)

## Build System

### Toolchain
- **CMake 4.x** at `C:\Program Files\CMake\bin\cmake.exe`
- **MinGW-w64 GCC 15.x** at `C:\msys64\mingw64\bin\gcc.exe`
- **Make** via `mingw32-make`

### Build Commands

Always set PATH first:
```powershell
$env:PATH = "C:\msys64\mingw64\bin;C:\Program Files\CMake\bin;" + $env:PATH
```

Configure (first time or after CMakeLists.txt changes):
```powershell
cmake -S . -B build-gui -G "MinGW Makefiles" -DCMAKE_C_COMPILER=gcc -DCMAKE_MAKE_PROGRAM=mingw32-make
```

For shell extension builds, pass Unicode mode explicitly:
- `-DCLAMWIN_SHELLEXT_UNICODE=ON` for `build-gui`, `build-x64`, `build-x86-mingw-winxp`
- `-DCLAMWIN_SHELLEXT_UNICODE=OFF` for `build-x86-mingw-win98`

Build GUI:
```powershell
Push-Location .\build-gui; mingw32-make clamwin 2>&1 | Select-Object -Last 20; Pop-Location
```

Build tests:
```powershell
.\test-gui.ps1
```

Run real-tool smoke tests:
```powershell
.\test-gui.ps1 -RealTools
```

Run stricter gated FreshClam probes:
```powershell
.\test-gui.ps1 -RealTools -FreshclamUpdate -FreshclamNegative
```

### Stopping the running instance
The linker cannot overwrite `clamwin.exe` while it is running. Always kill it before rebuilding:
```powershell
Stop-Process -Name "clamwin" -ErrorAction SilentlyContinue; Start-Sleep -Milliseconds 400
```
Combine with a build in one line:
```powershell
Stop-Process -Name "clamwin" -ErrorAction SilentlyContinue; Start-Sleep -Milliseconds 400; Push-Location .\build-gui; mingw32-make clamwin 2>&1 | Select-Object -Last 10; Pop-Location
```

### Working Directory
All cmake/make commands run from the repository root.

## Coding Standards

### Naming
- **Classes**: `CWPascalCase` (e.g. `CWDashboard`, `CWScanDialog`)
- **Methods**: `camelCase` (e.g. `onCreate`, `onPaint`)
- **Virtual handlers**: `onVerb` (e.g. `onCommand`, `onDestroy`)
- **Members**: `m_camelCase` (e.g. `m_hwnd`, `m_config`)
- **Static/global**: `s_prefixed` (e.g. `s_instance`)
- **Constants/IDs**: `UPPER_CASE` with prefix (`IDC_`, `IDM_`, `CW_`)
- **Files**: `cw_snake_case.cpp/h`

### Style
- 4-space indentation, no tabs
- Allman brace style (opening brace on its own line)
- `#pragma once` for header guards
- Forward-declare instead of including where possible
- One class per `.cpp`/`.h` pair

### C++ Features — SAFE (compile-time, statically linked)
- Classes, virtual functions, inheritance, RAII
- `std::string`, `std::vector` (via static libstdc++)
- `enum class`, references, `const`
- Constructors/destructors for resource cleanup

### C++ Features — AVOID
- `std::thread`, `std::mutex` → use `CreateThread`, `CRITICAL_SECTION`
- `std::filesystem` → uses Win32 APIs not on Win98
- C++ exceptions → use return codes
- `std::unique_ptr` → manual RAII in destructors
- Unicode `W` APIs → ANSI `A` APIs only

### Win32 Constraints
- `_WIN32_WINNT 0x0410` (Windows 98 minimum)
- ANSI APIs only (no `W` suffix functions)
- `GWL_USERDATA` not `GWLP_USERDATA` on true Win9x
- `InitCommonControls()` for Win95/98 compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clamwin/clamwin](https://github.com/clamwin/clamwin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
