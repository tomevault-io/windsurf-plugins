---
trigger: always_on
description: Copyright (C) 2024-2026 AgOpenWeb Contributors
---

<!--
AgOpenWeb
Copyright (C) 2024-2026 AgOpenWeb Contributors

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see <https://www.gnu.org/licenses/>.
-->

# AGENTS.md - AgOpenWeb3

This file provides guidance to Codex when working with this repository.

**Key documentation:**
- **[Plans/ARCHITECTURE.md](Plans/ARCHITECTURE.md)** - Full architecture: services, state management, data flow
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contributor guide with cross-platform parity rules
- **[PGN.md](PGN.md)** - UDP packet protocol for hardware communication

## Project Overview

AgOpenWeb3 is a cross-platform agricultural GPS guidance application built with Avalonia UI. It's a clean rewrite achieving **91.7% shared code** across platforms.

**What it does:**
- Real-time GPS guidance for agricultural equipment
- Field boundary management and recording
- Unified track guidance (AB lines and curves use same system)
- U-turn path generation and following
- Section control for sprayers/planters
- NTRIP RTK corrections support
- Configurable keyboard hotkeys
- Integration with AgOpenGPS ecosystem via UDP

## Architecture

```
AgOpenWeb3/
├── Shared/                              # ~92% - Platform-agnostic code
│   ├── AgOpenWeb.Models/            # Data models, geometry, configuration, DTOs
│   ├── AgOpenWeb.Services/          # Business logic, GPS, NTRIP, UDP
│   ├── AgOpenWeb.ViewModels/        # MVVM ViewModels (ReactiveUI)
│   └── AgOpenWeb.Views/             # Shared UI controls, panels, dialogs
│
├── Platforms/                           # ~8% - Platform-specific code
│   ├── AgOpenWeb.Desktop/           # Windows/macOS/Linux
│   ├── AgOpenWeb.iOS/              # iOS/iPadOS
│   └── AgOpenWeb.Android/          # Android
│
├── Tests/                              # NUnit test projects
│   ├── AgOpenWeb.Models.Tests/     # Geometry, coordinate conversion (72 tests)
│   ├── AgOpenWeb.Services.Tests/   # NMEA parsing, guidance (21 tests)
│   └── AgOpenWeb.UI.Tests/         # Headless UI tests via Avalonia.Headless (18 tests)
│
├── TestRunner/                         # Legacy test harness for guidance algorithms
└── AgOpenWeb.sln                    # Solution file
```

### Platform Support

| Platform | Project | Notes |
|----------|---------|-------|
| Windows | AgOpenWeb.Desktop | Same codebase as macOS/Linux |
| macOS | AgOpenWeb.Desktop | Same codebase as Windows/Linux |
| Linux | AgOpenWeb.Desktop | Same codebase as Windows/macOS |
| iOS/iPadOS | AgOpenWeb.iOS | Requires Xcode 26.3+, runs on ARM64 simulator |
| Android | AgOpenWeb.Android | APK build, sideload install |

## Build Commands

```bash
# Build and run Desktop (works on Windows, macOS, Linux)
dotnet build Platforms/AgOpenWeb.Desktop/AgOpenWeb.Desktop.csproj
dotnet run --project Platforms/AgOpenWeb.Desktop/AgOpenWeb.Desktop.csproj

# Build iOS (requires macOS with Xcode 26.3+)
dotnet build Platforms/AgOpenWeb.iOS/AgOpenWeb.iOS.csproj -c Debug -f net10.0-ios -r iossimulator-arm64

# Deploy and run iOS on simulator
dotnet build Platforms/AgOpenWeb.iOS/AgOpenWeb.iOS.csproj -c Debug -f net10.0-ios -r iossimulator-arm64 -t:Run

# Alternative iOS deployment (if -t:Run doesn't work)
xcrun simctl install booted Platforms/AgOpenWeb.iOS/bin/Debug/net10.0-ios/iossimulator-arm64/AgOpenWeb.iOS.app
xcrun simctl launch booted com.agopenweb.ios

# Build Android APK
dotnet build Platforms/AgOpenWeb.Android/AgOpenWeb.Android.csproj

# Build entire solution
dotnet build AgOpenWeb.sln

# Run tests
dotnet test Tests/
```

## Key Design Decisions

### Rendering: SkiaMapControl via CompositionCustomVisualHandler
All platforms render the map through `SkiaMapControl`, which leases the Skia
GPU surface inside a `CompositionCustomVisualHandler` and re-arms via
`RegisterForNextAnimationFrameUpdate`. This bucket sits outside the Av12
commit throttle that capped `OpenGlControlBase` at ~32 FPS on iPad
([issue #21409](https://github.com/AvaloniaUI/Avalonia/issues/21409)).
True perspective comes from `SKMatrix44`; top-down mode is just
`pitch = 90°` on the same control — no second renderer behind a toggle.

### Shared UI Components
All panels, dialogs, and controls live in `AgOpenWeb.Views`:
- `Controls/SkiaMapControl.cs` - Main map rendering
- `Controls/DialogOverlayHost.axaml` - Hosts all modal dialog overlays (shared across platforms)
- `Controls/Panels/` - LeftNavigationPanel, SimulatorPanel, SectionControlPanel, etc.
- `Controls/Dialogs/` - All modal dialogs (FieldSelection, DataIO, AgShare, etc.)
- `Converters/` - Shared value converters (BoolToColor, FixQualityToColor, etc.)

### Dialog System
Dialogs use a centralized state machine in `UIState`. Only one dialog can be open at a time.

**To show a dialog from a ViewModel:**
```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgOpenGPS-Official/AgOpenWeb](https://github.com/AgOpenGPS-Official/AgOpenWeb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
