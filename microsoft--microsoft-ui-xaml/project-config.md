---
trigger: always_on
description: This document explains **how** the sample is built and **why** each non-obvious step exists. The
---

# TableView sample — build & setup notes (for agents and maintainers)

This document explains **how** the sample is built and **why** each non-obvious step exists. The
end-user quick start lives in [README.md](README.md); this file is the deep reference for anyone
(human or automation) that needs to reproduce, debug, or maintain the build.

## Why this sample is unusual

`TableView` currently ships as a **split binary**: the control lives in
`Microsoft.UI.Xaml.Controls.Tabular.dll`, separate from the main framework DLL, and it is **not
yet exposed through the WindowsAppSDK NuGet package**. A normal WinUI app can't just add a package
reference and use it. To consume the locally built control, the sample:

1. links the freshly built native control DLL,
2. regenerates its WinRT projection from the freshly built WinMD, and
3. reconstructs, at build and startup time, the pieces the WindowsAppSDK packaging would normally
   provide (activatable-class registrations, theme resources, default styles).

Every workaround below disappears once `TableView` is delivered in-box through WindowsAppSDK.

## Environment setup

1. Install Visual Studio 2022 with the **Desktop development with C++** and **.NET Desktop**
   workloads.
2. From the repo root, run a full initialization once so packages are restored and the build
   environment is provisioned:

   ```
   .\init.cmd
   ```

   For one-shot commands in an already-initialized enlistment, `initrun.ps1 <command>` sets up the
   environment for a single invocation without persisting a shell.

## Step 1 — Build the Tabular control

```
.\initrun.ps1 controls\Build.cmd tabular
```

Produces:

- `BuildOutput\obj\amd64chk\controls\dev\dll-tabular\Microsoft.UI.Xaml.Controls.Tabular.dll`
- `BuildOutput\obj\amd64chk\controls\dev\dll-tabular\Merged\*.winmd` (the metadata the sample
  projects against)
- `BuildOutput\obj\amd64chk\controls\dev\dll-tabular\generic.xaml` (the default Style /
  ControlTemplate slice the sample compiles)

The sample consumes these outputs directly, so the control must be built **before** the sample.

## Step 2 — Build the sample

```
.\initrun.ps1 msb /q /restore Samples\TableViewSampleApp\TableViewSampleApp.csproj /p:Platform=x64 /p:RuntimeIdentifier=win-x64
```

The project (`TableViewSampleApp.csproj`) performs several workarounds during this build. Each
is described below with the failure it prevents.

### a. Stage the control DLL next to the executable

The native `Microsoft.UI.Xaml.Controls.Tabular.dll` is copied next to the app. TableView's
runtime classes are registered (see step *e*) but cannot activate without the DLL present.

*Prevents:* `CLASS_E_CLASSNOTAVAILABLE` (`0x80040111`) when a `TableView` is first activated.

### b. Regenerate the CsWinRT projection from the built WinMD

The mock WindowsAppSDK projection can carry stale `Microsoft.UI.Xaml.Controls.Tabular` metadata.
When the built control's interface IIDs diverge from the stale projection, calls such as
`TableView.get_Columns` fail their `QueryInterface`. The project regenerates the projection with
CsWinRT from the freshly built WinMD (`CsWinRTFilters` include the
`Microsoft.UI.Xaml.Controls.Tabular` namespace and the Tabular XAML metadata provider types) so
the projected IIDs match the control exactly.

*Prevents:* `E_NOINTERFACE` from stale metadata.

### c. Include the TableView theme resources — sourced from the control, never checked in

The split binary's theme resources are **not** deployed to consuming apps, so the sample compiles
and merges them itself. To guarantee they can never drift from the control, the sample references
the canonical sources directly instead of checking in copies:

- `TabularSurfaces_themeresources.xaml` ← `controls\dev\CommonStyles\TabularSurfaces_themeresources.xaml`
- `TableView_themeresources.xaml` ← `controls\dev\TableView\TableView_themeresources.xaml`
- `generic.xaml` (default `Style` + `ControlTemplate`) ← the freshly built
  `BuildOutput\...\dll-tabular\generic.xaml`

These are wired via `<Page>`/`<Content>` items with `<Link>` metadata (see the theme-resources
`ItemGroup` in the csproj), app-compiled to the same ms-appx paths the app expects, and merged at
startup by `App.xaml` / `App.xaml.cs`.

*Prevents:* the control rendering unstyled or blank; its native `MeasureOverride` throwing on
first layout when the template is missing.

### d. Build self-contained and unpackaged

The project sets `WindowsAppSdkSelfContained=true`, `WindowsPackageType=None`. This bundles the
WindowsAppRuntime framework natives and emits the in-process-server activation entries that the
packaged bootstrap would otherwise inject.

*Prevents:* startup failure (`0xC000027B`) on machines lacking a matching framework package. This
is standard WindowsAppSDK behavior for unpackaged apps, not a defect.

### e. Merge the InteractiveExperiences (IXP) app manifest

`Build\MergeIxpAppManifest.ps1` merges the WindowsAppSDK InteractiveExperiences component package
`appxfragment` into the app's side-by-side manifest, adding the lifted-WinRT activatable-class
registrations (CoreMessaging, Dispatching, Input, Windowing, etc.) that the mock aggregator's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/microsoft-ui-xaml](https://github.com/microsoft/microsoft-ui-xaml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
