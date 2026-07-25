---
trigger: always_on
description: PropertyTools is a collection of custom controls for WPF applications that provide enhanced property editing, data grid, and tree list capabilities. The project targets .NET 4.6.2 and .NET 8 - Windows platforms.
---

# GitHub Copilot Instructions for PropertyTools

## Project Overview

PropertyTools is a collection of custom controls for WPF applications that provide enhanced property editing, data grid, and tree list capabilities. The project targets .NET 4.6.2 and .NET 8 - Windows platforms.

## Key Components

- **PropertyTools.Wpf**: Main WPF control library
- **PropertyTools**: Core library
- **PropertyTools.Wpf.Extended.Toolkit**: Extended toolkit controls
- **Examples**: Various demo applications showcasing the controls
- **PropertyTools.Wpf.Tests**: NUnit test suite

## Development Guidelines

### Code Style and Standards

1. **Namespace Conventions**
   - Use `PropertyTools.Wpf` for WPF-specific code
   - Use `PropertyTools` for core/shared code
   - Example-specific code should use appropriate namespace (e.g., `ExampleLibrary`)

2. **Copyright Headers**
   - All source files must include the standard copyright header:
     ```csharp
     // --------------------------------------------------------------------------------------------------------------------
     // <copyright file="FileName.cs" company="PropertyTools">
     //   Copyright (c) 2014 PropertyTools contributors
     // </copyright>
     // --------------------------------------------------------------------------------------------------------------------
     ```

3. **Naming Conventions**
   
   - **Classes and Interfaces**
     - Use PascalCase for class names
     - Test classes: `{ClassUnderTest}Tests` (e.g., `ColorHelperTests`, `NaturalStringComparerTests`)
     - Helper classes: `{Purpose}Helper` (e.g., `ColorHelper`, `TypeHelper`)
     - Converters: `{Type}Converter` or `{Type}To{Type}Converter` (e.g., `EnumValuesConverter`, `BrushToColorConverter`)
     - Controls: Descriptive PascalCase names (e.g., `ColorPicker`, `EditableTextBlock`)
   
   - **Methods**
     - Use PascalCase for method names
     - Use descriptive, action-oriented names (e.g., `ChangeAlpha`, `Interpolate`, `Parse`)
   
   - **Unit Test Methods**
     - **Follow the pattern: `MethodName_StateUnderTest_ExpectedBehavior`**
     - Examples:
       - `Parse_Days_ReturnsCorrectValue`
       - `ChangeAlpha_ValidColor_ReturnsCorrectValue`
       - `HexToColor_InvalidColors_ReturnsUndefined`
     - Reference: [Roy Osherove's naming standards](https://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html)
   
   - **Fields**
     - Private fields: Use camelCase (e.g., `colorPickerPanel`, `selectedColor`)
     - Private const fields: Use PascalCase (e.g., `PartColorPickerPanel`)
     - Static readonly fields: Use PascalCase (e.g., `SelectedColorProperty` for dependency properties)
   
   - **Properties**
     - Use PascalCase for property names (e.g., `SelectedColor`, `UndefinedColor`)
   
   - **Parameters and Local Variables**
     - Use camelCase (e.g., `color`, `alpha`, `targetType`)

4. **XML Documentation**
   - Use XML documentation comments (`///`) for public APIs
   - Include `<summary>`, `<param>`, and `<returns>` tags as appropriate
   - Example:
     ```csharp
     /// <summary>
     /// Change the alpha value of a color.
     /// </summary>
     /// <param name="c">The source color.</param>
     /// <param name="alpha">The new alpha value.</param>
     /// <returns>The new color.</returns>
     public static Color ChangeAlpha(this Color c, byte alpha)
     ```

5. **Code Formatting**
   - Follow existing code style in the repository
   - Use meaningful variable and method names
   - Keep methods focused and concise
   - Use `this.` prefix for instance members in WPF controls

### Building and Testing

1. **Solution File**: `Source/PropertyTools.sln`
2. **Build Command**: Use Visual Studio or `dotnet build` for the solution
3. **Test Framework**: NUnit
4. **Test Project**: `PropertyTools.Wpf.Tests`

### Target Frameworks

- .NET Framework 4.6.2
- .NET 8 - Windows

Ensure changes are compatible with both target frameworks unless explicitly targeting a specific version.

### Git and Version Control

1. **Branching**: The main development branch is `develop`
2. **Versioning**: Uses GitVersion for semantic versioning
3. **Commits**: Write clear, descriptive commit messages

## Required Actions for Pull Requests

### Changelog Updates (CRITICAL)

**Every pull request created by GitHub Copilot MUST include an update to the `CHANGELOG.md` file.**

1. **Location**: Update the `## Unreleased` section at the top of `CHANGELOG.md`

2. **Categories**: Add entries under the appropriate category:
   - `### Added` - New features or capabilities
   - `### Fixed` - Bug fixes
   - `### Changed` - Changes to existing functionality
   - `### Removed` - Removed features or capabilities

3. **Format**: Use the following format:
   ```markdown
   - Brief description of change #123
   ```
   Here "#123" is an example. Make sure to use the reference for the issue, and not "#TBD".

4. **Example**:
   ```markdown
   ## Unreleased
   
   ### Added
   - PropertyGrid: Support for custom validators #123
   
   ### Fixed
   - DataGrid: Fix null reference exception when sorting empty collection #456
   ```

5. **Guidelines**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PropertyTools/PropertyTools](https://github.com/PropertyTools/PropertyTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
