---
trigger: always_on
description: > **Audience**: These instructions are for AI coding assistants, not for human developers.
---

# Matroska Batch Flow - Copilot Instructions

> **Audience**: These instructions are for AI coding assistants, not for human developers.  
> **Human developers**: See [README.md](../README.md) for setup and contribution guidelines.

> **Last Updated**: 2026-04-28  
> **Next Review**: When architecture changes, new patterns are introduced, or build commands change
> 
> **Maintenance**: When making significant code changes, update this file and Serena memories to keep them in sync with the codebase.

## Build, Test, and Run Commands

### Quick Reference
```powershell
# Most common commands (run from repository root)
dotnet build MatroskaBatchFlow.sln         # Build entire solution
dotnet test                                # Run all tests
dotnet run --project src/MatroskaBatchFlow.Uno/MatroskaBatchFlow.Uno.csproj -f net10.0-windows10.0.19041
```

### Building
```powershell
# Build the WinAppSDK target (Recommended)
dotnet build src/MatroskaBatchFlow.Uno/MatroskaBatchFlow.Uno.csproj -f net10.0-windows10.0.19041

# Build the Skia Desktop target (Experimental)
dotnet build src/MatroskaBatchFlow.Uno/MatroskaBatchFlow.Uno.csproj -f net10.0-desktop

# Build entire solution
dotnet build MatroskaBatchFlow.sln
```

### Running
```powershell
# Run the application (automatically builds if needed)
dotnet run --project src/MatroskaBatchFlow.Uno/MatroskaBatchFlow.Uno.csproj -f net10.0-windows10.0.19041
```

### Testing
```powershell
# Run all tests
dotnet test

# Run a specific test project
dotnet test tests/MatroskaBatchFlow.Core.UnitTests/MatroskaBatchFlow.Core.UnitTests.csproj
dotnet test tests/MatroskaBatchFlow.Uno.UnitTests/MatroskaBatchFlow.Uno.UnitTests.csproj
dotnet test tests/MatroskaBatchFlow.Uno.IntegrationTests/MatroskaBatchFlow.Uno.IntegrationTests.csproj

# Run a single test by filtering
dotnet test --filter "FullyQualifiedName~BatchConfigurationTests"
```

### Publishing
```powershell
# Use the Publish-Application.ps1 script for various distribution modes
.\Publish-Application.ps1 -BuildType SingleFile    # Self-contained single executable
.\Publish-Application.ps1 -BuildType MultiFile     # Multi-file deployment
.\Publish-Application.ps1 -BuildType Store         # MSIX for Microsoft Store
.\Publish-Application.ps1 -BuildType SelfSigned    # MSIX with self-signed certificate
```

## Architecture Overview

### Project Structure
- **MatroskaBatchFlow.Core** - Cross-platform core library containing business logic, file validation, processing engines, and external tool orchestration
- **MatroskaBatchFlow.Uno** - WinUI 3/Uno Platform GUI application (supports WinAppSDK and Skia Desktop)
- **MatroskaBatchFlow.Console** - Reserved for future CLI utility

### Core Components

#### Service Layer
- **File Processing Pipeline**: `IFileScanner` → `IFileValidationEngine` → `IFileProcessingOrchestrator` → `IMkvToolExecutor`
  - `FileScanner`: Discovers and scans `.mkv` files using MediaInfo for metadata extraction
  - `FileValidationEngine`: Validates file consistency using pluggable validation rules
  - `FileProcessingOrchestrator`: Coordinates batch file processing with cancellation support
  - `MkvPropeditExecutor`: Executes `mkvpropedit` commands via `ProcessRunner`

- **Validation System**: Rule-based validation with three strictness modes (Strict, Lenient, Custom)
  - Rules implement `IFileValidationRule` (e.g., `TrackCountConsistencyRule`, `LanguageConsistencyRule`)
  - Results are severity-based: Error, Warning, Information
  - Cross-file property comparison rules (e.g., `DefaultFlagConsistencyRule`, `ForcedFlagConsistencyRule`, `LanguageConsistencyRule`) use `RollingReferenceComparer` to handle batches where files have different track counts: for each track position, the first file that has that position serves as the rolling reference for all subsequent files

- **Configuration Management**: `IBatchConfiguration` tracks file-level settings and global per-slot `TrackIntent` collections
  - `IBatchTrackConfigurationInitializer`: Expands global track intent collections from scanned files
  - `ITrackIntentFactory`: Creates `TrackIntent` instances from scanned track data

#### Presentation Layer (MVVM + Messaging)
- **Architecture**: MVVM with CommunityToolkit.Mvvm (source generators for commands/properties)
- **Messaging**: Uses `WeakReferenceMessenger` for loosely-coupled communication between ViewModels and Views
  - Message types in `Messages/` folder: `DialogMessage`, `ExceptionDialogMessage`, `ShowValidationDetailsMessage`, etc.
- **Navigation**: `INavigationService` handles page navigation with `INavigationAware` interface for lifecycle hooks
- **Services**: Interface-first design with contracts in `Contracts/Services/` and implementations in `Services/`

#### Key Services
- `IFileListAdapter`: Manages the collection of scanned files with add/remove operations
- `IValidationSettingsService`: Manages validation strictness modes and track-level severity settings
- `IThemeApplierService`: Applies Light/Dark/System themes
- `IUIPreferencesService`: Persists user preferences via `WritableJsonSettings<T>`

### External Dependencies
- **MediaInfo**: Media file analysis (embedded library via P/Invoke)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TimGels/Matroska-Batch-Flow](https://github.com/TimGels/Matroska-Batch-Flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
