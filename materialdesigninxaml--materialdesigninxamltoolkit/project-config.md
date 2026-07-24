---
trigger: always_on
description: The MaterialDesignInXamlToolkit is a **theme library** for WPF applications that provides Material Design themes and styling. This is NOT a control library - the focus is on theming existing WPF controls and providing custom controls only when necessary to support Google's Material Design specifications.
---

# Copilot Instructions for MaterialDesignInXamlToolkit

## Repository Overview

The MaterialDesignInXamlToolkit is a **theme library** for WPF applications that provides Material Design themes and styling. This is NOT a control library - the focus is on theming existing WPF controls and providing custom controls only when necessary to support Google's Material Design specifications.

### Key Principles
- Theme library, not control library
- Styling existing WPF controls to match Material Design
- Custom controls only for Google-specified components that don't exist in WPF
- No application or business logic belongs in this library
- Provide base tools and springboard for developers to create their own controls

## Architecture and Structure

### Core Projects
- **`MaterialDesignThemes.Wpf`** - Main theming library with styles, templates, and controls
- **`MaterialDesignColors.Wpf`** - Color palette and theme management
- **`MaterialDesignThemes.MahApps`** - Integration with MahApps.Metro
- **`MainDemo.Wpf`** - Primary demonstration application
- **`MaterialDesign3.Demo.Wpf`** - Material Design 3 demonstration
- **`MaterialDesignToolkit.ResourceGeneration`** - Build-time resource generation tools

### Key Technologies
- **WPF (Windows Presentation Foundation)** - UI framework
- **XAML** - Markup for UI definitions and styles  
- **Material Design** - Google's design system implementation
- **.NET 8** and **.NET Framework 4.7.2** - Target frameworks for the library
- **.NET 9 SDK** - Required for building (as specified in `global.json`)
- **C# 12.0** - Programming language
- **PowerShell** - Build automation scripts

## Development Environment

### Requirements
- **Windows** - Required for WPF development and compilation
- **.NET 9 SDK** - As specified in `global.json` (note: projects target .NET 8 and .NET Framework 4.7.2)
- **Visual Studio 2022** or **Visual Studio Code** with C# extension
- **PowerShell** - For build scripts

### Build and Test
```powershell
# Restore dependencies
dotnet restore MaterialDesignToolkit.Full.slnx

# Build (requires Windows)
dotnet build MaterialDesignToolkit.Full.slnx --configuration Release --no-restore -p:Platform="Any CPU" -p:TreatWarningsAsErrors=True

# Run tests
dotnet test MaterialDesignToolkit.Full.slnx --configuration Release --no-build

# Build NuGet packages
.\build\BuildNugets.ps1 -MDIXVersion "x.x.x" -MDIXColorsVersion "x.x.x" -MDIXMahAppsVersion "x.x.x"
```

## Code Style and Conventions

### General Guidelines
- Follow standard Visual Studio settings with ReSharper suggestions
- Use .editorconfig settings (4-space indents for C#, 2-space for XAML/XML)
- Allman brace style (`csharp_new_line_before_open_brace = all`)
- No `this.` qualification unless necessary
- Prefer explicit types over `var` for built-in types
- Use PascalCase for public members, interfaces start with `I`

### C# Conventions
```csharp
// Preferred dependency property pattern
public static readonly DependencyProperty MyPropertyProperty =
    DependencyProperty.Register("MyProperty", typeof(string), typeof(MyControl), 
        new UIPropertyMetadata("DefaultValue", OnMyPropertyChanged));

public string MyProperty
{
    get => (string)GetValue(MyPropertyProperty);
    set => SetValue(MyPropertyProperty, value);
}

private static void OnMyPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
{
    var control = (MyControl)d;
    // Handle property change
}
```

### XAML Style Guidelines
- Use XamlStyler settings from `Settings.XamlStyler`
- 2-space indentation for XAML
- Keep first attribute on same line as element
- Order attributes according to defined groups
- Use `{StaticResource}` over `{DynamicResource}` where possible
- Follow resource naming: `MaterialDesign.Brush.Primary.Light`

## WPF and Material Design Context

### Theme Architecture
- **Base Themes**: Light and Dark variants
- **Color System**: Primary, Secondary, Surface, Background colors with variants
- **Elevation**: Shadow and overlay systems for depth
- **Typography**: Material Design text styles
- **Motion**: Transitions and animations

### Common Patterns
```csharp
// Theme modification pattern
private static void ModifyTheme(Action<Theme> modificationAction)
{
    var paletteHelper = new PaletteHelper();
    Theme theme = paletteHelper.GetTheme();
    
    modificationAction?.Invoke(theme);
    
    paletteHelper.SetTheme(theme);
}

// Color adjustment usage
theme.ColorAdjustment = new ColorAdjustment
{
    DesiredContrastRatio = desiredRatio,
    Contrast = contrastValue,
    Colors = colorSelection
};
```

### Resource Dictionary Patterns
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:materialDesign="http://materialdesigninxaml.net/winfx/xaml/themes">
  
  <ResourceDictionary.MergedDictionaries>
    <materialDesign:BundledTheme BaseTheme="Light" PrimaryColor="DeepPurple" SecondaryColor="Lime" />

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaterialDesignInXAML/MaterialDesignInXamlToolkit](https://github.com/MaterialDesignInXAML/MaterialDesignInXamlToolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
