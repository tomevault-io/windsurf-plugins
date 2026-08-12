---
trigger: always_on
description: This document captures repository-specific skills, architecture patterns, and problem areas for AI copilots working on SubZeroFramework.
---

# SubZeroFramework Copilot Skills

## Purpose
This document captures repository-specific skills, architecture patterns, and problem areas for AI copilots working on SubZeroFramework.

Use `docs/ReleasePlan.md` as the execution roadmap and priority list.

Use `docs/FunctionalitySpecification.md` as the source of truth for intended menu-item and page behavior.

Use `docs/Architecture.md` as the source of truth for the service/client split, privilege boundary, process lifecycle, IPC ownership, and multi-instance assumptions.

## Key areas of expertise

### Important steps
- Avoid using PowerShell unless MCP tools (such as Microsoft Knowledge Search or Nuget Package Search) are completely unavailable or fail to provide the required documentation, code samples, or best practices for the tasks at hand.
- Always refer to `docs/ReleasePlan.md` for the current list of required improvements and align your work with those items.
- Refer to `docs/FunctionalitySpecification.md` when working on navigation, page responsibilities, or user-facing surface behavior.
- Refer to `docs/Architecture.md` when working on service/client boundaries, privileges, shutdown behavior, IPC ownership, or multi-instance behavior.
- When modifying service or core service-boundary code, prefer structured DI-backed logging with `ILogger<T>` at lifecycle boundaries, mutating commands, direct stream writes, publish points, authorization rejections, and exceptional shutdown/restore paths.
- If you need source codes, preferably use the GitHub web interface to navigate and search the codebase, as it provides better context and understanding of the code structure. Use the file paths and class names mentioned in this document to locate relevant code sections.
If possible use ObservableProperty with ObservableObject, leveraging C# partial classes to reduce boilerplate and ensure change notifications are properly raised for UI updates. This is especially important for view models and any state that the UI binds to.
- Prefer `[NotifyPropertyChangedFor]` and `[NotifyCanExecuteChangedFor]` on `[ObservableProperty]` dependencies instead of manual `OnPropertyChanged(...)` or `NotifyCanExecuteChanged()` calls; repo analyzers `SZF0001` and `SZF0002` enforce this.
- Prefer `[ObservableProperty]` public partial properties over manual `SetProperty(...)` wrappers for bindable state; repo analyzer `SZF0012` enforces this.
- For inventory surfaces, prefer FrameworkDotnet data first and only use Hardware.Info to fill gaps, keeping that fallback flow behind the existing service/gRPC/client boundary.
- For service lifecycle work, keep install/update/shutdown/restart/autorun management out of gRPC. Prefer the packaged service executable and `FrameworkServiceManagementCli` so the client stays unelevated and the action still works when the service is offline.
- For service-owned runtime settings such as polling cadence or writable fan-command authorization, prefer `FrameworkServiceConfigurationGrpcService` / `IFrameworkServiceConfigurationClient` plus the persistent service-owned configuration overlay instead of client-local settings. That surface follows an Apply/Save/Load model (Apply changes live runtime and broadcasts, Save persists to the service-approved JSON path, Load re-reads and broadcasts). Keep lifecycle operations out of that gRPC surface. Relocating `service-settings.json` out of its default `ProgramData` location is deferred: only this one store exists service-side (display units are client-local — see below), so if implemented it would be a single service-side "relocate store" gRPC flow (validate writable by the service account, atomically copy → swap pointer → delete old → broadcast new path, persist a bootstrap pointer file in the default location), driven from the client by an Uno `FolderPicker` rather than a free-text path field.
- Keep service and gRPC payloads in canonical units. When the UI needs alternate display units, persist the choice locally through `IUserUnitPreferencesClient` and format values, suffixes, and axis labels through `IUnitFormattingService`.
- When a bindable display value depends on other observable properties, prefer analyzer-friendly dependency patterns such as `[NotifyPropertyChangedFor]` over manual `OnPropertyChanged(...)` refresh calls so the solution stays warning-clean.
- Preserve stable item identity in GridView/ListView card layouts. Prefer persistent mutable card/view-model instances exposed via `ReadOnlyObservableCollection<T>` over rebinding fresh arrays every refresh, otherwise cards blink and pointer/layout state resets.
- Keep shared recent-history window labels and separator-step defaults in `PresentationDefaults`; `TimeChartAxisHelper` should only translate those policies into axis limits and separators.
- Re-read any existing XAML page immediately before editing it. The user often makes small manual visual tweaks between turns, and those should be preserved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TekuSP/SubZeroFramework](https://github.com/TekuSP/SubZeroFramework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
