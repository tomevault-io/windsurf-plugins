---
trigger: always_on
description: **When you make an error and the user corrects you with specific guidance, patterns, or rules, you MUST update this instruction file with that information before proceeding.** This ensures:
---

# DIPS.Mobile.UI - AI Coding Agent Instructions

## Important Meta-Instruction
**When you make an error and the user corrects you with specific guidance, patterns, or rules, you MUST update this instruction file with that information before proceeding.** This ensures:
- Future sessions benefit from learned patterns
- Common mistakes are not repeated
- Project-specific conventions are documented
- The instruction file evolves with the project

Add corrections to the relevant section (Critical Patterns, Common Pitfalls, etc.) or create a new section if needed.

## Skills
**IMPORTANT**: Before performing any task, always check `.github/skills/` and `.github/agents/` for applicable skill and agent files. Each subfolder contains a `SKILL.md` or agent definition with specific workflows, templates, trigger phrases, and rules that MUST be followed when their trigger conditions match.

**Workflow**: List all folders in `.github/skills/` and `.github/agents/` (if they exist), read each definition file to check if it applies to the current task, and follow its steps exactly.

## Project Overview
DIPS.Mobile.UI is a .NET MAUI component library for iOS and Android mobile apps in the healthcare domain. Components follow a design system with resources (colors, sizes, icons) auto-generated from Figma via DIPS.Mobile.DesignTokens pipeline.

**Key Architecture Pattern**: Multi-target library (`net10.0`, `net10.0-ios`, `net10.0-android`) using partial classes for platform-specific implementations.

## Critical Patterns

### 1. Platform-Specific Code Structure
Use partial classes to separate shared and platform logic:
```
Effects/Touch/
├── Touch.cs                    # Shared API
├── TouchPlatformEffect.cs      # Shared effect base
├── iOS/TouchPlatformEffect.cs  # iOS implementation
├── Android/TouchPlatformEffect.cs  # Android implementation
└── dotnet/TouchPlatformEffect.cs   # Stub for net10.0
```

**Pattern**: Declare `partial void Init()` in shared file, implement in platform files. Never use `#if` preprocessor directives if not necessary.

### 2. Component Architecture
Components extend MAUI controls with custom handlers and platform effects:
- **Components**: UI elements in `Components/` (Button, ListItem, NavigationListItem)
- **Effects**: Cross-cutting behaviors via `RoutingEffect` or `Behaviour` (Touch, Layout, Accessibility)
- **Handlers**: Platform-specific renderers when MAUI handlers need customization

Example: `ListItem` is a `Grid` with configurable Options (TitleOptions, IconOptions, etc.) that bind to internal child views.

**BindableProperty Pattern**: Use inline lambda expressions in `propertyChanged` callback:
```csharp
// ✅ Correct pattern used in ListItem.Properties.cs
public static readonly BindableProperty TitleProperty = BindableProperty.Create(
    nameof(Title),
    typeof(string),
    typeof(ListItem),
    propertyChanged: (bindable, _, _) => ((ListItem)bindable).AddTitle());

// ❌ Wrong - don't create separate callback methods
propertyChanged: OnTitleChanged

private static void OnTitleChanged(BindableObject bindable, object oldValue, object newValue)
{
    ((ListItem)bindable).AddTitle();
}
```

### 3. Design Resources Usage
Always use generated resource APIs (ColorResources, IconResources, SizeResources), never hardcode values:

**CRITICAL**: Before using any color, size, or icon name, **always verify it exists** by checking the generated resource files:
- Colors: `src/library/DIPS.Mobile.UI/Resources/Colors/ColorName.cs`
- Sizes: `src/library/DIPS.Mobile.UI/Resources/Sizes/SizeName.cs`
- Icons: `src/library/DIPS.Mobile.UI/Resources/Icons/IconName.cs`

```csharp
// ✅ Correct - verified names from ColorName.cs and IconName.cs
Colors.GetColor(ColorName.color_surface_default)
Colors.GetColor(ColorName.color_surface_default_selected)  // NOT color_surface_selected!
Sizes.GetSize(SizeName.content_margin_small)
Icons.GetIcon(IconName.check_circle_line)  // NOT checkbox_circle_line!
Icons.GetIcon(IconName.radio_unchecked_line)  // NOT checkbox_blank_circle_line!

// ❌ Wrong - hardcoded values
Color.FromRgb(255, 0, 0)
new Thickness(16)

// ❌ Wrong - using non-existent resource names
Colors.GetColor(ColorName.color_surface_selected)  // Does not exist!
Icons.GetIcon(IconName.checkbox_circle_line)  // Does not exist! Use check_circle_line
Icons.GetIcon(IconName.checkbox_blank_circle_line)  // Does not exist! Use radio_unchecked_line
```

**Icon Syntax in XAML**:
```xaml
<!-- ✅ Correct - verified icon names from IconName.cs -->
<dui:Image Source="{dui:Icons check_circle_line}" />
<dui:Image Source="{dui:Icons radio_unchecked_line}" />

<!-- ❌ Wrong - incorrect syntax -->
<dui:Image Source="{dui:Icon Icon=check_circle_line}" />

<!-- ❌ Wrong - non-existent icon names -->
<dui:Image Source="{dui:Icons checkbox_circle_line}" />
<dui:Image Source="{dui:Icons checkbox_blank_circle_line}" />
```

Resources auto-update from design tokens - direct values break design system consistency.

### 4. Accessibility Requirements
When adding touch/tap behavior, **always** require `SemanticProperties.Description`:
```csharp
// TouchPlatformEffect pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DIPSAS/DIPS.Mobile.UI](https://github.com/DIPSAS/DIPS.Mobile.UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
