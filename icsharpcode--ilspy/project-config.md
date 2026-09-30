---
trigger: always_on
description: Guidance for coding agents when working on ILSpy.
---

# AGENTS.md

Guidance for coding agents when working on ILSpy.

## What this codebase is

ILSpy is a cross-platform .NET assembly browser / decompiler built on **Avalonia 12**, on top of the cross-platform `ICSharpCode.ILSpyX` and `ICSharpCode.Decompiler` core libraries.

## Tech stack

- **Avalonia 12** (not 11.x).
- **AvaloniaEdit** for the decompiled-code text view.
- **Dock** (wieslawsoltes/Dock) for the panel layout. NuGet id ≠ CLR namespace — `Dock.Controls.Recycling` lives in `Avalonia.Controls.Recycling`; decompile before guessing xmlns.
- **Avalonia.Xaml.Behaviors** for attached-behaviour glue.
- **Avalonia.ExtendedToolkit** (mameolan) for controls not in Avalonia core.
- **Simple** theme (not Fluent). Check `App.axaml` / csproj before assuming Fluent.
- **Microsoft.Extensions.DependencyInjection** + **System.Composition** MEF directly, with a small bridge.
- Central package management is enabled — every `PackageReference` needs a matching `PackageVersion` in `Directory.Packages.props`.
- Target framework: `net10.0` (cross-platform) for the main app. The test projects target `net11.0` (and `net11.0-windows` for tests that intentionally exercise Windows-only behaviour) so they run on the runtime the `net11.0` preview build SDK ships, without installing a separate `net10.0` runtime in CI. `TestPlugin` stays `net10.0` (it is loaded as a library by the net11 test host, so its TFM need not track the test projects').

## Project structure

Avalonia UI:
- `ILSpy/` — the Avalonia UI app
- `ILSpy.Tests/` — headless Avalonia UI tests (Avalonia.Headless.NUnit)
- `ILSpy.Tests.Windows/` — Windows-only UI tests (OS-gated; `net11.0-windows`)
- `ILSpy.ReadyToRun/` — ReadyToRun-viewer plugin.

Cross-platform core (decompiler engine + shared support):
- `ICSharpCode.Decompiler/` — core decompiler library (multi-targeted, cross-platform)
- `ICSharpCode.ILSpyX/` — shared UI-host-agnostic support library
- `ICSharpCode.BamlDecompiler/` — BAML parsing library
- `ICSharpCode.ILSpyCmd/` — CLI front-end (`ilspycmd`)
- `ICSharpCode.Decompiler.Tests/` + `ICSharpCode.Decompiler.TestRunner/` — decompiler test suite and its out-of-process runner
- `ICSharpCode.Decompiler.PowerShell/` — PowerShell cmdlets (`netstandard2.0`)

Test support:
- `TestPlugin/` — sample plugin exercising the plugin-loading system (`net10.0`)
- `TestFixtures.Resources/` — generates resource fixtures consumed by the decompiler tests
- `TestTools/` — file-based apps run by hand against real-world assemblies: `nugetfuzz` (crash/assert sweep over nuget.org packages) and `decompdiff` (output diff between two decompiler builds). In no solution and not run by CI; see `TestTools/README.md`

Windows-only frontends, packaging, and tests:
- `ILSpy.AddIn/`, `ILSpy.AddIn.VS2022/` — Visual Studio add-ins (`net472`)
- `ILSpy.Installer/` — WiX installer (`net472`)
- `ILSpy.BamlDecompiler.Tests/` — BAML-decompiler tests, still WPF/Windows-bound (`net11.0-windows`)

Solutions & filters: `ILSpy.sln` builds everything; `ILSpy.XPlat.slnf` is the decompiler libs + `ilspycmd` + their tests (no UI — the Linux CI target); `ILSpy.Desktop.slnf` is the UI plus its dependencies and tests; `ILSpy.Installer.sln` covers the legacy packaging, and `ILSpy.VSExtensions.slnx` the VS 2022 extension (`dotnet build`-able).

## ILSpy-tests submodule

- `ILSpy-tests/` is a **git submodule** (`https://github.com/icsharpcode/ILSpy-tests`, branch `master`) holding large real-world assemblies and pre-built fixtures used by the heavyweight decompiler tests — the round-trip suite (`ICSharpCode.Decompiler.Tests/RoundtripAssembly.cs`) and a few IL-pretty cases (e.g. `FSharp/FSharp.Core.dll`).
- **It is not checked out by default**, because it is large. Tests that need it call `Assert.Ignore` when the directory is absent (see `RoundtripAssembly`/`ILPrettyTestRunner`), so the rest of the suite runs without it — a green local run does **not** mean the round-trip tests ran. To run them, populate it first: `git submodule update --init ILSpy-tests` (or clone it separately to that path).
- **It opts out of the host build settings on purpose.** The submodule ships its own `ILSpy-tests/Directory.Build.props` that sets `TreatWarningsAsErrors=false`; its mere presence also stops MSBuild's upward `Directory.Build.props` search at the submodule root, so the repo-wide warnings-as-errors (and other root build properties) don't leak into the fixtures, which intentionally contain warning-generating code. Don't delete that file or "fix" warnings inside the fixtures.
- **Bumping it** is a normal submodule pointer update: check out the desired commit inside `ILSpy-tests/`, then commit the changed submodule gitlink in the host repo (subject like "Bump ILSpy-tests: ...").

## Code conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icsharpcode/ILSpy](https://github.com/icsharpcode/ILSpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
