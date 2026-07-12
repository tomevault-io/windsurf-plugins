---
trigger: always_on
description: This file helps AI coding agents work productively in this repository after the dependency-oriented refactor.
---

# AGENTS.md

## Scope

This file helps AI coding agents work productively in this repository after the dependency-oriented refactor.

For deep architecture diagrams and node-level flow details, prefer linking to `README.md` instead of duplicating content.

## Project Boundary Rule (Critical)

- Do not mix repository identity, product scope, and library namespaces as if they are the same project.
- The repository and solution file are still named `VisualCutterForm`, but the old `VisualCutterForm/VisualCutterForm.csproj` application module is deprecated.
- The active architecture now has three scopes:
  - Deprecated legacy app scope: `VisualCutterForm/...`
  - New dependency-oriented VisualMaster product scope: `VisualMaster.Application/...`, `VisualMaster.Api/...`, `VisualMaster.Impl/...`, `VisualMaster.UI/...`
  - Independent VisualMaster capability modules: `VisualMaster.CameraLink/...`, `VisualMaster.Communication/...`, and other sibling module projects
- `VisualMaster.Application`, `VisualMaster.Api`, `VisualMaster.Impl`, and `VisualMaster.UI` are not just ordinary sibling capability modules; together they are the new dependency-oriented application/product layer.
- `VisualMaster.CameraLink` and `VisualMaster.Communication` remain capability modules and should stay distinct from the new product-layer projects.
- When describing architecture, always state whether a path/class belongs to:
  - Deprecated legacy app module (`VisualCutterForm/...`)
  - New product-layer projects (`VisualMaster.Application/...`, `VisualMaster.Api/...`, `VisualMaster.Impl/...`, `VisualMaster.UI/...`)
  - VisualMaster capability modules (`VisualMaster.CameraLink/...`, `VisualMaster.Communication/...`)
- Never say "VisualMaster project" to mean the whole repository or solution.

## Current Solution Structure

- Tech stack: C# 7.3, .NET Framework 4.8, WinForms + WPF, solution format `.slnx`.
- Projects contained in solution `VisualCutterForm.slnx`:
  - `VisualMaster.Application/VisualMaster.Application.csproj` (new application composition root; owns host building, startup configuration, and module assembly)
  - `VisualMaster.Api/VisualMaster.Api.csproj` (new API/abstraction boundary for dependency-oriented product layer)
  - `VisualMaster.Impl/VisualMaster.Impl.csproj` (new implementation layer; depends on `VisualMaster.Api`)
  - `VisualMaster.UI/VisualMaster.UI.csproj` (new WPF UI component library; depends on `VisualMaster.Api`)
  - `VisualMaster.Config.Abstractions/VisualMaster.Config.Abstractions.csproj` (configuration abstractions)
  - `VisualMaster.CameraLink/VisualMaster.CameraLink.csproj` (WPF camera capability module)
  - `VisualMaster.Communication/VisualMaster.Communication.csproj` (WPF communication capability module)
  - `VisualMaster.CameraLink.TestApp/VisualMaster.CameraLink.TestApp.csproj` (camera sample app)
  - `VisualMaster.CameraLink.TestApp.Viewer/VisualMaster.CameraLink.TestApp.Viewer.csproj` (camera viewer sample)
  - `VisualMaster.Communication.TestApp/VisualMaster.Communication.TestApp.csproj` (communication sample app)
  - `VisualCutterForm/VisualCutterForm.csproj` (deprecated legacy WinForms app module)
  - `SetupVisualCutter/SetupVisualCutter.vdproj` (installer)

## Important Refactor Reality

- The old `VisualCutterForm` app module is deprecated. Do not treat it as the active main application unless the user explicitly asks for legacy work.
- New feature work should default to the dependency-oriented product layer:
  - Put application startup, Host construction, bootstrap configuration, and module assembly in `VisualMaster.Application`.
  - Put public contracts, interfaces, DTOs, and composition-facing abstractions in `VisualMaster.Api`.
  - Put concrete services, orchestration, adapters, and non-UI implementation in `VisualMaster.Impl`.
  - Put reusable WPF controls, views, view models, resources, and themes in `VisualMaster.UI`.
- Keep capability-specific hardware/runtime logic in the relevant capability modules:
  - Camera-specific code belongs in `VisualMaster.CameraLink`.
  - Communication-specific code belongs in `VisualMaster.Communication`.
- Do not collapse the new product layer and capability modules into one conceptual "VisualMaster" bucket when discussing ownership or making edits.
- There is no separate `VisualMaster.Forms` or `VisualMaster.WorkFlow` project.

## Dependency Direction

- Preferred product-layer direction:
  - `VisualMaster.Application` -> `VisualMaster.Api`, `VisualMaster.Impl`, `VisualMaster.UI`, and selected capability modules
  - `VisualMaster.Impl` -> `VisualMaster.Api`
  - `VisualMaster.UI` -> `VisualMaster.Api`
- `VisualMaster.Api` should stay lightweight and should not depend on UI or implementation projects.
- `VisualMaster.UI` is a component library and should not own Host construction, application startup, or module assembly.
- Capability modules should remain reusable and should not depend on the deprecated `VisualCutterForm` module.
- Avoid introducing references from capability modules back into `VisualMaster.UI` unless the user explicitly asks for a larger architecture change.

## Build and Run

- Build all projects:
  - `msbuild VisualCutterForm.slnx /p:Configuration=Debug`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AyalaKaguya/VisualCutterForm](https://github.com/AyalaKaguya/VisualCutterForm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
