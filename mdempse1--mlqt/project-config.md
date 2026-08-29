---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

MLQT is a cross-platform Blazor application built with .NET 10 targeting native platforms via .NET MAUI (Android, iOS, macOS Catalyst, Windows). UI components in the Shared project are hosted within MAUI using BlazorWebView.  Only the Windows build is currently included in the project.

The MLQT UI is intended to be a users primary way to manage Modelica libraries in revision control systems and supports SVN and Git. The intention is for users to work with MLQT to review and commit changes, pull updates, create new branches and push changes to the revision control system. It also provides static analysis of Modelica code to understand the impact of changes, apply formatting rules and check code against style guidelines.

Use the CODING_GUIDELINES.md whenever generating or refactoring code.

## Solution Structure

- **MLQT.Shared** - Shared Blazor components, pages, layouts, services
- **MLQT** - .NET MAUI application
- **MLQT.Services** / **MLQT.Services.Tests** - Business logic services
- **MLQT.McpServer** / **MLQT.McpServer.Tests** - Headless Model Context Protocol (MCP) server exposing MLQT's Modelica capabilities as tools over stdio; reuses the service layer without MAUI. See `MLQT.McpServer/README.md`
- **MLQT.McpTester** - MAUI Blazor (Windows) desktop app for manually testing any stdio MCP server: connect, list tools, auto-generate parameter fields from each tool's JSON Schema, call, and view results. Uses MudBlazor + the ModelContextProtocol client SDK. See `MLQT.McpTester/README.md`
- **ModelicaParser** / **ModelicaParser.Tests** - ANTLR-based Modelica parser
- **ModelicaGraph** / **ModelicaGraph.Tests** - Directed graph for file/model relationships
- **RevisionControl** / **RevisionControl.Tests** - Git/SVN integration
- **DymolaInterface** / **DymolaInterface.Tests** - Dymola HTTP JSON-RPC interface
- **OpenModelicaInterface** / **OpenModelicaInterface.Tests** - OpenModelica ZeroMQ interface

## Build and Run Commands

```bash
# Build entire solution
dotnet build MLQT.slnx

# Run tests for a project
dotnet test MLQT.Services.Tests
dotnet test ModelicaParser.Tests
dotnet test ModelicaGraph.Tests

# Run MAUI application (Windows)
dotnet build MLQT/MLQT.csproj && dotnet run --project MLQT/MLQT.csproj
```

## Architecture Patterns

### Platform Abstraction

Services that could be used outside Blazor are in `MLQT.Services/` with interfaces in `MLQT.Services/Interfaces/`.

**Pattern for platform-specific services:**
1. Define interface in `MLQT.Services/Interfaces/`
2. Implement in `MLQT/Services/` using MAUI APIs
3. Register in `MLQT/MauiProgram.cs`

**Pattern for reusable .NET services:**
1. Define interface in `MLQT.Services/Interfaces/`
2. Implement in `MLQT.Services/`
3. Register as singleton in `MauiProgram.cs`

### Core Services

**Reusable .NET services** (in `MLQT.Services/`):

| Service | Purpose |
|---------|---------|
| **ILibraryDataService** | Manages loaded Modelica libraries, combined graph, server-side tree data |
| **IRepositoryService** | Git/SVN repository management, library discovery, VCS operations |
| **IFileMonitoringService** | FileSystemWatcher-based change detection with debouncing |
| **ICodeReviewService** | Log messages and issues from parsing/style checking |
| **IStyleCheckingService** | Background style rule checking for models with queue management |
| **IImpactAnalysisService** | Dependency impact analysis with BFS traversal |
| **IExternalResourceService** | External resource analysis, validation, and monitoring |
| **ICustomDictionaryService** | User custom word list persistence (`%LocalAppData%/MLQT/custom_dictionary.txt`) |
| **IDictionaryManagerService** | Hunspell dictionary management (bundled + imported at `%LocalAppData%/MLQT/Dictionaries/`) |
| **IModelCheckingService** | Interface for external tool checking (Dymola, OpenModelica) |
| **DymolaCheckingService** | Model checking via Dymola HTTP JSON-RPC |
| **OpenModelicaCheckingService** | Model checking via OpenModelica ZeroMQ |
| **LoggingService** | Static NLog-based logging (`%LocalAppData%/MLQT/`) |

**Platform-specific services** (in `MLQT/Services/`, use MAUI APIs):

| Service | Purpose |
|---------|---------|
| **IFilePickerService** | Native file/folder picker dialogs |
| **IPowerManagementService** | Prevents system sleep during long operations |
| **ISettingsService** | Application settings persistence (JSON, per-project) |

### Application State (AppState)

Centralized state container in `MLQT.Shared/Models/AppState.cs`:
- **Model Selection**: `ModelID`, `SelectedModelIDs`, `SelectionMode`
- **Library State**: `IsLibraryLoaded`
- **Deferred Analysis**: `IsDeferredMode`, `HasDependencyAnalysisRun`, `HasStyleCheckingRun`, `HasExternalResourcesAnalyzed`
- **Events**:
  - Model/UI: `OnChangeModel`, `OnSelectedModelsChanged`, `OnEnableMultiSelect`, `OnModelContentChanged`, `OnThemeChanged`
  - Library: `OnLibraryLoaded`, `OnLibraryCleared`
  - Settings: `OnSaveSettings`, `OnClearLogMessages`, `OnRepositorySettingsApplied`
  - VCS: `OnVcsFilesChanged`, `OnVcsModelsChanged`
  - Projects: `OnProjectSwitchStarting`, `OnProjectChanged`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mdempse1/MLQT](https://github.com/mdempse1/MLQT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
