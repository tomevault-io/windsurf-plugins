---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

SyncClipboard is a cross-platform clipboard synchronization tool (Windows/macOS/Linux) with clipboard history management. It supports a standalone ASP.NET Core server, client-built-in server, WebDAV, or S3-compatible storage as the sync backend. Desktop clients use either Avalonia (cross-platform) or WinUI3 (Windows-native) for UI.

## Build & Test Commands

This is a multi-platform .NET solution. The sln cannot be built as a whole (it contains platform-specific projects that fail on incompatible OS/arch). All projects use [central package management](https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management) via `src/Directory.Packages.props`. Commands below are run from the repo root unless noted.

### Windows (WinUI3)

WinUI3 is the primary Windows client (`net9.0-windows10.0.19041.0`). Requires the Windows App SDK — only builds on Windows. The CI uses msbuild, not dotnet CLI.

```bash
# Restore
dotnet restore src/SyncClipboard.WinUI3

# Build (Release, x64 — CI command)
msbuild src\SyncClipboard.WinUI3\SyncClipboard.WinUI3.csproj \
  /p:Platform=x64 /p:RuntimeIdentifier=win-x64 /p:Configuration=Release \
  /p:WindowsAppSDKSelfContained=true /p:SelfContained=true /v:m -restore
```

For `arm64`, swap `/p:Platform=arm64 /p:RuntimeIdentifier=win-arm64`. Set `/p:SelfContained=false` to skip bundling the .NET runtime; set `/p:WindowsAppSDKSelfContained=false` to skip bundling the Windows App SDK.

### Linux (Avalonia)

Linux uses `SyncClipboard.Desktop.Default` (Avalonia, `net8.0`).

```bash
dotnet build src/SyncClipboard.Desktop.Default/SyncClipboard.Desktop.Default.csproj
```

### macOS (Avalonia)

macOS uses `SyncClipboard.Desktop.MacOS` (Avalonia, `net9.0-macos`). Requires the `macos` workload and only builds on macOS. CI runs from the project directory.

```bash
# Restore (from src/SyncClipboard.Desktop.MacOS/)
dotnet restore src/SyncClipboard.Desktop.MacOS

# Publish (CI command, from src/SyncClipboard.Desktop.MacOS/)
dotnet publish src/SyncClipboard.Desktop.MacOS/SyncClipboard.Desktop.MacOS.csproj \
  -r osx-x64 -c Release
```

Replace `osx-x64` with `osx-arm64` for Apple Silicon.

### Server

The standalone server is cross-platform `net8.0`.

```bash
# Restore
dotnet restore src/SyncClipboard.Server

# Build (Debug)
dotnet build src/SyncClipboard.Server

# Publish (CI command)
dotnet publish src/SyncClipboard.Server/SyncClipboard.Server.csproj \
  --configuration Release --no-restore
```

### Shared / Core Libraries

These are platform-agnostic and build anywhere:

```bash
dotnet build src/SyncClipboard.Shared
dotnet build src/SyncClipboard.Core
dotnet build src/SyncClipboard.Server.Core
dotnet build src/SyncClipboard.Desktop
```

### Tests

```bash
# Run all tests
dotnet test src/SyncClipboard.Test
dotnet test src/SyncClipboard.Test.Desktop
dotnet test src/SyncClipboard.Test.WinUI3   # Windows only

# Run a single test
dotnet test src/SyncClipboard.Test --filter "FullyQualifiedName~TestClassName"
```

Tests use MSTest with Moq. `ServiceProviderDataSource` attributes drive DI validation tests that ensure all registered services can be resolved.

### Code Style

```bash
# Check formatting (from src/ directory)
dotnet format --verify-no-changes --severity info --no-restore
```

Rules are defined in `src/.editorconfig`.

## Architecture

### Project Dependency Graph

```
SyncClipboard.Shared          — Profile models, IProfileEnv, utilities (no heavy deps)
    ↑
SyncClipboard.Server.Core     — ASP.NET Core library: controllers, SignalR hub, EF Core SQLite history, basic auth
    ↑              ↑
SyncClipboard.Core ───────────┘  — Core business logic: services, remote server adapters, ViewModels, DI setup (AppCore)
    ↑
SyncClipboard.Desktop         — Shared Avalonia desktop UI: clipboard factory, views, tray icon, hotkeys
    ↑
    ├── SyncClipboard.Desktop.Default   — Windows/Linux desktop executable (Avalonia, net8.0)
    ├── SyncClipboard.Desktop.MacOS     — macOS desktop executable (Avalonia, net9.0-macos)
    └── SyncClipboard.WinUI3            — WinUI3 native Windows executable (net9.0-windows10.0.19041.0)

SyncClipboard.Server           — Standalone server executable (wraps SyncClipboard.Server.Core)
```

### Core Concepts

**Profile Model (`SyncClipboard.Shared`):** Abstract `Profile` is the unified clipboard content representation. Subtypes: `TextProfile`, `ImageProfile`, `FileProfile`, `GroupProfile`. Every profile has a lazily-computed hash used for deduplication. Profiles can be serialized to `ProfileDto` for wire transfer and `ProfilePersistentInfo`/`ProfileLocalInfo` for storage.

**Service Pattern (`SyncClipboard.Core`):** Every runnable feature implements `IService` (abstract base: `Service`). `ServiceManager` orchestrates all services through their lifecycle: `Start()` → `RegistEventHandler()` → `Load()` (on config change) → `Stop()`. Register new services in `AppCore.ConfigurateUserService()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jeric-X/SyncClipboard](https://github.com/Jeric-X/SyncClipboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
