---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Restore packages
dotnet restore

# Build Debug (x64 only — the solution does not support x86/AnyCPU)
dotnet build InfoPanel.sln -c Debug

# Build Release
dotnet build InfoPanel.sln -c Release

# Publish for deployment (Windows x64)
dotnet publish InfoPanel/InfoPanel.csproj -c Release -r win-x64 --self-contained -p:PublishProfile=FolderProfile -p:Platform=x64

# Run the main application
dotnet run --project InfoPanel/InfoPanel.csproj

# Run plugin simulator for testing plugins
dotnet run --project InfoPanel.Plugins.Simulator/InfoPanel.Plugins.Simulator.csproj
```

> **Note:** InfoPanel.Extras is built automatically via custom MSBuild targets in `InfoPanel.csproj` (`BuildExtras` / `CopyExtrasToBuildOutput`). No separate build step is needed.

## Architecture Overview

InfoPanel is a WPF desktop application built on .NET 8.0 that displays hardware monitoring data on desktop overlays and USB LCD panels. The codebase follows MVVM architecture with a modular plugin system.

### Projects

| Project | Role |
|---|---|
| **InfoPanel** | Main WPF application (entry point, UI, services, drawing) |
| **InfoPanel.Plugins** | Plugin interface definitions (`IPlugin`, `IPluginSensor`, `BasePlugin`, `IPluginConfigurable`, `PluginActionAttribute`) |
| **InfoPanel.Plugins.Graphics** | Image provider interfaces (`IPluginImageProvider`, `IPluginImageWriter`) and MMF-backed double buffering |
| **InfoPanel.Plugins.Ipc** | IPC interfaces and DTOs for out-of-process communication (`IPluginHostService`, `IPluginClientCallback`) |
| **InfoPanel.Plugins.Host** | Out-of-process plugin host executable (StreamJsonRpc over named pipes) |
| **InfoPanel.Plugins.Loader** | Dynamic plugin loading with assembly isolation (`PluginLoadContext`, `PluginWrapper`) |
| **InfoPanel.Plugins.Simulator** | Test harness for plugin development |
| **InfoPanel.Extras** | Built-in plugins (system info, network, drives, weather) |

> **Local DLL references:** LibreHardwareMonitor DLLs are referenced from `../LibreHardwareMonitor/` relative to the repo root. FlyleafLib.dll is referenced from `../libs/` (custom build with headless renderer fix). Ensure that sibling repos/directories are present when building.

### Application Startup

Entry point is `Program.cs` (`[STAThread] Main`) → `App.xaml.cs` → `OnStartup()`:

1. Single-instance check (prompts to kill existing process)
2. `_host.Start()` — starts the Generic Host, which runs `ApplicationHostService` (an `IHostedService` that creates `MainWindow` and sets up navigation)
3. Flyleaf media engine initialization
4. `ConfigModel.Instance.Initialize()` — loads settings and profiles from disk
5. Default profile creation if first run
6. `HWHash.Launch()` — starts HWiNFO shared-memory reader on its own thread
7. `LibreMonitor.Instance.StartAsync()` — if enabled in settings
8. `PluginMonitor.Instance.StartAsync()`
9. `StartPanels()` — starts BeadaPanel, TuringPanel, and WebServer tasks based on settings

> **Note:** `Startup.cs` exists in the project but is entirely commented out — it is not part of the startup flow.

### Key Singletons

These singletons use the `Lazy<T>` / `Instance` pattern and are **not** in the DI container:

| Singleton | File | Role |
|---|---|---|
| `ConfigModel.Instance` | `Models/ConfigModel.cs` | Settings, profiles, persistence (XML). Thread-safe with locks. |
| `SharedModel.Instance` | `Models/SharedModel.cs` | Shared app state: selected profile, display items, sensor data. Inherits `ObservableObject`. |
| `DisplayWindowManager.Instance` | `DisplayWindowManager.cs` | Manages overlay windows on a dedicated STA thread. |
| `LibreMonitor.Instance` | `Monitors/LibreMonitor.cs` | LibreHardwareMonitor sensor polling. |
| `PluginMonitor.Instance` | `Monitors/PluginMonitor.cs` | Plugin lifecycle management. |
| `BeadaPanelTask.Instance` | `Services/BeadaPanelTask.cs` | BeadaPanel USB device communication. |
| `TuringPanelTask.Instance` | `Services/TuringPanelTask.cs` | TuringPanel USB/serial communication. |
| `WebServerTask.Instance` | `Services/WebServerTask.cs` | Built-in HTTP API and web interface. |
| `ProfilePreviewCoordinator.Instance` | `Services/ProfilePreviewCoordinator.cs` | Batched profile preview rendering. |
| `UndoManager.Instance` | `Services/UndoManager.cs` | Per-profile undo/redo via XML snapshots. |

### DI Container Services

Registered in `App.xaml.cs` via `Host.CreateDefaultBuilder().ConfigureServices(...)`:

- **Hosted:** `ApplicationHostService` (bridges host lifecycle → WPF)
- **Singletons:** `IThemeService`, `ITaskBarService`, `ISnackbarService`, `IContentDialogService`, `IPageService`, `INavigationService`
- **Scoped:** `MainWindow` (as `INavigationWindow`), all Pages and their ViewModels

### Threading Model

| Thread | Purpose |
|---|---|
| **Main UI thread** | WPF dispatcher, MainWindow, all settings pages |
| **DisplayWindowThread** | Dedicated STA thread created by `DisplayWindowManager` with its own `Dispatcher`. Hosts all `DisplayWindow` overlays. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habibrehmansg/infopanel](https://github.com/habibrehmansg/infopanel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
