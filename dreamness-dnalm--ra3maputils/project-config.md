---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ra3MapUtils (RA3地编伴侣) is a Windows desktop application built with WPF (.NET 8.0) to assist with Command & Conquer: Red Alert 3 map editing. It provides map management, Lua script importing, auto-updates, and integrates with the "NewWorldBuilder" map editor through plugins and nano-programs.

## Build & Development Commands

### Build the Solution
```powershell
# Build in Debug mode
dotnet build Ra3MapUtils.sln

# Build in Release mode (x64)
dotnet build Ra3MapUtils.sln -c Release
```

### Run the Application
```powershell
# Run from the main project directory
dotnet run --project Ra3MapUtils/Ra3MapUtils.csproj
```
 
### Package for Release
```powershell
# Package using Velopack (requires vpk CLI and 7z)
.\dev_tools\package.ps1
# Output: dev_tools\publish\v{version}\Ra3MapUtils-v{version}.7z
```

### Package Lua Library
```powershell
.\dev_tools\package_lualib.ps1
```

### Restore Dependencies
```powershell
dotnet restore Ra3MapUtils.sln
```

## Solution Architecture

### Project Structure

```
Ra3MapUtils/              # Main WPF application (NET8.0-Windows)
├── Views/                # XAML UI components
├── ViewModels/           # MVVM ViewModels (CommunityToolkit.Mvvm)
├── Services/             # Business services with DI
├── Models/               # Data models
├── API/                  # RESTful API controllers (embedded ASP.NET Core)
├── MCP/                  # Model Context Protocol tools for AI integration
├── Utils/                # Utilities (converters, validators, path helpers)
├── data/                 # Application data files
│   ├── plugins/          # NewWorldBuilder plugins
│   ├── nano_programs/    # C# script-based extensions
│   └── plugins_lib/      # Shared libraries for plugins
└── lib/                  # External DLLs (Dreamness.RA3.Map.*)

SharedFunctionLib/        # Shared business logic (NET4.5)
├── Business/             # Business layer (LuaImporter, Update, etc.)
├── DAO/                  # Data access layer (SQLite)
└── Models/               # Shared data models

UtilCoreLib/              # RA3 map utilities (NET4.5)
├── mapFileHelper/        # Map file operations
├── mapScriptHelper/      # Lua script helpers
├── mapstrFileHelper/     # Map string operations
└── mapXmlOperator/       # XML parsing

KnowledgeBaseLib/         # Full-text search database (NET8.0)
└── DAO/                  # SQLite FTS5 with Jieba tokenizer

KnowledgeBaseCli/         # CLI for knowledge base (NET8.0)
test_field/               # Testing playground
```

### Dependency Graph

```
Ra3MapUtils
├── KnowledgeBaseLib
├── SharedFunctionLib
│   └── UtilCoreLib
└── UtilCoreLib

KnowledgeBaseCli
└── KnowledgeBaseLib
```

### Key Architectural Patterns

**MVVM Pattern**: Uses `CommunityToolkit.Mvvm` with:
- `ObservableObject` base class for ViewModels
- `[ObservableProperty]` for bindable properties
- `RelayCommand` for commands
- `WeakReferenceMessenger` for inter-component messaging

**Dependency Injection**: Configured in `App.xaml.cs`:
- Services registered in `ConfigureServices()` method
- Uses Microsoft.Extensions.DependencyInjection
- Singletons for pages, view models, and most services

**Layered Architecture**:
```
UI (XAML + ViewModels)
  ↓
Services Layer (Ra3MapUtils/Services/Impl/)
  ↓
Business Layer (SharedFunctionLib/Business/)
  ↓
DAO Layer (SharedFunctionLib/DAO/)
  ↓
SQLite Database
```

**Embedded Web Server**:
- ASP.NET Core hosted on port 30033
- Provides RESTful APIs and Swagger UI
- MCP (Model Context Protocol) server for AI integration
- Started in `App.xaml.cs` using `WebApplication.CreateBuilder()`

## Extensibility Systems

### 1. NewWorldBuilder Plugins

Plugins extend the NewWorldBuilder map editor with custom functionality. They are managed through `INewWorldBuilderPluginService`.

**Plugin Structure**:
```
data/plugins/RA3MapUtil_LuaImporter/
├── plugin_meta.json      # Metadata (name, version, file mappings)
├── Main.cs               # Plugin implementation
└── readme.txt            # Documentation
```

**Plugin Metadata** (`plugin_meta.json`):
```json
{
  "PluginName": "RA3MapUtil_LuaImporter",
  "PluginVersion": "v1.1.4",
  "RequireFileDictionary": {
    "source_file.cs": "target_file.cs"
  }
}
```

**Installation**: Plugins are auto-installed to NewWorldBuilder's script directory when detected.

### 2. Nano-Programs

Nano-programs are C# script-based micro-utilities that extend Ra3MapUtils functionality. They can be executed from the UI or via API.

**Locations**:
- Built-in: `Ra3MapUtils/data/nano_programs/`
- User: `%AppData%/Ra3MapUtils/nano_programs/user/`
- Store: `%AppData%/Ra3MapUtils/nano_programs/store/`

**Nano-Program Structure**:
```
UUID_Generator/
├── info.json             # Metadata (ID, Name, Description, Author)
└── Main.cs               # Executable C# code with Run() entry point
```

**Metadata** (`info.json`):
```json
{
  "ID": "uuid-generator",
  "Name": "UUID Generator",
  "Description": "Generates UUIDs for map objects",
  "Author": "dreamness"
}
```

**Service**: Managed by `INanoProgramService` - handles discovery, execution order, enable/disable state.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dreamness-dnalm/Ra3MapUtils](https://github.com/dreamness-dnalm/Ra3MapUtils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
