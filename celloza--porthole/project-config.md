---
trigger: always_on
description: - Build Porthole as a pure native Windows desktop app.
---

# Porthole Copilot Instructions

## Product direction

- Build Porthole as a pure native Windows desktop app.
- Use WinUI 3 for the dashboard UI.
- Do not introduce web-based wrappers such as Electron, WebView shells, or browser-hosted UI.
- Do not pivot this project to MAUI unless the user explicitly asks for that change again.
- Keep the visual direction modern, desktop-first, and Windows-native.

## Core technical constraints

- Target `net8.0-windows10.0.19041.0` across the solution unless the user explicitly approves a framework migration.
- Keep `Microsoft.WSL.Containers` isolated to the tray/backend side of the solution.
- Do not add `Microsoft.WSL.Containers` references to the WinUI app project or to shared UI-facing layers.
- Do not scrape `stdout` from `wslc` for core functionality when the SDK can provide the data directly.
- Prefer typed integrations and explicit contracts over shell parsing.

## Solution architecture

The solution currently has three main projects:

### `src/Porthole.App`

- WinUI 3 dashboard application.
- Owns shell, navigation, pages, window behavior, and desktop UX.
- Uses dependency injection and MVVM.
- Should stay focused on UI orchestration and app lifecycle.

### `src/Porthole.Core`

- Shared models, service abstractions, IPC contracts, and viewmodels.
- Safe place for app-facing interfaces and DTOs.
- Must remain free of direct `Microsoft.WSL.Containers` dependencies.

### `src/Porthole.Tray`

- Background tray host and backend process.
- Owns real container/image operations through `Microsoft.WSL.Containers`.
- Hosts the named-pipe server used by the dashboard.
- Double-clicking the tray icon should open or foreground the dashboard.
- Tray startup currently auto-launches the dashboard and reuses an existing dashboard window when possible.

## IPC rules

- The app and tray communicate through named pipes using JSON request/response/progress envelopes.
- Shared contracts live under `src/Porthole.Core/Services/NamedPipe`.
- If you add or change an operation, update both the client and server sides together.
- Preserve typed progress reporting for long-running operations such as image pulls.

**Operation Codes:**
- Image operations (0-19): List, Pull, Delete, Tag, Prune
- Session operations (20-24): ListSessions, CreateSession, DeleteSession, SetActiveSession, GetActiveSession
- Networking operations (30-31): GetNetworkingSnapshot, SetNetworkMode

**Async Operations:**
- Long-running operations like `PullImage` use `ExecuteWithTimeoutAsync` and yield progress updates
- Synchronous-looking operations like `DeleteImage` complete without progress tracking
- Port binding enumeration (`GetNetworkingSnapshot`) is async to handle JSON parsing and multiple container inspections
- Always await async operations in pipe server handlers; use `cancellationToken` for timeout safety

## UI and MVVM expectations

- Use `CommunityToolkit.Mvvm` patterns for viewmodels and commands.
- Prefer DI-constructed services and viewmodels over ad-hoc singletons.
- Keep page code-behind light; business logic belongs in viewmodels or services.
- Avoid duplicate top-level page titles when the shell header already provides the page title.

## Windows App SDK and startup guidance

- The dashboard is configured for unpackaged local runs.
- Preserve the current `Porthole.App.csproj` startup-related settings unless there is a strong reason to change them:
  - `WindowsPackageType=None`
  - `EnableWinAppRunSupport=false`
  - `WindowsAppSdkBootstrapInitialize=true`
  - `WindowsAppSdkDeploymentManagerInitialize=false`
- Be careful when changing app startup code because this project previously hit pre-managed startup failures.

## Build and run expectations

- Prefer validating changes with `dotnet build Porthole.slnx -c Debug`.
- If `Porthole.Tray.exe` is already running, rebuilds of the tray project can fail because Windows locks the executable.
- When running the tray locally with `dotnet run`, the tray now resolves `Porthole.App.exe` from either:
  - its own output directory, or
  - the dashboard project's build output under `src/Porthole.App/bin/...`
- If you change output paths or runtime identifiers, verify tray-to-dashboard launch behavior still works.

## Current implementation status

**Implemented:**
- ✅ Shell/navigation: full page routing and menu structure
- ✅ Dashboard page: real-time system metrics and container status
- ✅ Images page: pull, tag, delete with progress tracking
- ✅ Containers page: start, stop, remove with inspection
- ✅ **Sessions page**: multi-session management with create/switch/delete operations
- ✅ **Networking page**: network mode toggle, port binding display, proxy configuration
- ⏳ Settings page: placeholder/lightweight
- ⏳ Run Wizard page: planned for next iteration

**Architectural Patterns in Use:**
- Multi-session support: All container operations target the active session context (set via `_activeSessionName` in backend)
- Session storage: WSL Containers SDK manages session filesystem; `_sessionSettings` dict tracks name→path mapping as workaround for Session.Settings not being exposed
- Port binding enumeration: Iterates running containers (State=2), calls `wslc inspect <name>` for each, parses Ports JSON object

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [celloza/porthole](https://github.com/celloza/porthole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
