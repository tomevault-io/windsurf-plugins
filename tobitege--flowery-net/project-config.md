---
trigger: always_on
description: This rule outlines the complete workflow for creating a new control in Flowery.NET, whether it's a brand new control or a port from another library.
---

# Creating a New Flowery.NET Control

This rule outlines the complete workflow for creating a new control in Flowery.NET, whether it's a brand new control or a port from another library.

## Overview

Creating a new control involves these major phases:

1. **Control Implementation** - C# class with properties and logic
2. **Theme/Styling** - AXAML theme file for visual appearance
3. **Gallery Examples** - Demo page in the gallery application
4. **Sidebar Integration** - Add to the component sidebar
5. **Documentation** - Markdown file for supplementary docs

---

## Phase 1: Control Implementation

### 1.1 Create the C# Control File

**Location:** `Flowery.NET/Controls/Daisy[ControlName].cs`

> If the control is a **custom extension** (not a direct DaisyUI component), place it under:
>
> - `Flowery.NET/Controls/Custom/Daisy[ControlName].cs`
> - Namespace: `Flowery.Controls.Custom`

**Required elements:**

```csharp
using System;
using Avalonia;
using Avalonia.Controls;
using Avalonia.Controls.Primitives;
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
    /// Brief description of what the control does.
    /// </summary>
    public class Daisy[ControlName] : TemplatedControl // or appropriate base class
    {
        // StyleKey override for theming
        protected override Type StyleKeyOverride => typeof(Daisy[ControlName]);

        /// <summary>
        /// Gets or sets the property description.
        /// </summary>
        public static readonly StyledProperty<Type> PropertyNameProperty =
            AvaloniaProperty.Register<Daisy[ControlName], Type>(nameof(PropertyName), defaultValue);

        public Type PropertyName
        {
            get => GetValue(PropertyNameProperty);
            set => SetValue(PropertyNameProperty, value);
        }
    }
}
```

### 1.2 Key Requirements

- **XML Documentation**: Every class and property MUST have `/// <summary>` comments
- **StyledProperty Pattern**: Use the exact pattern shown above for documentation generator
- **Enums at Namespace Level**: Define enums outside the class, with `public` access
- **Using Directives**: Add ALL required usings at the file TOP (avoid inline fully-qualified names)
- **Preset “modes” / “bases” (optional)**: If the control supports multiple input/format modes (like `DaisyNumericUpDown.NumberBase`), implement them as a `public enum` + `StyledProperty<Enum>` and apply the mode in code-behind when the property changes.

---

## Phase 2: Theme/Styling

### 2.1 Create the Theme File

**Location:** `Flowery.NET/Themes/Daisy[ControlName].axaml`

> If the control is in `Flowery.NET/Controls/Custom/`, put the theme under:
>
> - `Flowery.NET/Themes/Custom/Daisy[ControlName].axaml`
> - and use the matching XAML namespace: `clr-namespace:Flowery.Controls.Custom;assembly=Flowery.NET`

```xml
<ResourceDictionary xmlns="https://github.com/avaloniaui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:controls="clr-namespace:Flowery.Controls;assembly=Flowery.NET">

    <ControlTheme x:Key="{x:Type controls:Daisy[ControlName]}" TargetType="controls:Daisy[ControlName]">
        <Setter Property="Template">
            <ControlTemplate>
                <!-- Control template here -->
            </ControlTemplate>
        </Setter>
        <!-- Style setters for variants, states, etc. -->
    </ControlTheme>

</ResourceDictionary>
```

### 2.2 Register the Theme

Add the theme to `Flowery.NET/DaisyUITheme.axaml`:

```xml
<ResourceInclude Source="avares://Flowery.NET/Themes/Daisy[ControlName].axaml" />
```

> For custom controls, include the custom theme path:
>
> - `/Themes/Custom/Daisy[ControlName].axaml`

---

## Phase 3: Gallery Examples

### 3.1 Create or Update Examples File

**Location:** `Flowery.NET.Gallery/Examples/[Category]Examples.axaml`

Add a section for the new control:

```xml
<!-- Control Name -->
<StackPanel Spacing="12">
    <local:SectionHeader SectionId="controlname" Title="Control Name" />
    <TextBlock Text="Description of the control." Opacity="0.7" />
    <WrapPanel Orientation="Horizontal">
        <StackPanel Spacing="10" Margin="0,0,40,20">
            <TextBlock Text="Basic Usage" FontSize="12" Opacity="0.7"/>
            <controls:Daisy[ControlName] />
        </StackPanel>
        <!-- More examples -->
    </WrapPanel>
</StackPanel>

<controls:DaisyDivider />
```

### 3.2 Code-Behind (if needed)

If the examples need event handlers or dynamic behavior, add them to the `.axaml.cs` file.

### 3.3 Implement IScrollableExample

Ensure the examples class implements `IScrollableExample`:

```csharp
public partial class [Category]Examples : UserControl, IScrollableExample
{
    public void ScrollToSection(string sectionName)
    {
        // Standard implementation - copy from existing examples
    }
}
```

### 3.4 Register SectionId → Control mapping (required)

If you add a new `<local:SectionHeader SectionId="..." />`, ensure the ID maps to the control name in BOTH places:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobitege/Flowery.NET](https://github.com/tobitege/Flowery.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
