---
trigger: always_on
description: Guidance for GitHub Copilot when working on the Syncfusion Toolkit for .NET MAUI repository.
---


# GitHub Copilot Development Environment Instructions

This document provides specific guidance for GitHub Copilot when working on the Syncfusion Toolkit for .NET MAUI repository. It serves as context for understanding the project structure, development workflow, and best practices.

## Code Review Instructions

When performing a code review on PRs that change functional code, verify that the PR title and description accurately match the actual implementation and reference the correct issue number(s) in the `Fixes #` field.

## Repository Overview

**Syncfusion Toolkit for .NET MAUI** is an open-source UI controls library built on top of .NET MAUI, providing rich, cross-platform UI controls and components for Android, iOS, macOS, and Windows applications.

### Key Technologies

- **.NET MAUI** — Cross-platform UI framework (Android, iOS, macOS, Windows)
- **.NET SDK** — Version `10.0.x` (see `pipelines/build.yml`)
- **MSBuild** — Solution: `Syncfusion.Maui.Toolkit.sln` at repository root
- **xUnit** — Unit testing framework (`Syncfusion.Maui.Toolkit.UnitTest`)
- **C# with nullable reference types** enabled globally (`Directory.Build.props`)

### Repository Source

- **GitHub**: `https://github.com/syncfusion/maui-toolkit`
- **Primary branch**: `main` (all PRs target `main`)

---

## Development Environment Setup

### Prerequisites

- **.NET 10 SDK** — `dotnet --version` should return `10.x`
- **Visual Studio 17.10+** (Windows) or **VS Code + .NET MAUI Dev Kit** (macOS)
- **Android SDK** — Install via Visual Studio or Android SDK Manager
- **OpenJDK 17** — Required for Android targets
- **Xcode** (macOS only) — Required for iOS and MacCatalyst

### Initial Setup

```bash
# Restore NuGet packages and build the solution
dotnet restore ./Syncfusion.Maui.Toolkit.sln
dotnet build ./Syncfusion.Maui.Toolkit.sln
```

### Platform Requirements

| Platform | Host | Requirements |
|----------|------|-------------|
| Android | Windows / macOS | OpenJDK 17 + Android SDK |
| iOS / MacCatalyst | macOS | Xcode (current stable) |
| Windows | Windows | Windows SDK |

---

## Project Structure

### Repository Root

```
maui-toolkit/
├── maui/
│   ├── src/           ← All control source code
│   ├── tests/         ← Unit tests
│   └── samples/       ← Sample gallery and sandbox
├── samples/           ← Top-level sample apps
├── pipelines/         ← Azure DevOps CI pipelines
├── targets/           ← MSBuild targets
├── .github/           ← GitHub workflows, templates, agents
├── Directory.Build.props
└── Syncfusion.Maui.Toolkit.sln
```

### Source Controls (`maui/src/`)

Each control lives in its own folder under `maui/src/`:

| Control | Path |
|---------|------|
| Charts (Cartesian, Circular, Funnel, Pyramid) | `maui/src/Charts/` |
| Spark Charts | `maui/src/SparkCharts/` |
| Sunburst Chart | `maui/src/SunburstChart/` |
| Calendar | `maui/src/Calendar/` |
| Popup | `maui/src/Popup/` |
| Bottom Sheet | `maui/src/BottomSheet/` |
| Tab View | `maui/src/TabView/` |
| Accordion / Expander | `maui/src/Accordion/`, `maui/src/Expander/` |
| Navigation Drawer | `maui/src/NavigationDrawer/` |
| Carousel | `maui/src/Carousel/` |
| Cards | `maui/src/Cards/` |
| Chip | `maui/src/Chip/` |
| Segmented Control | `maui/src/SegmentedControl/` |
| Button | `maui/src/Button/` |
| Numeric Entry | `maui/src/NumericEntry/` |
| OTP Input | `maui/src/OtpInput/` |
| Text Input Layout | `maui/src/TextInputLayout/` |
| Picker | `maui/src/Picker/` |
| Pull to Refresh | `maui/src/PullToRefresh/` |
| Progress Bar | `maui/src/ProgressBar/` |
| Shimmer | `maui/src/Shimmer/` |
| Effects View | `maui/src/EffectsView/` |
| Core (shared utilities) | `maui/src/Core/` |

### Core Infrastructure (`maui/src/Core/`)

Shared utilities used across all controls:

- `Animation/` — Animation helpers
- `BaseView/` — Base view classes (`SfView`)
- `ButtonBase/` — Button base implementation
- `DrawableView/` — Custom drawable canvas view
- `EntryView/` — Entry-based view utilities
- `Extensions/` — Extension methods
- `FontElement/` — Font-related bindable properties
- `GestureDetector/` — Cross-platform gesture support
- `Helper/` — General helper utilities
- `KeyboardDetector/` — Keyboard event handling
- `Legend/` — Shared legend for chart controls
- `Localization/` — Localization resources
- `PickerView/` — Base picker implementation
- `TabView/` — Tab view base
- `TextMeasurer/` — Text size measurement
- `TextStyle/` — Text styling utilities
- `Theme/` — Light/dark theme support
- `Tooltip/` — Tooltip implementation
- `TouchDetector/` — Touch event detection
- `ViewExtensions/` — View helper extensions
- `WindowOverlay/` — Window overlay support

### Platform-Specific Code

Platform-specific files use folder and naming conventions:

**Subfolder convention:**
- `Android/` — Android-only implementations
- `iOS/` — iOS-only implementations
- `MacCatalyst/` — MacCatalyst-only implementations
- `Windows/` — Windows-only implementations

**File extension convention:**
- `.android.cs` — Android TFM only
- `.ios.cs` — iOS and MacCatalyst TFMs
- `.maccatalyst.cs` — MacCatalyst TFM only (does NOT compile for iOS)
- `.windows.cs` — Windows TFM only


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syncfusion/maui-toolkit](https://github.com/syncfusion/maui-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
