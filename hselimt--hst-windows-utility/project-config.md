---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

HST WINDOWS UTILITY is a Windows optimization tool (registry tweaks, service management, task scheduler debloat, system cleanup, app/bloatware removal) shipped as two independent, feature-equivalent products:

- **`GUI/`** — Electron desktop app wrapping a local ASP.NET Core 8 backend + React 18 frontend.
- **`CLI/`** — A single self-contained Windows batch script (`HST WINDOWS UTILITY - SCRIPT VERSION v1.8.0.cmd`) implementing the same optimizations without any GUI dependency.

Both require Administrator privileges — nearly every operation is a registry write, service change, or scheduled task edit that silently fails without elevation.

## Architecture (GUI)

Three-process model at runtime:
1. **Electron shell** (`GUI/main.js`) — spawns the published .NET backend as a child process, polls `http://localhost:5200` until it's up, then loads that URL into a frameless `BrowserWindow`. On quit it kills the backend child process. Also handles: single-instance enforcement (`requestSingleInstanceLock`, quitting if another instance already holds it, focusing the existing window on a `'second-instance'` event); a native error dialog (`dialog.showErrorBox`) if the backend fails to start within `waitForServer`'s retry window, instead of quitting silently; external links (`setWindowOpenHandler` + a `will-navigate` guard) routed to the OS default browser via `shell.openExternal` rather than navigating inside the app window; and `app.disableHardwareAcceleration()`, called before `app.whenReady()` to avoid a GPU-compositor paint failure that otherwise renders a blank window.
2. **ASP.NET Core backend** (`GUI/Program.cs`, `GUI/Controllers/`) — serves the React build as static files AND exposes the REST API on port 5200. There is only one controller, `SystemController` (`GUI/Controllers/Controller.cs`), which fronts every feature as a POST/GET endpoint under `api/system/*`.
3. **React frontend** (`GUI/View/src/App.js`) — a single large component file (App.js, no router) that calls the backend via plain `fetch("http://localhost:5200/api/system/...")`. There is no build-time API client/SDK — endpoints are called by string literal.

Backend feature modules live under `GUI/Controllers/<Area>/` and are registered as scoped services in `Program.cs`:
- `Registry/RegistryOptimizer.cs` — registry tweaks (grouped into private methods like `OptimizePriorityAndPower`, `DisableGameFeatures`, `OptimizeLatency`, etc., all invoked from `OptimizeRegistryAsync`)
- `Services/SetServices.cs`, `Services/DisableUpdates.cs` — Windows service enable/disable
- `System/TaskSchedulerOptimizer.cs`, `System/SetPowerPlan.cs`, `System/CleanUp.cs`, `System/RestorePointCreator.cs`, `System/SystemInfoDto.cs`
- `Debloat/Debloater.cs` — app/package removal (MS apps, Xbox, Store, Edge, OneDrive, startup apps)
- `Helpers/` — shared infrastructure (see below)

Shared helpers (`GUI/Controllers/Helpers/`), used consistently across every feature module:
- `ConfigLoader.cs` — loads JSON config from the app base directory into typed dictionaries; used instead of hardcoding lists of services/apps/tasks to modify.
- `ProcessRunner.cs` — runs external commands elevated (`Verb = "runas"`) with a timeout, wrapping failures as exceptions.
- `Logger.cs` — writes timestamped entries to `%TEMP%\HST-WINDOWS-UTILITY.log`; log is reset (`InitializeLog`) on every app start. Every controller/service catches exceptions and calls `Logger.Error(operation, ex)`.
- `Paths.cs` — centralizes `Environment.SpecialFolder`-based paths (AppData, ProgramData, Desktop, etc.) so nothing hardcodes user-specific paths.
- `FileManager.cs` — file/dir cleanup helpers.

Data-driven config (JSON files at `GUI/` root, copied to the output dir on build via `.csproj` `<None Update>` entries):
- `ServicesConfig.json` — services grouped by category (`recommended`, `bluetooth`, `hyperv`, `xbox`), each with a `service` (short name), `name`, and `defaultStartup` (used for revert).
- `AppsConfig.json` — app/package lists (`startupApps`, `msApps`, `xboxApps`, `storeApps`).
- `ScheduledTasksConfig.json` — scheduled tasks to disable.
- `GUIDConfig.json` — power plan GUIDs used when reverting to Windows defaults.
- `wwwroot/Powerplan/HST.pow` — the custom power plan imported/activated by `SetPowerPlan.cs`; also kept as `GUI/HST.pow` and synced into `wwwroot/Powerplan/` at build/publish time (see `Build-GUI.bat` step 1, and the `CopyPowerPlanFile` target in the `.csproj`).

Controller endpoint pattern (`SystemController`): every action-taking endpoint follows the same shape — try/catch around the operation, `Logger.Error` on failure, and a JSON body of `{ status, message, success }` (200 on success, 400 for missing/invalid options, 500 on operation failure). Endpoints that accept a feature subset take an `Options` DTO (e.g. `ServiceOptions`, `DebloatOptions`, `CleanUpOptions`, `RevertOptions`) with boolean flags per category — follow this pattern for any new endpoint rather than inventing a new response shape.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hselimt/HST-WINDOWS-UTILITY](https://github.com/hselimt/HST-WINDOWS-UTILITY) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
