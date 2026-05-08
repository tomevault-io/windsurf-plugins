---
trigger: always_on
description: This rule outlines the complete workflow for creating a new control in Flowery.Uno, whether it's a brand new control or a port from another library.
---

# Creating a New Flowery.Uno Control (Uno Platform / WinUI)

This rule outlines the complete workflow for creating a new control in Flowery.Uno, whether it's a brand new control or a port from another library.

> **Platform**: Uno Platform / WinUI (NOT Avalonia). Use `Microsoft.UI.Xaml.*` namespaces, `.xaml` files, and `DependencyProperty`.

## Overview

Creating a new control involves these major phases:

1. **Control Implementation** - C# class with properties and logic
2. **Theme/Styling** - XAML ResourceDictionary for visual appearance
3. **Gallery Examples** - Demo page in the gallery application
4. **Sidebar Integration** - Add to the sidebar data
5. **Documentation** - Markdown file for supplementary docs

---

## Phase 1: Control Implementation

### 1.1 Create the C# Control File

**Location:** `Flowery.Uno/Controls/Daisy[ControlName].cs`

> If the control is a **custom extension** (not a direct DaisyUI component), place it under:
>
> - `Flowery.Uno/Controls/Custom/Daisy[ControlName].cs`
> - Namespace: `Flowery.Controls`

**Required elements:**

```csharp
using System;
using Flowery.Theming;
using Microsoft.UI.Xaml;
using Microsoft.UI.Xaml.Controls;
using Microsoft.UI.Xaml.Input;
using Microsoft.UI.Xaml.Media;
using Windows.UI;
// Add other required usings at the TOP

namespace Flowery.Controls
{
    // Define enums at namespace level
    public enum Daisy[ControlName]Variant
    {
        Default,
        Primary,
        // ... other values
    }

    /// <summary>
    /// Brief description of what the control does (Uno/WinUI).
    /// </summary>
    public partial class Daisy[ControlName] : ContentControl // or appropriate base class
    {
        /// <summary>
        /// Gets or sets the property description.
        /// </summary>
        public static readonly DependencyProperty VariantProperty =
            DependencyProperty.Register(
                nameof(Variant),
                typeof(Daisy[ControlName]Variant),
                typeof(Daisy[ControlName]),
                new PropertyMetadata(Daisy[ControlName]Variant.Default, OnAppearanceChanged));

        public Daisy[ControlName]Variant Variant
        {
            get => (Daisy[ControlName]Variant)GetValue(VariantProperty);
            set => SetValue(VariantProperty, value);
        }

        private static void OnAppearanceChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            if (d is Daisy[ControlName] control)
            {
                control.ApplyAll();
            }
        }

        public Daisy[ControlName]()
        {
            DefaultStyleKey = typeof(Daisy[ControlName]);
            Loaded += OnLoaded;
            Unloaded += OnUnloaded;
        }

        private void OnLoaded(object sender, RoutedEventArgs e)
        {
            BuildVisualTree();
            ApplyAll();
        }

        private void OnUnloaded(object sender, RoutedEventArgs e)
        {
            // Cleanup (unsubscribe from events, etc.)
        }

        private void BuildVisualTree()
        {
            // Build control's visual tree programmatically
        }

        private void ApplyAll()
        {
            // Apply sizing, colors, states
        }
    }
}
```

### 1.2 Key Requirements

- **XML Documentation**: Every class and property MUST have `/// <summary>` comments
- **DependencyProperty Pattern**: Use WinUI's `DependencyProperty.Register` with `PropertyMetadata`
- **Enums at Namespace Level**: Define enums outside the class, with `public` access
- **Using Directives**: Add ALL required usings at the file TOP (avoid inline fully-qualified names)
- **Partial class**: Mark class as `partial` for Uno source generators
- **DefaultStyleKey**: Set in constructor for templated controls

### 1.3 Common Base Classes

| Base Class | Use Case |
|------------|----------|
| `ContentControl` | Controls with single content slot |
| `Button` | Button-like controls (inherits click handling) |
| `Control` | Generic base for custom controls |
| `ItemsControl` | Controls with item collections |
| `UserControl` | Simple composite controls with XAML |

---

## Phase 2: Theme/Styling

### 2.1 Programmatic Visual Tree (Recommended)

Most Flowery.Uno controls build their visual tree in code-behind for runtime flexibility:

```csharp
private void BuildVisualTree()
{
    _rootGrid = new Grid();
    // Build tree...
    Content = _rootGrid;
}
```

### 2.2 XAML Template (Alternative)

If using XAML templates, add to `Flowery.Uno/Themes/DaisyControls.xaml`:

```xml
<!-- In DaisyControls.xaml -->
<Style TargetType="controls:Daisy[ControlName]">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="controls:Daisy[ControlName]">
                <Grid>
                    <!-- Control template here -->
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

### 2.3 Theme Registration

Themes are merged in `Flowery.Uno/Themes/Generic.xaml`:

```xml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobitege/Flowery.Uno](https://github.com/tobitege/Flowery.Uno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
