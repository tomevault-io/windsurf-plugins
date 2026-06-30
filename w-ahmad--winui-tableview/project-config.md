---
trigger: always_on
description: **WinUI.TableView** is a lightweight and fast data grid control for WinUI 3 apps with support for the Uno Platform. It's derived from `ListView` and provides an Excel-like data table experience with features like column filtering, sorting, editing, and data export.
---

# GitHub Copilot Instructions for WinUI.TableView

## Project Overview

**WinUI.TableView** is a lightweight and fast data grid control for WinUI 3 apps with support for the Uno Platform. It's derived from `ListView` and provides an Excel-like data table experience with features like column filtering, sorting, editing, and data export.

- **Repository**: https://github.com/w-ahmad/WinUI.TableView
- **Documentation**: https://w-ahmad.github.io/WinUI.TableView/
- **NuGet Package**: WinUI.TableView
- **License**: MIT

## Technology Stack

- **Primary Platform**: WinUI 3 (Windows App SDK)
- **Cross-Platform Support**: Uno Platform (WASM, Desktop)
- **Target Frameworks**: 
  - .NET 8.0 and .NET 9.0 (Uno Platform targets)
  - .NET 8.0-windows10.0.19041.0 and .NET 9.0-windows10.0.19041.0 (WinUI 3 targets)
- **Languages**: C# with XAML
- **Dependencies**:
  - Microsoft.WindowsAppSDK (for WinUI targets)
  - Uno.WinUI (for non-Windows targets)
  - CommunityToolkit.WinUI.Behaviors

## Repository Structure

```
/
├── src/                          # Main library source code
│   ├── Columns/                  # Column type implementations
│   ├── Controls/                 # Custom control implementations
│   ├── Converters/               # XAML value converters
│   ├── EventArgs/                # Custom event argument classes
│   ├── Extensions/               # Extension methods
│   ├── Helpers/                  # Helper classes and utilities
│   ├── ItemsSource/              # Data source handling
│   ├── Primitives/               # Base/primitive control classes
│   ├── Strings/                  # Localization resources
│   ├── Themes/                   # XAML styles and control templates
│   ├── TableView.cs              # Main TableView control
│   └── WinUI.TableView.csproj    # Project file
├── tests/                        # Unit tests
│   └── WinUI.TableView.Tests.csproj
├── docs/                         # Documentation (DocFX)
├── .github/                      # GitHub configuration
│   ├── workflows/                # CI/CD workflows
│   └── ISSUE_TEMPLATE/           # Issue templates
├── README.md                     # Project readme
├── CONTRIBUTING.md               # Contribution guidelines
└── WinUI.TableView.slnx          # Solution file
```

## Build and Test Instructions

### Building the Project

**Prerequisites**:
- Visual Studio 2022 or later
- Windows 10 SDK (10.0.19041.0 or later)
- .NET 8.0 and/or .NET 9.0 SDK

**Build Command**:
```bash
msbuild /restore /t:Build,Pack src/WinUI.TableView.csproj /p:Configuration=Release
```

**Build Outputs**:
- NuGet packages: `artifacts/NuGet/Release/`
- Documentation is built automatically on commits to docs/ folder

### Running Tests

**Build Tests**:
```bash
msbuild /restore /t:Build tests/WinUI.TableView.Tests.csproj /p:Platform=x64 /p:Configuration=Release /p:OutputPath=build
```

**Run Tests**:
```bash
vstest.console.exe tests\build\WinUI.TableView.Tests.build.appxrecipe --logger:"console;verbosity=normal" /InIsolation
```

**Note**: Tests are WinUI 3 app tests and require Windows with visual UI. They run on x64 platform only.

## Coding Standards and Conventions

### Code Style

This project uses `.editorconfig` for consistent code formatting. Key conventions:

- **Indentation**: 4 spaces
- **Line Endings**: CRLF (Windows style)
- **Namespace Style**: File-scoped namespaces preferred
- **Nullable Reference Types**: Enabled (`<Nullable>enable</Nullable>`)
- **Private Fields**: Use underscore prefix (e.g., `_fieldName`)
- **Interfaces**: Prefix with `I` (e.g., `ITableViewColumn`)
- **Naming**: PascalCase for public members, camelCase with underscore for private fields
- **Using Directives**: Place outside namespace, no separation of System directives
- **File Header**: Not required

### XML Documentation

- **All public types and members must have XML documentation comments** (enforced as error via CS1591)
- Use `<summary>`, `<param>`, `<returns>`, and other standard XML doc tags
- Generate documentation file is enabled for the library project

### Platform-Specific Code

The project supports both WinUI 3 and Uno Platform. When writing platform-specific code:

- Use conditional compilation based on target framework
- WinUI-specific code: Use `#if WINDOWS` or check `$([MSBuild]::GetTargetPlatformIdentifier($(TargetFramework))) == 'windows'`
- Uno Platform specific: Check for non-Windows targets
- Test changes on both WinUI 3 and Uno Platform targets when possible

### XAML Conventions

- Use WinUI 3 XAML syntax
- Control templates and styles are in `src/Themes/` folder
- Generic.xaml serves as the main resource dictionary
- Use resource keys for reusable styles and templates

## Key Features and Components

### Core Components

1. **TableView**: Main control derived from ListView
2. **TableViewColumn Types**:
   - TableViewTextColumn
   - TableViewCheckBoxColumn
   - TableViewComboBoxColumn
   - TableViewNumberColumn
   - TableViewToggleSwitchColumn
   - TableViewTemplateColumn
   - TableViewTimeColumn
   - TableViewDateColumn


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [w-ahmad/WinUI.TableView](https://github.com/w-ahmad/WinUI.TableView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
