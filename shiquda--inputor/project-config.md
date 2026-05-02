---
trigger: always_on
description: This file is for coding agents working in this repository.
---

# AGENTS.md - inputor agent guide
This file is for coding agents working in this repository.
Prefer the source tree over older docs when they disagree; `README.md` might be stale in a few places.

## Project snapshot
- Product: Windows-first desktop app for privacy-safe Chinese and English input statistics.
- Current UI stack: WinUI 3 on .NET 8, not Avalonia.
- Monitoring stack: FlaUI UIA3 over Windows UI Automation.
- Persistence: local JSON files under `%LocalAppData%\inputor`.
- Export: CSV files under `%Documents%\inputor-exports`.
- Privacy rule: never persist raw captured text; only counts, metadata, and derived debug fields.

## Repository layout
- `inputor.sln` - single-solution entry point.
- `src/inputor.WinUI/` - only active app project.
- `src/inputor.WinUI/Program.cs` - process entry point and CLI test harness.
- `src/inputor.WinUI/App.cs` - app bootstrap, service wiring, lifecycle, tray coordination.
- `src/inputor.WinUI/MainWindow.cs` - main dashboard shell with navigation and page switching.
- `src/inputor.WinUI/StatisticsPage.cs` - trend, heatmap, and distribution UI.
- `src/inputor.WinUI/DebugPage.cs` - debug capture UI and event inspection.
- `src/inputor.WinUI/SettingsPage.cs` - settings, export, and data-management UI.
- `src/inputor.WinUI/NotifyIconService.cs` - tray icon integration through Windows Forms.
- `src/inputor.WinUI/TrayMenuWindow.cs` - custom tray popup window.
- `src/inputor.WinUI/Services/` - business logic and OS integration.
- `src/inputor.WinUI/Models/` - POCO-style state and snapshot models.

## Architecture notes
- The project name is `inputor.WinUI`, but the assembly name stays `inputor.App` for compatibility.
- Namespaces are split intentionally: UI/root files use `Inputor.WinUI`, models use `Inputor.App.Models`, and services use `Inputor.App.Services`.
- There is no DI container. `App` constructs and owns the service graph directly.
- There is no ViewModel layer today. UI is built in C# code-behind, not XAML-heavy MVVM.
- `MainWindow` hosts a `NavigationView` with Overview, Statistics, Apps, Debug, and Settings surfaces.
- `StatsStore` is the shared state hub. UI and tray subscribe to its `Changed` event.
- `MonitoringService` runs on its own STA worker thread and polls the foreground control through UIA.
- Character counting flows through `CompositionAwareDeltaTracker`, paste detection, and bulk-load filtering before counts are recorded.
- `StartupDiagnostics` writes startup and crash clues to `%LocalAppData%\inputor\startup.log`.

## Build, run, and verification commands
Use these from the repository root.

### Restore
```bash
dotnet restore inputor.sln
```

### Build
```bash
dotnet build inputor.sln
```

### Clean build
```bash
dotnet clean inputor.sln
dotnet build inputor.sln
```

### Run the desktop app
```bash
dotnet run --project src/inputor.WinUI/inputor.WinUI.csproj
```

- If the repository provides a usable `just` startup recipe, prefer it first (for this repo, use `just dev`) and do not launch the exe directly, because Windows may open an installed build instead of the current worktree output.

## Test and lint reality
- There is no dedicated test project in this repository today.
- There is no lint command or StyleCop configuration in this repository today.
- There is no supported `dotnet test` target, so single-test execution is not available.
- If a task asks for "run a single test", state that no automated test suite exists yet and use the CLI probes below or a manual smoke test instead.

## Built-in CLI probes
- Count supported characters:
```bash
dotnet run --project src/inputor.WinUI/inputor.WinUI.csproj -- --count-sample "Hello世界"
```
- Simulate IME composition-aware deltas:
```bash
dotnet run --project src/inputor.WinUI/inputor.WinUI.csproj -- --simulate-sequence "你|你好|你好世|你好世界"
```
- Simulate paste detection:
```bash
dotnet run --project src/inputor.WinUI/inputor.WinUI.csproj -- --simulate-paste "Hello" "Hello World" "World"
```
- Simulate bulk-load filtering:
```bash
dotnet run --project src/inputor.WinUI/inputor.WinUI.csproj -- --simulate-bulk 12 "Hello world" "Edit" false
```

## Manual smoke test checklist
Run this before calling substantial app changes done.
1. `dotnet build inputor.sln` succeeds.
2. Launch the app and keep it open for at least 5 seconds.
3. Confirm the tray icon appears.
4. Confirm the main window opens and navigation works.
5. Type in a supported external app such as Notepad and verify counts change.
6. Open Settings and verify save/export actions still work.
7. Exit through the tray/menu path and confirm clean shutdown.

## Code style expectations

### File organization
- Keep one top-level class per file.
- Match file names to class names exactly.
- Use file-scoped namespaces.
- Put reusable business logic in `Services/` and data contracts in `Models/`.
- Keep WinUI page and window composition close to the owning UI file unless a helper is clearly reusable.

### Naming
- Classes, methods, properties, enums: PascalCase.
- Private fields: `_camelCase`.
- Locals and parameters: `camelCase`.
- Interfaces: `I` prefix.
- Constants: PascalCase, not screaming snake case.

### Imports and usings
- `ImplicitUsings` is enabled; do not add redundant basic framework usings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiquda/inputor](https://github.com/shiquda/inputor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
