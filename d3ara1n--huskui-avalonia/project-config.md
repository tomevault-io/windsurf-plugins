---
trigger: always_on
description: - When creating a `ControlTemplate`, internal template parts that are NOT exposed as `TemplatePart` attributes should **not** use the `PART_` prefix in their names. Reserve `PART_` exclusively for parts that are required/expected by the control's code-behind via `TemplatePart` attributes. This makes the distinction clear between contractually required parts and internal implementation details.
---

# Project Conventions

## UI Design Rules

- When creating a `ControlTemplate`, internal template parts that are NOT exposed as `TemplatePart` attributes should **not** use the `PART_` prefix in their names. Reserve `PART_` exclusively for parts that are required/expected by the control's code-behind via `TemplatePart` attributes. This makes the distinction clear between contractually required parts and internal implementation details.

- Every control consists of a **paired `.cs` and `.axaml` file** in the same `Controls/` directory. The `.axaml` file is a `ResourceDictionary` containing `ControlTheme`(s), not a standalone control definition. New controls must follow this pairing.

- The default theme for a control uses `x:Key="{x:Type local:ControlName}"` for implicit styling. Alternative visual styles (e.g. Outline, Ghost) use named keys like `OutlineButtonTheme`.

- **Style classes (variant selectors) are `PascalCase`.** A class denotes a **variant** — a named look the consumer opts into — and the name is an adjective or noun describing that variant. Examples in use: `Primary`, `Danger`, `Small`. Color variants (Primary, Success, Warning, Danger) and size variants (Small, Large) are applied via **CSS classes**, not enum properties.

- **Pseudo-classes (state selectors) are `all-lowercase`.** A pseudo-class denotes a **runtime state** of the control. Examples: `:pointerover`, `:pressed`, `:checked`, `:disabled`, `:focus`, `:selected`, `:error`. Never capitalize a pseudo-class.

- The distinguishing question: **is the consumer choosing a look (`Primary`) or is the control reporting its own state (`:pressed`)?** Variant → PascalCase class; state → lowercase pseudo-class.

- Style variants (Default, Outline, Ghost) are applied by switching the `Theme` property to a different `ControlTheme` resource. Classes represent persistent visual variants — their styles target the control itself directly. Pseudo-classes represent transient states and their styles target internal template parts.

- **A variant class may only set the control's own exposed properties** (e.g. `Background`, `Foreground`, `BorderBrush`, `CornerRadius`, `Padding`). It must **not** reach into the control template and restyle named parts (`/template/ Border#PART_Xxx`). Restyling template internals is reserved for pseudo-class-driven states inside the same ControlTheme; variant classes stay at the public-property surface so they compose cleanly when consumers stack them (`Classes="Primary Small"`). For example, a `Primary` class on a Button changes `Background`/`Foreground`; it does **not** touch the inner `ContentPresenter` or `PART_Background` rectangle directly.

- The project uses a **Radix-inspired 12-step color scale** where step 9 is always the base/interactive color. When adding new semantic colors, follow this 12-step pattern and generate both Default (light) and Dark theme dictionary variants.

- Animation durations must reference the **named constants** from `Themes/Basics.axaml` (e.g. `ControlFasterAnimationDuration`, `ControlNormalAnimationDuration`) rather than hard-coded `TimeSpan` values. Transitions should use `SineEaseOut` easing.

- Corner radii must use the **indirection layer** — reference `StaticResource` keys like `SmallCornerRadius`, `FullCornerRadius` etc. from `CornerRadii.axaml`, not raw `CornerRadius` values. These keys already handle dynamic forwarding internally. When a converter is needed on a `StaticResource` corner radius, use `StaticResourceBinding` instead of `StaticResource`.

- The XAML namespace for the library is `https://github.com/d3ara1n/Huskui.Avalonia` (mapped to prefix `husk`). All new namespaces must be registered via `[XmlnsDefinition]` in `AssemblyInfo.cs`.

- **Do NOT run any formatting tools** (`csharpier`, `xstyler`, etc.). They can produce unintended changes across the entire repo. Only the user may invoke formatting.

## Control Implementation Patterns

- `StyledProperty` registration uses multi-line generic syntax with type parameters on separate lines:
  ```csharp
  public static readonly StyledProperty<bool> IsReadOnlyProperty = AvaloniaProperty.Register<
      ControlName,
      bool
  >(nameof(IsReadOnly));
  ```

- Use `OnPropertyChanged` override to react to property changes (including setting `PseudoClasses`), not property change callbacks.

- The `field` keyword is used with `SetAndRaise` for `DirectProperty` accessors:
  ```csharp
  public int PageCount
  {
      get;
      private set => SetAndRaise(PageCountProperty, ref field, value);
  }
  ```

- In `OnApplyTemplate`, event handlers for template parts must be **unregistered then re-registered within the same method** — keep the `-=` and `+=` pairs together so subscription and unsubscription are visible at a glance. Do not split them into separate lifecycle methods.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d3ara1n/Huskui.Avalonia](https://github.com/d3ara1n/Huskui.Avalonia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
