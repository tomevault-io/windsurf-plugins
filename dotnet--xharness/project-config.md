---
trigger: always_on
description: XHarness is a .NET command-line tool that enables running xUnit/NUnit tests on mobile platforms (Android, Apple iOS/tvOS/watchOS/xrOS/Mac Catalyst), WASI, and desktop browsers (WASM). It is part of the .NET ecosystem and is essential for cross-platform testing in the .NET Foundation projects.
---

# XHarness Repository Copilot Instructions

## Project Overview

XHarness is a .NET command-line tool that enables running xUnit/NUnit tests on mobile platforms (Android, Apple iOS/tvOS/watchOS/xrOS/Mac Catalyst), WASI, and desktop browsers (WASM). It is part of the .NET ecosystem and is essential for cross-platform testing in the .NET Foundation projects.

**Key Capabilities:**
- Device/emulator management and discovery
- Application lifecycle management (install, run, uninstall)
- Test execution and result collection in multiple formats (text, xUnit/NUnit XML)
- Crash dump collection and symbolication
- TCP and USB connection modes
- Apple Simulator runtime installation
- Integration with Helix cloud testing infrastructure

## Architecture Overview

XHarness is organized into two main layers:

### Tooling Layer (src/)
- **Microsoft.DotNet.XHarness.CLI** - Main CLI entry point with command definitions
- **Microsoft.DotNet.XHarness.Android** - Android-specific operations using ADB
- **Microsoft.DotNet.XHarness.Apple** - Apple platform operations using mlaunch
- **Microsoft.DotNet.XHarness.iOS.Shared** - Apple mobile platforms shared functionality
- **Microsoft.DotNet.XHarness.Common** - Core building blocks (logging, execution, utilities, diagnostics)

### Application Layer (src/)
- **Microsoft.DotNet.XHarness.TestRunners.Common** - Test discovery, execution, and results aggregation
- **Microsoft.DotNet.XHarness.TestRunners.Xunit** - XUnit framework integration
- **Microsoft.DotNet.XHarness.TestRunners.NUnit** - NUnit framework integration
- **Microsoft.DotNet.XHarness.DefaultAndroidEntryPoint.Xunit** - Default Android entry point

## Command Structure

XHarness follows a platform → command pattern:
```bash
xharness [platform] [command] [options]
```

### Supported Platforms and Commands:

**Android Commands:**
- `AndroidTest`, `AndroidDevice`, `AndroidInstall`, `AndroidRun`, `AndroidUninstall`, `AndroidAdb`, `AndroidState`

**Apple Commands:**
- `AppleTest`, `AppleRun`, `AppleInstall`, `AppleUninstall`, `AppleJustTest`, `AppleJustRun`, `AppleDevice`, `AppleMlaunch`, `AppleState`

**Apple Simulator Commands:**
- `List`, `Find`, `Install`, `ResetSimulator`

**WASM Commands:**
- `WasmTest`, `WasmTestBrowser`, `WebServer`

**WASI Commands:**
- `WasiTest`

## Development Guidelines

### System Requirements
- .NET 6+ for development and runtime
- macOS with full Xcode installation for Apple scenarios
- Linux/macOS/Windows for Android scenarios
- Linux for browser scenarios

### Build System
- Use `./build.sh` (Linux/macOS) or `Build.cmd` (Windows) for proper SDK setup
- Alternative: `dotnet build XHarness.slnx` (requires correct .NET version)
- Integration with Arcade SDK for .NET Foundation build standards
- Azure DevOps pipelines for CI/CD

### Key Dependencies
- **ADB (Android Debug Bridge)** - Required for Android operations
- **mlaunch** - Required for Apple platform operations
- **Helix SDK** - For cloud testing integration
- Downloaded automatically during CLI build process

### Exit Codes
XHarness uses standardized exit codes (see `src/Microsoft.DotNet.XHarness.Common/CLI/ExitCode.cs`):
- `0` - SUCCESS
- `1` - TESTS_FAILED
- `70` - TIMED_OUT
- `78` - PACKAGE_INSTALLATION_FAILURE
- `80` - APP_CRASH
- `81` - DEVICE_NOT_FOUND
- And many more specific failure scenarios

## Platform-Specific Knowledge

### Android Development
- Uses ADB for device communication
- Supports APK installation and logcat collection
- Package name-based application identification
- Emulator and physical device support

### Apple Development
- Uses mlaunch for device/simulator communication
- Supports .app bundle and .ipa installations
- Requires proper code signing and provisioning profiles
- Complex simulator runtime management
- TCP connection workarounds for test result streaming

### WASM/WASI Development
- Browser-based test execution
- WebAssembly runtime requirements
- Custom web server for test hosting

## Testing Strategy

### Unit Tests
- Located in `tests/` directory with platform-specific test projects
- Follow naming convention: `Microsoft.DotNet.XHarness.[Component].Tests`
- Use xUnit framework consistently

### Integration Tests
- Located in `tests/integration-tests/`
- E2E tests that use Helix cloud infrastructure
- Test real device/simulator scenarios
- Use `./tools/run-e2e-test.sh` for execution

### Test Runners
- Applications must include TestRunner library for `apple test` command
- TestRunner handles environmental variables and TCP connections
- Alternative: Use `apple run` for apps without TestRunner

## Common Patterns and Conventions

### Command Implementation
- Extend `XHarnessCommand<T>` abstract base class
- Implement required properties: `CommandUsage`, `CommandDescription`, `Arguments`
- Use dependency injection for logging and services
- Return appropriate `ExitCode` enum values

### Logging
- Multiple logger types: `ConsoleLogger`, `FileLogger`, `MemoryLogger`, `AggregatedLogs`, `CallbackLogger`
- Console logger is default for commands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/xharness](https://github.com/dotnet/xharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
