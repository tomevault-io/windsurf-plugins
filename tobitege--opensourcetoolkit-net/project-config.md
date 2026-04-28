---
trigger: always_on
description: - ONLY call `dotnet` commands if requested by user.
---

# Agent Rules

- ONLY call `dotnet` commands if requested by user.

## Avalonia UI Rules

- **RelayCommand CanExecute**: When creating a `RelayCommand` with a `CanExecute` condition (e.g. `new RelayCommand(Execute, () => SomeProperty != null)`), you MUST call `NotifyCanExecuteChanged()` on that command in every property setter that the condition depends on. Failure to do this will leave buttons permanently disabled!
- When the user mentions `glyphs`, use `PathIcon` with the appropriate data attribute.
- Avalonia's compiled bindings require an `x:DataType` on the `DataTemplate` so it knows the item type. Thus add `x:DataType="vm:DataTypeItem"` to the template.
- **UI Composition**: Avoid rigid mutual exclusion in ViewModel setters. Use computed properties (e.g. `IsVisible => EnableZoom || ShowComparison`) to drive UI visibility.
- **Visual Feedback**: Explicitly style active buttons (e.g. `Classes.Add("accent")`) to indicate state; do not rely on default button appearance.
- **Window Sizing**: Use conservative default dimensions (e.g. 600x500) with explicit `MinWidth`/`MinHeight` to support high-DPI scaling.
- **Clipboard Access**: Do not access clipboard directly from ViewModel. Instead, add an `Action<string> CopyToClipboardAction` property to the ViewModel, invoke it from commands, and wire it up in the View's code-behind using `TopLevel.GetTopLevel(this)?.Clipboard?.SetTextAsync(text)`.
- **Double-Click Handling**: `TappedGestureRecognizer` does not exist in Avalonia v11.x. Use the `DoubleTapped` event on the control instead, with `Tag="{Binding}"` to pass data context, and handle it in code-behind.
- **App-Wide Styles**: For common control settings (e.g., `VerticalContentAlignment="Top"` for TextBox), add app-wide styles in `App.axaml` under `<Application.Styles>` rather than repeating them on individual controls. This ensures consistency and reduces duplication.
- **Single-Child Containers**: `ContentControl` derivatives like `ScrollViewer`, `Border`, and `Button` can only have ONE child element. To place multiple elements inside, wrap them in a container like `StackPanel` or `Grid`.
- **ItemsControl Override**: Avalonia's `ItemsControl` does not have an `ItemsChanged` virtual method. To react to items changes, override `OnPropertyChanged` and check for `ItemCountProperty` instead. Use `VisualTreeAttachmentEventArgs` from `Avalonia.VisualTree` namespace for `OnAttachedToVisualTree`.

## Theme Rules

- **Icons Use StaticResource**: `PathIcon.Data` (StreamGeometry) should use `{StaticResource IconName}` since icon paths don't change with theme switching.
- **ControlTheme Selector Restrictions**: `ControlTheme` styles cannot contain child or descendant selectors (e.g. `^[Property=Value] ChildControl`). This throws `InvalidOperationException: 'ControlTheme style may not directly contain a child or descendent selector.'` To fix this:
  1. Change the root element from `<ResourceDictionary>` to `<Styles>`
  2. Wrap `ControlTheme` definitions inside `<Styles.Resources>...</Styles.Resources>`
  3. Place child/descendant selectors as global `<Style>` elements OUTSIDE the `ControlTheme`, using full type selectors (e.g. `controls|MyControl[Property=Value] ChildControl`)
- **Border Has No Foreground**: `Border` does not have a `Foreground` property. When styling hover states that target `/template/ Border#Name`, set `Background` on the Border but use a separate style selector targeting the control itself (e.g. `^:pointerover`) for `Foreground` changes.

### Theming Pitfalls

- **Duplicate Setter Exception**: Do NOT set both `Foreground` and `TextElement.Foreground` on the same element in styles - this causes `System.InvalidOperationException: Duplicate setter encountered for property 'Foreground'`.
- **Always Use DynamicResource for Theme Colors**: `{StaticResource Daisy*Brush}` will NOT update when theme changes at runtime. Always use `{DynamicResource Daisy*Brush}`.
- **TextBlock Foreground Fallback**: If a global `TextBlock` style in `App.axaml` doesn't apply in certain contexts (TabItem headers, DataGrid cells, control templates), add explicit `Foreground="{DynamicResource DaisyBaseContentBrush}"` to those TextBlocks.

## Flowery.NET Usage

This project uses **Flowery.NET** for theming and controls. The theming system uses `DaisyThemeManager` for runtime theme switching.

### Namespace Declaration

```xml
xmlns:controls="clr-namespace:Flowery.Controls;assembly=Flowery.NET"
```

Or use the shorter alias commonly seen:

```xml
xmlns:daisy="clr-namespace:Flowery.Controls;assembly=Flowery.NET"
```

### Dynamic Resource Keys

**Always use Daisy resource keys directly** (not aliases) for proper runtime theme updates:

| Resource Key | Purpose |
|-------------|---------|
| `DaisyBase100Brush` | Primary background |
| `DaisyBase200Brush` | Secondary background (cards, sidebar) |
| `DaisyBase300Brush` | Tertiary background, borders |
| `DaisyBaseContentBrush` | Primary text color |
| `DaisyNeutralBrush` | Neutral fill |
| `DaisyNeutralContentBrush` | Secondary text color |
| `DaisyPrimaryBrush` | Primary accent color |
| `DaisySecondaryBrush` | Secondary accent |
| `DaisyAccentBrush` | Tertiary accent |
| `DaisyInfoBrush` | Info status color |
| `DaisySuccessBrush` | Success status color |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobitege/OpenSourceToolkit.NET](https://github.com/tobitege/OpenSourceToolkit.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
