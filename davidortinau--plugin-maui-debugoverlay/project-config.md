---
trigger: always_on
description: Plugin.Maui.DebugOverlay is a .NET MAUI plugin that provides debug overlay functionality including a visual debug ribbon and comprehensive visual tree dumping capabilities for MAUI applications.
---

# Plugin.Maui.DebugOverlay

Plugin.Maui.DebugOverlay is a .NET MAUI plugin that provides debug overlay functionality including a visual debug ribbon and comprehensive visual tree dumping capabilities for MAUI applications.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Repository Setup and Requirements
- Install .NET 9.0 or later SDK
- Install .NET MAUI workload: `dotnet workload install maui`
- CRITICAL: Repository currently targets .NET 10 (not yet released) causing build issues
- Repository has missing MauiVersion property definition in build configuration

### Build Process - CURRENT LIMITATIONS
- **BROKEN BUILD**: The repository currently cannot be built due to configuration issues:
  - Main library targets `net10.0-android;net10.0-ios;net10.0-maccatalyst;net10.0` which are not available
  - Sample app targets `net9.0-android;net9.0-ios;net9.0-maccatalyst` which require MAUI workload
  - Missing `$(MauiVersion)` property definition causes package reference failures
- **Required Fix**: Update library target frameworks to `net9.0-*` to match sample and stable SDK
- **MauiVersion Property**: Add to Directory.Build.props or define in project files (e.g., `<MauiVersion>9.0.0</MauiVersion>`)
- DO NOT attempt to build until these configuration issues are resolved
- Expected build command when working: `dotnet build src/Plugin.Maui.DebugOverlay.sln -c Debug`
- NEVER CANCEL: MAUI builds typically take 5-15 minutes. Set timeout to 30+ minutes.
- **CI/CD Note**: You should be able to build from Linux for net9.0-android in CI. Other platforms may not work in CI environment.

### Temporary Build Workaround (If Needed)
If you need to test compilation of individual files:
1. Create temporary test project with net9.0 target
2. Copy plugin source files to test project  
3. Add MAUI package references manually
4. Test compilation and basic functionality

### Project Structure
- **Main Library**: `src/Plugin.Maui.DebugOverlay/Plugin.Maui.DebugOverlay.csproj`
- **Sample Application**: `samples/Plugin.Maui.DebugOverlay.Sample/Plugin.Maui.DebugOverlay.Sample.csproj`
- **Core Components**:
  - `DebugOverlay.cs` - Main overlay window management
  - `DebugRibbonElement.cs` - Corner ribbon UI element  
  - `DebugOverlayPanel.cs` - Interactive debug panel with tree view
  - `VisualTreeDumpService.cs` - Visual tree analysis and dumping
  - `MauiProgramExtensions.cs` - Extension methods for MAUI app setup

### Validation Scenarios
When making changes, ALWAYS test these complete scenarios:

#### Basic Integration Test
1. Add plugin to a MAUI app via `.UseDebugRibbon(Colors.Orange)` in `MauiProgram.cs`
2. Run app in DEBUG mode on at least one platform
3. Verify debug ribbon appears in top-left corner with "DEBUG" text
4. Tap ribbon to show/hide debug panel - NEVER CANCEL: UI interaction may take 10-30 seconds

#### Visual Tree Dump Test  
1. Open debug panel by tapping ribbon
2. Tap "🔍 Dump Visual Tree" button
3. Verify output appears in debug console
4. Check that dump files are created in `{AppDataDirectory}/debug-dumps/`
5. Validate tree structure shows proper hierarchy with indentation

#### Shell Hierarchy Test
1. In debug panel, tap "🧭 Dump Shell Hierarchy" button  
2. Verify Shell navigation structure is dumped to console and file
3. Confirm both latest and timestamped files are created

#### Platform Validation
- Test ribbon visibility on iOS, Android, Windows, macOS
- Verify graphics rendering works consistently across platforms
- Confirm debug-only exclusion in Release builds

## Configuration and Usage

### Plugin Integration
```csharp
// In MauiProgram.cs - typical setup
public static MauiApp CreateMauiApp()
{
    var builder = MauiApp.CreateBuilder();
    builder
        .UseMauiApp<App>()
        .UseDebugRibbon(Colors.Orange)  // Enable debug overlay
        .ConfigureFonts(fonts =>
        {
            fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
        });

    return builder.Build();
}

// Custom color examples
builder.UseDebugRibbon(Color.FromArgb("#FF6B35"));      // Specific hex color
builder.UseDebugRibbon(Colors.Red);                     // Named color
builder.UseDebugRibbon(Color.FromArgb("#FF3300"));      // Sample app uses this
```

### Extension Method Details
- **Method**: `UseDebugRibbon(this MauiAppBuilder builder, Color ribbonColor = null)`
- **Default Color**: `Colors.MediumPurple` if no color specified
- **Debug Only**: Automatically wrapped in `#if DEBUG` preprocessor directive
- **Handler Registration**: Uses `WindowHandler.Mapper.AppendToMapping` to add overlay to windows

### Key Features to Test
- **Debug Ribbon**: Corner ribbon showing "DEBUG" or MAUI version
- **Interactive Panel**: Tap ribbon to show/hide debug tools
- **Visual Tree Dump**: Comprehensive layout and handler information
- **Shell Hierarchy**: Navigation structure analysis
- **File Output**: Dumps saved to app data directory
- **Cross-Platform**: Consistent behavior across all MAUI platforms

### Development Commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidortinau/Plugin.Maui.DebugOverlay](https://github.com/davidortinau/Plugin.Maui.DebugOverlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
