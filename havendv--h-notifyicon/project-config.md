---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

H.NotifyIcon is a cross-platform system tray icon library for .NET. It's a continuation of the hardcodet/wpf-notifyicon project, extended to support WinUI, Uno Platform, MAUI, and Console applications. The library provides native Windows tray icon functionality without relying on Windows Forms.

## Key Architecture Concepts

### Multi-Platform Structure

The codebase uses a **shared code pattern** with platform-specific implementations:

- **H.NotifyIcon.Shared/**: Contains shared source files compiled into each platform-specific library
  - Files are organized by feature: `TaskbarIcon.cs`, `TaskbarIcon.Properties.cs`, `TaskbarIcon.ContextMenu.cs`, etc.
  - Platform-specific implementations use naming conventions: `.Wpf.cs`, `.WinUI.cs`, `.WinRT.cs`
  - Conditional compilation with `#if HAS_WPF`, `#if HAS_WINUI`, `#if HAS_UNO`, `#if HAS_MAUI`

- **Platform Libraries**:
  - `H.NotifyIcon`: Core library with Windows interop via CsWin32, supports Console apps (targets net4.6.2, netstandard2.0, net8.0, net9.0)
  - `H.NotifyIcon.Wpf`: WPF wrapper with XAML integration (targets net4.6.2, net8.0-windows, net9.0-windows)
  - `H.NotifyIcon.WinUI`: WinUI 3 wrapper with WindowsAppSDK (targets net8.0-windows, net9.0-windows)
  - `H.NotifyIcon.Uno`: Uno Platform wrapper (targets net9.0, net9.0-android, net9.0-ios, net9.0-maccatalyst)
  - `H.NotifyIcon.Uno.WinUI`: Uno Platform with WinUI syntax (targets net9.0-windows10.0.19041.0)
  - `H.NotifyIcon.Maui`: MAUI wrapper (targets net9.0-android, net9.0-ios, net9.0-maccatalyst, net9.0-windows10.0.19041.0)

- **Icon Generation Libraries**:
  - `H.GeneratedIcons.System.Drawing`: Dynamic icon generation using System.Drawing
  - `H.GeneratedIcons.SkiaSharp`: Dynamic icon generation using SkiaSharp (alternative)

### Graphics Library Configuration

The `GraphicsLibrary` property in `src/libs/Directory.Build.props` controls which graphics backend is used:
- `System.Drawing` (default): Uses System.Drawing with `HAS_SYSTEM_DRAWING` define
- `SkiaSharp`: Uses SkiaSharp with `HAS_SKIA_SHARP` define

### Core Components

**TaskbarIcon** (main class split across multiple files):
- `TaskbarIcon.cs`: Core initialization and TrayIcon management
- `TaskbarIcon.Properties.cs`: Dependency properties using DependencyPropertyGenerator
- `TaskbarIcon.ContextMenu.*.cs`: Platform-specific context menu implementations
- `TaskbarIcon.MouseEvents.cs`, `TaskbarIcon.KeyboardEvents.cs`: Input handling
- `TaskbarIcon.Notifications.cs`: Balloon notifications
- `TaskbarIcon.IconSource.cs`: Icon source binding and conversion

**TrayIcon** (in H.NotifyIcon core):
- Low-level Windows Shell_NotifyIcon API wrapper via CsWin32
- Manages NOTIFYICONDATA structure and message window

**GeneratedIconSource**:
- Allows dynamic icon generation from text, emojis, or graphics
- Cross-platform implementations for WPF and WinRT

### Source Generators

The project heavily uses source generators:
- **DependencyPropertyGenerator**: Generates WPF/WinUI dependency properties from attributes
- **EventGenerator**: Generates event infrastructure
- **CsWin32**: Generates P/Invoke code from Windows metadata

## Build and Development

### Prerequisites

- .NET 9 SDK
- MAUI workload: `dotnet workload install maui`
- Tizen workload (Windows): Run the workload-install.ps1 script from Samsung Tizen.NET repo

### Building

Build all libraries:
```powershell
Get-ChildItem -Path src/libs -Recurse -Filter *.csproj | ForEach-Object { dotnet build $_.FullName --configuration Release }
```

Build specific platform library:
```bash
dotnet build src/libs/H.NotifyIcon.Wpf/H.NotifyIcon.Wpf.csproj --configuration Release
```

Build core library only:
```bash
dotnet build src/libs/H.NotifyIcon/H.NotifyIcon.csproj --configuration Release
```

### Testing

Run integration tests (Windows only):
```bash
dotnet test src/tests/H.NotifyIcon.IntegrationTests/H.NotifyIcon.IntegrationTests.csproj
```

Tests target both net4.8 and net9.0-windows frameworks.

### Running Sample Apps

Sample applications demonstrate platform-specific usage:

WPF app:
```bash
dotnet run --project src/apps/H.NotifyIcon.Apps.Wpf/H.NotifyIcon.Apps.Wpf.csproj
```

WinUI app (requires x64/x86/arm64 RID):
```bash
dotnet run --project src/apps/H.NotifyIcon.Apps.WinUI/H.NotifyIcon.Apps.WinUI.csproj
```

Console app:
```bash
dotnet run --project src/apps/H.NotifyIcon.Apps.Console/H.NotifyIcon.Apps.Console.csproj
```

MAUI app:
```bash
dotnet build src/apps/H.NotifyIcon.Apps.Maui/H.NotifyIcon.Apps.Maui.csproj -f net9.0-windows10.0.19041.0
```

## Code Patterns and Conventions

### Conditional Compilation

Platform-specific code uses consistent defines:
- `HAS_WPF`: WPF-specific code
- `HAS_WINUI`: WinUI 3-specific code
- `HAS_UNO`: Uno Platform-specific code
- `HAS_MAUI`: MAUI-specific code
- `HAS_SYSTEM_DRAWING`: System.Drawing graphics backend
- `HAS_SKIA_SHARP`: SkiaSharp graphics backend

### Assembly Signing

All library projects are signed with `src/libs/key.snk` (except Uno which has `SignAssembly>false`).

### NuGet Packaging

- Packages are automatically generated on Release builds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HavenDV/H.NotifyIcon](https://github.com/HavenDV/H.NotifyIcon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
