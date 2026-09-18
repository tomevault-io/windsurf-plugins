---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Lean Launcher is a native Windows application launcher (Alt+Space, type, Enter), built as an independent fork of [Takeoff](https://github.com/akiraeng/takeoff-launcher). Pure C++17, Win32, Direct2D/DirectWrite - no Electron, no third-party UI frameworks. Design priorities: <500 KB binary, ~10 MB RAM, <1ms search, zero telemetry.

**Naming note:** the project is mid-rebrand from "Takeoff" to "Lean Launcher". CMake targets, the repo, and internal identifiers (`LeanLauncher.exe`, `kMutexName`, window title "Lean Launcher") use the new name; the README's `<h1>`, some comments, and `scripts/package.ps1` still say "Takeoff". Don't be surprised by the mismatch - prefer "Lean Launcher"/`LeanLauncher` in new code and treat leftover "Takeoff" references as not-yet-migrated rather than intentional.

## Build

```powershell
# CMake (primary)
cmake -S . -B cmake -A x64
cmake --build cmake --config Release
# Binary at cmake/Release/LeanLauncher.exe

# Or Visual Studio 2022: open LeanLauncher.sln, set Release/x64, Ctrl+Shift+B
# (outputs under msbuild/bin and msbuild/obj - see LeanLauncher.vcxproj)
```

Requires Windows 10 1809+ or Windows 11, VS2022 with the Desktop C++ workload, Windows SDK. Compiler flags: `/W4 /permissive- /utf-8`, C++17, `UNICODE _UNICODE NOMINMAX WIN32_LEAN_AND_MEAN`. Keep builds clean at `/W4` - CI enforces both the CMake build and the raw `msbuild LeanLauncher.sln` build.

## Test

```powershell
# Core regression tests (fuzzy matching, text navigation, hotkeys, recency, version compare)
ctest --test-dir cmake -C Release --output-on-failure
# or directly:
.\cmake\Release\LeanLauncherCoreTests.exe

# Run a single check: core_tests.cpp is one flat main() of sequential Check(condition, "label")
# assertions (see tests/core_tests.cpp) - there's no test-name filter, so isolate by
# temporarily commenting out other Check() calls, or grep the label to find the line.

# Interactive UI smoke test (window creation, acrylic, caret, selection, actions overlay)
py tests/ui_smoke.py cmake/Release/LeanLauncherUiTests.exe
```

`LeanLauncherUiTests` is a second executable built from the same sources (`add_launcher` in CMakeLists.txt is a function invoked twice) but compiled with `LEANLAUNCHER_UI_TEST` defined, giving it a separate window class/mutex and no global hotkey registration - it never attaches to or replaces a real running launcher instance. This is the mechanism to know about before adding anything that behaves differently under test vs. production; check `kUiTest` (from `LEANLAUNCHER_UI_TEST`) at the call site.

No linter is configured; `/W4 /permissive-` warnings-as-errors-in-spirit is the correctness bar.

## Architecture

Everything lives in `src/` as headers included by `main.cpp` (single translation unit); there is no `.cpp`/`.h` split for the app logic itself.

- **`main.cpp`** - `wWinMain` entry point, single-instance mutex handling (replaces a running instance via `kExitLauncherMessage` + process wait/terminate rather than allowing two instances), the app-scanning worker thread (`BuildAppIndex`, Start Menu + common install dirs via bounded recursive directory scan with skip-lists for noise dirs like `node_modules`/`temp`), and the `AppEntry`/`RankedResult` data model. Wraps `LauncherWindow` (from `launcher.h`).
- **`launcher.h`** - the `LauncherWindow` class (~3700 lines): owns the Win32 window, Direct2D render target, all UI drawing (`DrawSearch`, `DrawResults`, `DrawSettings`, `DrawActions`, `DrawHotkeyWarningModal`, etc.), input handling (`HandleKeyDown`, `HandleClick`, IME composition), settings UI state machine (row navigation, hotkey recording), and two background workers started from `Create()`: `IconWorkerMain` (icon extraction/caching) and `IndexWorkerMain` (file index rebuild triggering). This is the largest and most central file - read the relevant `Draw*`/`Handle*` method rather than the whole file when making UI changes.
- **`search.h`** - `Normalize`/`Condense` string prep and `MatchScore` fuzzy-matching (acronym, prefix, word-boundary, multi-token, condensed matching with weighted scoring - see `tests/core_tests.cpp` for the scoring contract, since it's easier to infer intent from the test expectations than the scoring code itself). Also `SearchInput` (text field state: selection, caret, IME) and Explorer PIDL helpers.
- **`file_index.h`** - `FileIndex` singleton: background-threaded, chunked (`IndexChunk`/`IndexSnapshot`, copy-on-write via `shared_ptr<const IndexChunk>`) file/folder index for file search, separate from the app index built in `main.cpp`.
- **`settings.h`** - `Settings` struct (hotkey bindings, startup/tray/update/search toggles) and hotkey serialization/naming helpers. Persisted via `LauncherWindow::LoadSettings`/`SaveSettings` in `launcher.h`.
- **`calculator.h`** - standalone expression lexer/parser (`Lexer`, `Parser`, `CalculationResult`) for inline math evaluation (`sqrt(144)`, `2^10`, etc.), no dependency on the rest of the app.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdkasper/lean-launcher](https://github.com/sdkasper/lean-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
