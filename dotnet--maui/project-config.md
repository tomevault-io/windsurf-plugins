---
trigger: always_on
description: Guidance for GitHub Copilot when working on the .NET MAUI repository.
---


# GitHub Copilot Development Environment Instructions

This document provides specific guidance for GitHub Copilot when working on the .NET MAUI repository. It serves as context for understanding the project structure, development workflow, and best practices.

## Code Review Instructions

When performing a code review on PRs that change functional code, run the pr-finalize skill to verify that the PR title and description accurately match the actual implementation. This ensures proper documentation and helps maintain high-quality commit messages.

## Repository Overview

**.NET MAUI** is a cross-platform framework for creating mobile and desktop applications with C# and XAML. This repository contains the core framework code that enables development for Android, iOS, iPadOS, macOS, and Windows from a single shared codebase.

### Key Technologies

- **.NET SDK** - Version is **ALWAYS** defined in `global.json` at repository root
  - **main branch**: Latest stable .NET version
  - **Feature branches**: Each `netN.0` branch targets the .NET N SDK. By convention, the highest `netN.0` branch is the current development branch for new features and API changes.
- **Cake build system** for compilation and packaging (`dotnet cake`)
- **MSBuild** with custom build tasks (must build `Microsoft.Maui.BuildTasks.slnf` first)
- **Testing frameworks**:
  - **xUnit** - Unit tests (`*.UnitTests.csproj`)
  - **NUnit** - UI tests (`TestCases.Shared.Tests`)
  - **Appium WebDriver** - UI test automation

## Development Environment Setup

This guidance assumes:
- Repository is already cloned and tools are restored (`dotnet tool restore` completed)
- Build tasks are compiled (`Microsoft.Maui.BuildTasks.slnf` built successfully)
- Correct .NET SDK version installed (verify with `dotnet --version` against `global.json`)

### Platform-Specific Requirements

- **Android**: OpenJDK 17 + Android SDK (install via `android` command after `dotnet tool restore`)
- **iOS/macOS**: Xcode (current stable version)
- **Windows**: Windows SDK

## Project Structure

### Important Directories
- `src/Core/` - Core MAUI framework code
- `src/Controls/` - UI controls and components
- `src/Essentials/` - Platform APIs and essentials
- `src/TestUtils/` - Testing utilities and infrastructure
- `docs/` - Development documentation
- `eng/` - Build engineering and tooling
- `.github/` - GitHub workflows and configuration

### Platform-Specific Code Organization
- **Android** specific code is inside folders labeled `Android`
- **iOS** specific code is inside folders labeled `iOS`
- **MacCatalyst** specific code is inside folders named `MacCatalyst`
- **Windows** specific code is inside folders named `Windows`

### Platform-Specific File Extensions

Platform-specific files use naming conventions to control compilation:

**File extension patterns**:
- `.windows.cs` - Windows TFM only
- `.android.cs` - Android TFM only
- `.ios.cs` - iOS and MacCatalyst TFMs (both)
- `.maccatalyst.cs` - MacCatalyst TFM only (does NOT compile for iOS)

**Important**: Both `.ios.cs` and `.maccatalyst.cs` files compile for MacCatalyst. There is no precedence mechanism that excludes one when the other exists.

**Example**: If you have both `CollectionView.ios.cs` and `CollectionView.maccatalyst.cs`, both will compile for MacCatalyst builds. The `.maccatalyst.cs` file won't compile for iOS, but the `.ios.cs` file will compile for both iOS and MacCatalyst.

### Sample Projects

- `src/Controls/samples/Maui.Controls.Sample` - Full gallery sample with all controls and features
- `src/Controls/samples/Maui.Controls.Sample.Sandbox` - Empty project for testing/reproduction
- `src/Essentials/samples/Essentials.Sample` - Essentials API demonstrations (non-UI MAUI APIs)
- `src/BlazorWebView/samples/` - BlazorWebView sample applications

## Development Workflow

### Testing

Major test projects:
- **Core**: `src/Core/tests/UnitTests/Core.UnitTests.csproj`
- **Essentials**: `src/Essentials/test/UnitTests/Essentials.UnitTests.csproj`
- **Controls**: `src/Controls/tests/Core.UnitTests/Controls.Core.UnitTests.csproj`
- **XAML**: `src/Controls/tests/Xaml.UnitTests/Controls.Xaml.UnitTests.csproj`

Find all tests: `find . -name "*.UnitTests.csproj"`

### CI Pipelines (Azure DevOps)

When referencing or triggering CI pipelines, use these current pipeline names:

| Pipeline | Name | Purpose |
|----------|------|---------|
| Overall CI | `maui-pr` | Full PR validation build |
| Device Tests | `maui-pr-devicetests` | Helix-based device tests |
| UI Tests | `maui-pr-uitests` | Appium-based UI tests |

**⚠️ Old pipeline names** (e.g., `MAUI-UITests-public`, `MAUI-public`) are **outdated** and should NOT be used. Always use the names above.

### Investigating CI Failures

**🚨 ALWAYS use the `azdo-build-investigator` skill when investigating CI failures or assessing merge readiness.** Its instructions direct you to invoke the `ci-analysis` skill first for the core investigation workflow, then apply MAUI-specific corrections (correct pipeline names, XHarness quirks, binlog guidance).

Do NOT default to manually querying AzDO APIs or rely solely on `gh pr checks` pass/fail counts.

**When to use it:**
- "How does CI look?" / "Is CI green?" / "Can we merge?"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/maui](https://github.com/dotnet/maui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
