---
trigger: always_on
description: InkCanvasForClass-Remastered (ICC-Re) is a .NET 8 WPF classroom ink canvas/whiteboard application optimized for interactive displays and teaching environments. This is a Windows-only desktop application that provides drawing, annotation, and presentation tools for educational settings.
---

# InkCanvasForClass-Remastered Development Instructions

InkCanvasForClass-Remastered (ICC-Re) is a .NET 8 WPF classroom ink canvas/whiteboard application optimized for interactive displays and teaching environments. This is a Windows-only desktop application that provides drawing, annotation, and presentation tools for educational settings.

**ALWAYS follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Environment Setup
- **CRITICAL**: This project ONLY builds and runs on Windows. Do NOT attempt to build on Linux/macOS.
- Install [.NET 8 Desktop Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) from Microsoft
- Use Visual Studio 2022 or later (recommended) or VS Code with C# extension
- Windows 10/11 required for full WPF functionality and UI testing

### Build Commands (Windows Only)
- **Navigate to Project**: `cd InkCanvasForClass-Remastered` (from repo root) or work from solution root
- **Package Restore**: `dotnet restore` (takes ~10 seconds)
- **Debug Build**: `dotnet build` (takes ~15-30 seconds, NEVER CANCEL - set timeout to 60+ seconds)
- **Release Build**: `dotnet build -c Release` (takes ~20-40 seconds, NEVER CANCEL - set timeout to 60+ seconds)
- **Publish Single File**: `dotnet publish -c Release -f net8.0-windows -r win-x64 --self-contained false -o publish -p:PublishSingleFile=true` (takes ~30-60 seconds, NEVER CANCEL - set timeout to 120+ seconds)

### Running the Application (Windows Only)
- **Debug Mode**: `dotnet run` from project directory
- **From Build Output**: Navigate to `bin/Debug/net8.0-windows/` and run `InkCanvasForClass-Remastered.exe`
- **From Published Output**: Run `publish/InkCanvasForClass-Remastered.exe`

### Validation Requirements
- **CRITICAL**: After any UI changes, ALWAYS run the application and test basic functionality
- **NEVER rely on build success alone** - the app must be tested with user scenarios
- **ALWAYS test these core scenarios after changes**:
  1. Application startup and main window display
  2. Drawing with pen/mouse on the canvas
  3. Switching between pen, eraser, and selection tools
  4. Color picker and brush size adjustments
  5. Clearing the canvas
  6. Settings dialog accessibility and basic settings changes

## Repository Structure

### Key Directories
- `InkCanvasForClass-Remastered/` - Main project directory
  - `MainWindow.xaml/.xaml.cs` - Primary application window
  - `App.xaml/.xaml.cs` - Application entry point and dependency injection setup
  - `Services/` - Core business logic and external integrations
    - `SettingsService.cs` - Application configuration management
    - `PowerPointService.cs` - Microsoft PowerPoint integration
    - `FileFolderService.cs` - File system operations and logging
    - `Logging/` - Custom logging implementation
  - `ViewModels/` - MVVM pattern view models
  - `Windows/` - Additional UI windows (dialogs, tools)
    - `RandWindow.xaml` - Random name picker tool
    - `CountdownTimerWindow.xaml` - Classroom timer
    - `OperatingGuideWindow.xaml` - Help and tutorials
  - `Controls/` - Custom WPF controls and user controls
  - `Helpers/` - Utility classes
    - `TimeMachine.cs` - Undo/redo functionality for ink strokes
  - `Models/` - Data models and settings classes
    - `Settings.cs` - Application settings model
  - `Resources/` - Icons, cursors, styles, and other assets
  - `Converters/` - XAML value converters
  - `Enums/` - Application enumerations

### Important Files
- `InkCanvasForClass-Remastered.csproj` - Project configuration with dependencies
- `Settings.XamlStyler` - XAML formatting configuration
- `.github/workflows/build.yml` - CI/CD pipeline (Windows-only)
- `README.md` - Project documentation (Chinese)
- `CHANGELOG.md` - Version history and changes
- `Manual.md` - User manual and feature documentation

## Development Workflow

### Making Changes
- **ALWAYS build and test on Windows** - Linux/macOS builds will fail
- **Code Style**: Follow existing C# and XAML conventions in the codebase
- **XAML Formatting**: Use XamlStyler configuration in `Settings.XamlStyler`
- **Logging**: Use the custom logging service through dependency injection
- **Settings**: Access application settings via `SettingsService` singleton

### Testing Changes
- **NEVER skip manual testing** - this project has no automated tests
- **UI Changes**: Start the app and verify visual elements render correctly
- **Drawing Features**: Test pen, eraser, shapes, and color changes
- **File Operations**: Test save/load functionality if modified
- **PowerPoint Integration**: Test with actual PowerPoint if you modify `PowerPointService.cs`

### Common Modification Areas
- **UI Adjustments**: Modify `.xaml` files for layout and appearance
- **Drawing Logic**: Update `MainWindow_cs/` files and `Helpers/TimeMachine.cs`
- **Settings/Preferences**: Edit `Models/Settings.cs` and `Services/SettingsService.cs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiuYan-xwx/InkCanvasForClass-Remastered](https://github.com/LiuYan-xwx/InkCanvasForClass-Remastered) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
