---
trigger: always_on
description: This document defines strict guardrails for any AI-assisted or automated agent contributions (including Copilot, custom prompt runners, or scripted refactors) working in the **Uno.Themes** repository. Human contributors must also ensure generated changes comply before merge. It is the single source of truth for repo-wide orientation *and* the rules agents must follow; `CLAUDE.md` at the repo root is a thin pointer that includes this file.
---

# AI Agents Contribution & Coding Instructions

This document defines strict guardrails for any AI-assisted or automated agent contributions (including Copilot, custom prompt runners, or scripted refactors) working in the **Uno.Themes** repository. Human contributors must also ensure generated changes comply before merge. It is the single source of truth for repo-wide orientation *and* the rules agents must follow; `CLAUDE.md` at the repo root is a thin pointer that includes this file.

<repository_orientation>

## Repository overview

Uno.Themes ships the design-system theme libraries used by Uno Platform / WinUI apps (Material Design, Cupertino / iOS-style, and the Simple density-driven theme), plus the shared Uno Themes core (semantic palette, seed-color generation, design tokens) and C# Markup helpers. The repo produces seven NuGet packages:

- `Uno.Themes.WinUI` — base theme infrastructure (semantic palette, seed-color generation, design tokens, shared converters/extensions). Built from `src/library/Uno.Themes/Uno.Themes.WinUI.csproj`. All other theme libraries reference it.
- `Uno.Material.WinUI` — Material Design 3 styles and resources. Built from `src/library/Uno.Material/Uno.Material.WinUI.csproj`. Has a v1/v2 style split (see `Styles/Application/v1`, `Styles/Application/v2`, etc.).
- `Uno.Cupertino.WinUI` — Cupertino / iOS-style theme. Built from `src/library/Uno.Cupertino/Uno.Cupertino.WinUI.csproj`.
- `Uno.Simple.WinUI` — minimal density-driven theme (see `Density.cs`, `SimpleTheme.cs`). Built from `src/library/Uno.Simple.WinUI/Uno.Simple.WinUI.csproj`.
- `Uno.Themes.WinUI.Markup`, `Uno.Material.WinUI.Markup`, `Uno.Simple.WinUI.Markup` — C# Markup helpers (extension methods, brush/color/style accessors) layered on top of the corresponding theme library.

Despite the legacy folder name `src/library/Uno.Themes`, the assembly is `Uno.Themes.WinUI` (see `AssemblyName` in the csproj). Internals of the base library are visible to `Uno.Material(.WinUI)`, `Uno.Cupertino(.WinUI)`, and `Uno.Simple.WinUI` via `InternalsVisibleTo` (see `themes-common.props`).

## Solution layout

- `Uno.Themes.sln` (repo root) — main solution. Includes all libraries, samples, and the runtime tests that live inside the sample apps. Open this for full development.
- `Uno.Themes-packages.slnf` (repo root) — solution filter limited to the packable libraries; useful for fast restore/build without the sample apps.
- `src/library/` — all packable libraries. Subdirectories match the project list above. Theme XAML lives under each library's `Styles/` folder (e.g. `src/library/Uno.Material/Styles/{Application,Controls}/{v1,v2,Common}/`, `src/library/Uno.Cupertino/Styles/{Application,Controls}/`, `src/library/Uno.Simple.WinUI/Styles/`). The base `src/library/Uno.Themes/Styles/Applications/Common/` holds shared color/typography palettes (`SharedColorPalette.xaml`, `SharedColors.xaml`, `SharedTypography.xaml`, `Converters.xaml`).
- `src/library/xamlmerge.targets` — pulls in `Uno.XamlMerge.Task`. Each library's `*-common.props` declares `XamlMergeInput` items; the task merges them into `mergedpages.xaml` (Material splits into `mergedpages.v1.xaml` / `mergedpages.v2.xaml`). **Do not hand-edit anything generated under `obj/` or referenced via the merged dictionary.**
- `src/samples/` — runnable sample apps and the shared sample project:
  - `src/samples/SamplesApp.Shared/` — shared project (`.shproj` / `.projitems`) holding the sample UI, `Shell.xaml`, content pages, view models, helpers.
  - `src/samples/MaterialSampleApp/` — Material sample head.
  - `src/samples/CupertinoSampleApp/` — Cupertino sample head.
  - `src/samples/SimpleSampleApp/` — Simple sample head; **also hosts the runtime tests** under `src/samples/SimpleSampleApp/RuntimeTests/Given_*.cs` (e.g. `Given_SeedColorPalette.cs`, `Given_SemanticStyles.cs`, `Given_ColorOverridePrecedence.cs`).
- `doc/` — published documentation (see §13).

There is **no separate runtime-tests project** — runtime tests live inside the sample apps and are driven by `Uno.UI.RuntimeTests.Engine` (`PackageReference Include="Uno.UI.RuntimeTests.Engine"` in each sample csproj).

## Target frameworks and platform builds

Target frameworks are managed centrally:

- `src/library/tfm-common-winui.props` expands library projects to `net9.0` plus per-platform suffixes (`net9.0-ios`, `net9.0-android`, `net9.0-windows10.0.19041`, `net9.0-maccatalyst`) based on `TargetFrameworkOverride` and the `Build_iOS` / `Build_Android` / `Build_Windows` switches.
- Each sample csproj declares its own `net10.0-*` set directly (see `MaterialSampleApp.csproj`, `CupertinoSampleApp.csproj`, `SimpleSampleApp.csproj`). Without an override, samples target `net10.0-android;net10.0-ios;net10.0-browserwasm;net10.0-desktop`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unoplatform/Uno.Themes](https://github.com/unoplatform/Uno.Themes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
