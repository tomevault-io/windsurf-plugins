---
trigger: always_on
description: This repository contains **DeviceRunners**, a comprehensive cross-platform device testing framework for .NET applications. This document provides context and guidelines to help GitHub Copilot assist with development.
---

# DeviceRunners - GitHub Copilot Instructions

This repository contains **DeviceRunners**, a comprehensive cross-platform device testing framework for .NET applications. This document provides context and guidelines to help GitHub Copilot assist with development.

## Project Overview

DeviceRunners enables running unit tests, integration tests, and UI tests on real devices across multiple platforms (Android, iOS, macOS, Windows) using various testing frameworks (Xunit, NUnit). The project originated from migrating and modernizing existing testing solutions to .NET MAUI.

### Key Value Propositions
- **Device-native testing**: Run tests directly on target devices rather than just on development machines
- **Multi-platform support**: Single codebase supporting Android, iOS, macOS (Catalyst), and Windows
- **Multiple test runners**: Visual runners for interactive testing, CLI runners for CI/CD
- **Framework flexibility**: Support for Xunit, NUnit, and extensible for other frameworks

## Architecture Overview

### Core Components

1. **DeviceRunners.Core**: Shared foundation library containing platform-agnostic interfaces and base classes
2. **DeviceRunners.VisualRunners**: Interactive GUI test runners with real-time test execution and results
3. **DeviceRunners.XHarness**: CLI-based test runners integrating with Microsoft's XHarness tool for CI/CD
4. **DeviceRunners.UITesting**: UI testing infrastructure with Xunit integration
5. **DeviceRunners.Cli**: Cross-platform command-line tool for device operations (certificates, packages, testing)

### Testing Framework Integration

- **DeviceRunners.VisualRunners.Xunit**: Xunit integration for visual runners
- **DeviceRunners.VisualRunners.NUnit**: NUnit integration for visual runners
- **DeviceRunners.XHarness.Xunit**: Xunit integration for XHarness CLI runners

### Platform Support

```xml
<TargetFrameworks>net9.0;net9.0-android;net9.0-ios;net9.0-maccatalyst;net9.0-windows10.0.19041.0</TargetFrameworks>
```

Each library uses conditional compilation and platform-specific folders:
- `Platforms/Android/` - Android-specific implementations
- `Platforms/iOS/` - iOS-specific implementations  
- `Platforms/MacCatalyst/` - macOS Catalyst implementations
- `Platforms/Windows/` - Windows/WinUI implementations
- `Platforms/Apple/` - Shared iOS/macOS implementations
- `Platforms/All/` - Cross-platform implementations

## Project Structure Conventions

### Source Organization
```
src/
├── DeviceRunners.Core/                 # Core abstractions and interfaces
├── DeviceRunners.VisualRunners/        # Base visual runner implementation
├── DeviceRunners.VisualRunners.Maui/   # MAUI-specific visual runner components
├── DeviceRunners.VisualRunners.Xunit/  # Xunit visual runner
├── DeviceRunners.VisualRunners.NUnit/  # NUnit visual runner
├── DeviceRunners.XHarness/             # Base XHarness runner
├── DeviceRunners.XHarness.Maui/        # MAUI XHarness integration
├── DeviceRunners.XHarness.Xunit/       # Xunit XHarness runner
├── DeviceRunners.UITesting/            # UI testing base
├── DeviceRunners.UITesting.Maui/       # MAUI UI testing
├── DeviceRunners.UITesting.Xunit/      # Xunit UI testing
└── DeviceRunners.Cli/                  # Command-line tool
```

### Test Organization
```
test/
├── DeviceRunners.Cli.Tests/            # CLI tool unit tests
├── DeviceRunners.VisualRunners.Tests/  # Visual runner unit tests
└── TestProject.Tests/                  # General test utilities
```

### Sample Applications
```
sample/
├── src/
│   ├── DeviceTestingKitApp/            # Main MAUI sample app
│   ├── DeviceTestingKitApp.Library/    # Framework-agnostic library
│   └── DeviceTestingKitApp.MauiLibrary/ # MAUI-specific library
└── test/
    ├── DeviceTestingKitApp.DeviceTests/           # Device-specific tests
    ├── DeviceTestingKitApp.Library.NUnitTests/    # NUnit tests
    └── DeviceTestingKitApp.MauiLibrary.XunitTests/ # Xunit tests
```

## Coding Conventions

### General .NET Standards
- **Nullable reference types**: Enabled project-wide (`<Nullable>enable</Nullable>`)
- **Implicit usings**: Enabled (`<ImplicitUsings>enable</ImplicitUsings>`)
- **Target framework**: .NET 9.0 as baseline with platform-specific targets
- **Central Package Management**: All package versions managed in `Directory.Packages.props`

### Platform-Specific Code Patterns

Use conditional compilation for platform-specific implementations:

```csharp
#if ANDROID
    // Android-specific code
    return Android.App.Application.Context.CacheDir.AbsolutePath;
#elif IOS || MACCATALYST
    // iOS/macOS-specific code
    var root = NSBundle.MainBundle.BundlePath;
#elif WINDOWS
    // Windows-specific code
    if (IsPackagedApp.Value)
        return Windows.ApplicationModel.Package.Current.InstalledLocation.Path;
#endif
```

### Service-Oriented Architecture

The CLI tool demonstrates clean service architecture:

```csharp
// Services are injected and focused on single responsibilities
public class PackageService
{
    public Task<PackageIdentity> GetPackageIdentity(string packagePath) { }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattleibow/DeviceRunners](https://github.com/mattleibow/DeviceRunners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
