---
trigger: always_on
description: ﻿# PanoramicData.Blazor - Copilot Instructions
---

﻿# PanoramicData.Blazor - Copilot Instructions

## Overview

**PanoramicData.Blazor** is a comprehensive Blazor component library providing rich UI controls for data visualization, forms, and user interaction.

**Repository**: .NET 10 solution with Blazor WebAssembly, Razor Pages demo applications, and a reusable component library.

---

## 🚨 CRITICAL BUILD RULE

**Build individual projects when making component changes, build the full solution only for verification.**

```sh
# ✅ CORRECT: Build only the component library
dotnet build PanoramicData.Blazor/PanoramicData.Blazor.csproj

# ✅ CORRECT: Run the demo to test changes
dotnet run --project PanoramicData.Blazor.Demo/PanoramicData.Blazor.Demo.csproj

# ✅ CORRECT: Run tests
dotnet test PanoramicData.Blazor.Test/PanoramicData.Blazor.Test.csproj

# ⚠️ USE SPARINGLY: Full solution build
dotnet build PanoramicData.Blazor.slnx
```

---

## 📚 Project Structure

### Core Projects

| Project | Purpose | Technology |
|---------|---------|------------|
| **PanoramicData.Blazor** | Main component library | Blazor Components, .NET 9 |
| **PanoramicData.Blazor.Demo** | Demo application (Razor Pages) | Blazor Server, Razor Pages |
| **PanoramicData.Blazor.Web** | Additional demo/web host | Blazor Server |
| **PanoramicData.Blazor.WebAssembly** | WebAssembly demo (Client/Server) | Blazor WASM |
| **PanoramicData.Blazor.Test** | Unit tests | xUnit, bUnit |

### Key Directories

```
PanoramicData.Blazor/
├── Components/          # Individual Blazor components
│   ├── PDTimeline.razor(.cs/.css/.js)
│   ├── PDTable.razor
│   ├── PDTree.razor
│   └── ... (40+ components)
├── Models/             # Data models and DTOs
├── Services/           # Shared services
└── wwwroot/           # Static assets (JS, CSS)

PanoramicData.Blazor.Demo/
├── Pages/             # Demo pages for each component
│   ├── PDTimelinePage.razor
│   └── ...
└── Data/              # Demo data generators
```

---

## 🎯 Component Development Patterns

### Component Structure

Each component typically consists of:
- **`.razor`** - Markup and component structure
- **`.razor.cs`** - C# code-behind (partial class)
- **`.razor.css`** - Scoped CSS styles
- **`.razor.js`** - JavaScript interop (optional)

### Example: PDTimeline Component Files
```
PDTimeline.razor       # SVG markup, event handlers
PDTimeline.razor.cs    # Business logic, state management
PDTimeline.razor.css   # Component-specific styles
PDTimeline.razor.js    # JS interop for DOM measurements
```

### Component Naming Convention
- Components: `PD` prefix (e.g., `PDTimeline`, `PDTable`, `PDTree`)
- Demo pages: Component name + `Page` (e.g., `PDTimelinePage`)
- Models: Descriptive names (e.g., `TimelineOptions`, `TimeRange`, `DataPoint`)

---

## 🔧 Development Workflow

### 1. Making Component Changes

```sh
# 1. Make changes to component in PanoramicData.Blazor/
# 2. Build the component library
dotnet build PanoramicData.Blazor/PanoramicData.Blazor.csproj

# 3. Run demo to verify
dotnet run --project PanoramicData.Blazor.Demo/PanoramicData.Blazor.Demo.csproj

# 4. Navigate to: https://localhost:5301/component-name
```

### 2. Adding New Components

1. Create component files in `PanoramicData.Blazor/`
   - `PDNewComponent.razor`
   - `PDNewComponent.razor.cs` (if needed)
   - `PDNewComponent.razor.css` (if needed)
   - `PDNewComponent.razor.js` (if JS interop needed)

2. Create demo page in `PanoramicData.Blazor.Demo/Pages/`
   - `PDNewComponentPage.razor`
   - `PDNewComponentPage.razor.cs` (if needed)

3. Add navigation link in demo `NavMenu.razor`

### 3. Testing

```sh
# Run all tests
dotnet test PanoramicData.Blazor.Test/PanoramicData.Blazor.Test.csproj

# Run specific test class
dotnet test --filter "FullyQualifiedName~FilterTests"

# Run with detailed output
dotnet test PanoramicData.Blazor.Test/PanoramicData.Blazor.Test.csproj -v detailed
```

**Note**: Currently, test coverage is minimal (only `FilterTests.cs` exists). Consider adding tests when fixing bugs or adding features.

---

## 📝 Code Style Guidelines

### C# Conventions

```csharp
// ✅ GOOD: Use primary constructors for DI (C# 12+)
public partial class PDTimeline(IJSRuntime jsRuntime) : ComponentBase
{
    [Inject] public IJSRuntime JSRuntime { get; set; } = jsRuntime;
}

// ✅ GOOD: Use file-scoped namespaces
namespace PanoramicData.Blazor;

// ✅ GOOD: Use modern C# features
private readonly Dictionary<int, DataPoint> _dataPoints = [];
DataPoint[] tempArray = [.. points.Where(x => x != null)];

// ✅ GOOD: Allman braces (opening brace on new line)
public bool CanZoomIn()
{
    if (IsEnabled)
    {
        return true;
    }
    return false;
}

// ❌ AVOID: K&R style braces
public bool CanZoomIn() {
    if (IsEnabled) {
        return true;
    }
    return false;
}

// ✅ GOOD: Private fields with underscore prefix
private int _selectionStartIndex = -1;
private bool _isChartDragging;

// ✅ GOOD: Async methods with ConfigureAwait(true) in Blazor
await SelectionChanged.InvokeAsync(_selectionRange).ConfigureAwait(true);

// ✅ GOOD: XML documentation for public APIs
/// <summary>
/// Gets or sets the current scale of the timeline.
/// </summary>
[Parameter]
public TimelineScale Scale { get; set; } = TimelineScale.Years;
```

### Blazor-Specific Patterns

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panoramicdata/PanoramicData.Blazor](https://github.com/panoramicdata/PanoramicData.Blazor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
