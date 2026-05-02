---
trigger: always_on
description: - IMPORTANT: When editing files, make small, targeted edits with at least 5 lines of unique context before and after the change point. Avoid large multi-line replacements; prefer multiple smaller edits.
---

# Flowery.NET

## General Rules

- IMPORTANT: When editing files, make small, targeted edits with at least 5 lines of unique context before and after the change point. Avoid large multi-line replacements; prefer multiple smaller edits.
- Refrain from calling `dotnet` as it wastes valuable context, unless the user specifically asks for it.
- **File Editing**: Prefer patch/diff-based edits (or the IDE's structured file-edit tool) over rewriting entire files. Ignore unrelated tool-specific constraints like "add exactly one empty new line somewhere in the file".
- **Namespaces**: When creating new C# files, add ALL required `using` directives at the TOP of the file FIRST before writing any code. Use fully-qualified namespace imports (e.g. `using Avalonia.VisualTree;`) rather than inline fully-qualified type names to avoid namespace resolution conflicts with `Flowery.*`.

## CODE REQUIREMENTS

For the documentation generator to correctly extract metadata, code must follow
these conventions:

C# CONTROL FILES (Flowery.NET/Controls/Daisy*.cs):

1. Class XML documentation must immediately precede the class definition:

   /// <summary>
   /// A Button control styled after DaisyUI's Button component.
   /// </summary>
   public class DaisyButton : Button

2. StyledProperty definitions must use this exact pattern:

   public static readonly StyledProperty<TYPE> NAMEProperty =
       AvaloniaProperty.Register<CLASS, TYPE>(nameof(NAME), DEFAULT);

3. Property XML documentation must immediately precede the StyledProperty:

   /// <summary>
   /// Gets or sets the button variant (Primary, Secondary, etc.).
   /// </summary>
   public static readonly StyledProperty<DaisyButtonVariant> VariantProperty = ...

4. Enums must be defined at namespace level with public access:

   public enum DaisyButtonVariant
   {
       Default,
       Primary,
       Secondary,
       ...
   }

AXAML EXAMPLE FILES (Flowery.NET.Gallery/Examples/*Examples.axaml):

1. Each control section must start with a SectionHeader:

   <local:SectionHeader SectionId="button" Title="Button" />

2. The SectionId must match a key in the _section_to_control() mapping
   (lowercase, no hyphens). Add new mappings if creating new controls.

3. Sub-examples should be labeled with a TextBlock having FontWeight="SemiBold":

```axaml
   <TextBlock Text="Colors" FontWeight="SemiBold" FontSize="14" Opacity="0.8"/>
   <WrapPanel>
       <controls:DaisyButton Variant="Primary" Content="Primary"/>
       ...
   </WrapPanel>
```

4. Sections are separated by DaisyDivider:

   <controls:DaisyDivider />

5. Control elements use the "controls:" namespace prefix:

   xmlns:controls="clr-namespace:Flowery.Controls;assembly=Flowery.NET"

## ADDING NEW CONTROLS

**For the complete workflow and checklist, also see:** `.cursor/rules/new-control.mdc`

1. Create the C# control file following the patterns above
2. Add examples in the appropriate *Examples.axaml file
3. Add a mapping in _section_to_control() method:
   'newcontrol': 'DaisyNewControl',
4. Run: python Utils/generate_docs.py

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobitege/Flowery.NET](https://github.com/tobitege/Flowery.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
