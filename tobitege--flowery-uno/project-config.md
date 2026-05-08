---
trigger: always_on
description: This file captures learnings and patterns discovered during Flowery.Uno development for future reference.
---

# Flowery.Uno MANDATORY Development Notes

This file captures learnings and patterns discovered during Flowery.Uno development for future reference.

## 🚨 READ THIS FIRST (top crash-prevention rules)

If you only read the first ~50 lines of this file, read this section. These rules prevent the most common Uno “mystery” runtime crashes:

- **ContentControls that rebuild `Content`**: ALWAYS detach before re-parenting.
  - Pattern: `_userContent = Content; Content = null; BuildVisualTree(); _presenter.Content = _userContent;`
  - See **“UIElement can only have one parent”** (search for `### 19`).
- **Never bind a `ContentPresenter` back to `this.Content`** in a control that later sets `Content = _rootGrid` (creates self-parenting / re-parenting issues).
  - See the anti-pattern example under `### 19`.
- **Don’t use `PathIcon` dynamically in code-behind** (Uno can throw `ArgumentException: Value does not fall within the expected range.`).
  - Use `Microsoft.UI.Xaml.Shapes.Path` or `Flowery.Helpers.FloweryPathHelpers` instead (search for `### 18`).
- **Use `DaisyControlExtensions.Icon` for button icons** - This is the GOLD STANDARD (see section below).
- **Ambiguous type names**: `Path`, `Color`, etc. often need aliasing/qualification.
  - Example: `using Path = Microsoft.UI.Xaml.Shapes.Path;` (search for `### 12`).
- **Do NOT merge `ms-appx:///uno.toolkit.*/Styles/Generic.xaml`**.
  - `Uno.Toolkit.*` packages do not ship `Styles/Generic.xaml`. Adding that dictionary will fail to load resources and can crash at startup. ShadowContainer only needs the package reference.

## Neumorphic Takeaways

For the distilled, field-tested fixes and integration notes from recent stability work, see `llms-static/neumorphic.md` → **Field-Tested Integration Notes (Session Takeaways)**.

### ThemeShadow + Elevation Learnings

- **ThemeShadow requires explicit receivers**: add the receiver to `ThemeShadow.Receivers` (see Uno tests in `!uno/src/SamplesApp/UITests.Shared/Windows_UI_Xaml_Media/ThemeShadowTests`).
- **Translation.Z defines shadow depth**: the casting element must have a `Translation` Z (e.g., `0,0,6`).
- **CornerRadius is respected** when ThemeShadow is applied directly to `Border` or `Rectangle` with radius properties (no custom masking required).
- **WASM/Skia elevation**: for rounded shadows, apply `SetElevation` to the template’s `Border` (e.g., `ButtonBorder`), not the `Button` control itself.

## Pitfalls (Session)

These are compile-time issues that should be avoided up front:

- Do not use `??` with different operand types (e.g., `Border ?? DaisyCard`). Cast to a shared base like `FrameworkElement` first.
- Do not assign `UIElement` to `FrameworkElement` without an explicit cast and a null/type check.
- Do not reference non-existent WinUI/Uno members like `FrameworkElement.IsVisibleChanged`; use supported events or `RegisterPropertyChangedCallback`.
- Do not access internal or private helpers (e.g., `PlatformCompatibility`) from outside their assembly.
- Do not set `null` into non-nullable reference types; update the type or use a nullable value.
- Do not use `x:Bind` paths that are not real properties on the page (e.g., `Localization`); ensure the property exists and follow the required localization binding pattern.

---

## WASM/Browser (Skia) Heads-Up (Session)

These are the practical fixes and gotchas encountered when getting the Browser head running with Skia:

- Use `Microsoft.NET.Sdk.WebAssembly` for the Browser head when using `Uno.WinUI.Runtime.Skia.WebAssembly.Browser`. Do NOT reference `Uno.WinUI.Runtime.WebAssembly` (triggers `UNOB0017`).
- Target `net9.0` with `RuntimeIdentifier=browser-wasm` and run via `dotnet run` on the project (not the output folder) to avoid `hostpolicy.dll` self-contained errors.
- Use `HostBuilder.UseWebAssembly()` directly; avoid reflection-based host builder hacks (inaccessible method errors).
- Fix culture crashes by disabling invariant globalization and including ICU data:
  - `<InvariantGlobalization>false</InvariantGlobalization>`
  - `<WasmIncludeFullIcuData>true</WasmIncludeFullIcuData>`
- Keep SkiaSharp versions aligned across managed + native:
  - `SkiaSharp` and `SkiaSharp.NativeAssets.WebAssembly` MUST match (e.g., `3.119.1`) or you will hit undefined symbol errors.
- Static web assets duplicates (library layout + root assets) cause:
  - `Two assets found targeting the same path with incompatible asset kinds`
  - Fix by disabling root asset copies for `net9.0` library projects and using `ms-appx:///Flowery.Uno.Gallery/Assets/...` paths.
  - Add a Browser-head build target that `RemoveDuplicates` on `@(UnoAllCopyToOutputItems)` before `_UnoAssetsGetCopyToPublishDirectoryItems`.
- CSP warnings (workers, `unsafe-eval`) and `WEBGL_invalid_enum` messages are expected in debug and are not fatal.
- Keep Browser script ports aligned (5236) to avoid mismatched logs vs URL.
- `ms-appx:///` is the correct scheme for Content assets in Uno (including WASM); it is supported by `Image`/`BitmapImage`.
- Referencing assets in code can use `ms-appx:///Assets/...` or `ms-appx:///AssemblyName/Assets/...` and can be bound as a string.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobitege/Flowery.Uno](https://github.com/tobitege/Flowery.Uno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
