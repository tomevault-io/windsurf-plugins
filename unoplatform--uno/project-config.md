---
trigger: always_on
description: This file provides guidance to AI Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI Agents when working with code in this repository.

## Project Overview

Uno Platform is an open-source .NET UI cross-platform framework for building .NET applications from a single codebase using the WinUI 3 API. It targets Web (WebAssembly), Desktop (Windows, macOS, Linux via Skia), and Mobile (iOS, tvOS, Android).

**Reference these instructions first**, then lean on skills (workflows) and the path-scoped rules below.

#### Claude Code Skills (invoke via `/skill-name`)

| Skill | Command | Use For |
|-------|---------|---------|
| Add Sample | `/add-sample` | Creating SamplesApp sample pages with correct registration |
| Runtime Tests | `/runtime-tests` | Building and running Uno runtime tests (Skia Desktop/WASM) |
| WinUI Runtime Tests | `/winui-runtime-tests` | Running runtime tests against native WinUI on Windows |
| WinUI Porting | `/winui-port` | Porting WinUI C++ code to Uno Platform C# (full deep reference) |
| DevServer | `/devserver` | DevServer CLI/Host build, test, MCP proxy, add-in discovery |
| Docs Build | `/docs-build` | Building, previewing & validating the docs website (DocFX), incl. external-doc commit bumps in `import_external_docs.ps1` |

#### Pre-commit review (invoke via `/review-panel`)

`/review-panel [scope]` runs an eight-lens reviewer panel (architect, contract, skeptic, performance, operability, quality, security, jerome) in parallel and synthesizes one report with a `ship` / `fix-first` / `block-merge` verdict. Run it before you commit or open a PR — pass a scope (`master..HEAD`, a `#PR`, `HEAD~1`) or omit it to auto-detect uncommitted changes / branch-vs-`master`. The panel learns from corrections recorded in `specs/lessons.md`. Lenses, scopes, and loop recipes: `.claude/review-panel-cheatsheet.md`.

#### Path-scoped rules (`.claude/rules/`)

These load **automatically** when you touch matching files — you don't invoke them. They hold the non-obvious, subsystem-specific conventions so this always-loaded file stays lean:

| Rule | Applies to | Covers |
|------|-----------|--------|
| `code-style.md` | `src/**/*.cs` | nullable, file headers (MUX/MIT), logging, `[Uno.NotImplemented]` |
| `platform-targeting.md` | `src/**/*.cs` | file-suffix vs `#if` vs `OperatingSystem.IsX()` vs `ApiExtensibility` |
| `debugging-discipline.md` | `src/**/*.cs` | full root-cause/validation/diagnosis-bias protocols |
| `dependency-properties.md` | `src/Uno.UI/**` | `[GeneratedDependencyProperty]`, metadata, callbacks |
| `runtime-tests.md` | `src/Uno.UI.RuntimeTests/**` | `[RunsOnUIThread]`, `[PlatformCondition]`, `UITestHelper` |
| `unit-tests.md` | `src/Uno.UI.UnitTests/**` | MSTest, no-visual-tree logic tests |
| `source-generators.md` | `src/SourceGenerators/**` | incremental gens, LOH/perf, cancellation |
| `samples.md` | `src/SamplesApp/**` | `[Sample]`, theming, XamlStyler |
| `build-system.md` | `src/**/*.{csproj,props,targets}` | TFMs, output paths, package versions |

**Which to reach for:** the relevant `.claude/rules/*.md` is already in context (path-scoped) — use it as the checklist. Use a `/skill` for the actual build/run/scaffold/port workflow (and its deep reference).

---

## Quick Reference

### Technology Stack

| Technology | Purpose |
|------------|---------|
| .NET 10.0/9.0 | Multi-target framework |
| C# & XAML | Primary languages |
| TypeScript | WebAssembly/Web APIs only |
| Skia | Cross-platform rendering |
| MSBuild | Build orchestration |
| Roslyn | Source generators |

### Platform File Suffixes

| Suffix | Platform |
|--------|----------|
| `.Android.cs` | Android |
| `.iOS.cs` | iOS |
| `.UIKit.cs` | iOS & tvOS |
| `.wasm.cs` | WebAssembly |
| `.skia.cs` | Skia |
| `.reference.cs` | Reference implementation |
| `.crossruntime.cs` | Skia + WebAssembly + Reference (shared) |

### Key Source Directories

- `src/Uno.UI/` - Core UI framework (WinUI controls, layout, XAML runtime)
- `src/Uno.UWP/` - Non-UI WinRT APIs (platform-specific assemblies)
- `src/Uno.Foundation/` - Foundation APIs (platform-specific assemblies)
- `src/Uno.UI.Runtime.Skia.*/` - Skia platform runtimes
- `src/SourceGenerators/` - XAML parser, DependencyProperty generator
- `src/SamplesApp/` - Sample app for validation and tests
- `src/Uno.UI.RuntimeTests/` - Platform runtime tests
- `src/Uno.UI.DevServer.Cli/` - DevServer CLI tool
- `src/Uno.UI.RemoteControl.Host/` - DevServer Host process

### Build Setup (Required)

**1. Setup cross-targeting override:**
```bash
cd src
cp crosstargeting_override.props.sample crosstargeting_override.props
```

**2. Edit `crosstargeting_override.props`** (recommended fast-iteration config):
```xml
<Project>
  <PropertyGroup>
    <!-- Choose ONE target: -->
    <UnoTargetFrameworkOverride>net10.0</UnoTargetFrameworkOverride>              <!-- WebAssembly/Skia -->
    <!-- <UnoTargetFrameworkOverride>net10.0-android</UnoTargetFrameworkOverride>  Android -->
    <!-- <UnoTargetFrameworkOverride>net10.0-ios</UnoTargetFrameworkOverride>      iOS -->
    <!-- <UnoTargetFrameworkOverride>net10.0-windows10.0.19041.0</UnoTargetFrameworkOverride> Windows -->

    <!-- Disables analyzers + code-style enforcement for local builds. No effect on CI. -->
    <UnoFastDevBuild>true</UnoFastDevBuild>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unoplatform/uno](https://github.com/unoplatform/uno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
