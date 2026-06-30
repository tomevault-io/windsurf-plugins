---
trigger: always_on
description: - This repo is a .NET 10 solution rooted at `Polymerium.slnx`.
---

# AGENTS

## Repo Shape

- This repo is a .NET 10 solution rooted at `Polymerium.slnx`.
- `src/Polymerium.Avalonia` is the only app in this repo.
- `submodules/Trident.Net` is a git submodule and is part of the solution build. Treat it as an integral part of this project: it participates in the same development workflow and should be edited freely alongside the main codebase. Do not treat submodule changes as out-of-scope — feel free to modify files under `submodules/Trident.Net` when the task requires it. `Huskui.Avalonia` is consumed as a NuGet package, not a submodule.
- Fresh clones need submodules initialized: `git submodule update --init --recursive`.
- `plans/` holds task blueprints — independent design docs written so a fresh session can pick up a task from the plan alone, without re-deriving the decisions or reconstructing progress from code. See `plans/README.md` for the writing conventions; treat `plans/archived/` as a graveyard (no reference value, do not read).

## Documentation Website

- The project's public-facing docs site lives at `website/` — a Next.js app built with [Fumadocs](https://fumadocs.dev).
- Deployed on Vercel at **polymerium.dearain.dev**.
- Content is written in MDX under `website/content/docs/`, organized into sections: `getting-started`, `concepts`, `managing`, `advanced`, `guides`, `comparisons`.
- Every `.mdx` page has a Chinese counterpart (`.zh.mdx`). When editing content, update both files.
- Navigation structure is defined per-section via `meta.json` (and `meta.zh.json`).
- Dev server: `cd website && npm run dev` → opens at `http://localhost:3000`.
- Build: `cd website && npm run build`. Post-build syncs search index to Algolia via `scripts/sync-algolia.mjs`.

## Verified Commands

- Full solution build: `dotnet build "Polymerium.slnx"`
- Focused app build: `dotnet build "src/Polymerium.Avalonia/Polymerium.Avalonia.csproj"`
- There are no test projects in this repo right now. `dotnet test "Polymerium.slnx"` is not a meaningful verification step; use build plus targeted checks instead.
- **Do NOT run any formatting tools** (`scripts/Format-Files.ps1`, `csharpier`, `xstyler`, etc.). They operate across the entire repo including submodules and will produce unintended changes, and can also corrupt parts of the code. Only the user may invoke formatting.

## Architecture Entry Points

- App bootstrap starts in `src/Polymerium.Avalonia/Program.cs`.
- DI wiring lives in `src/Polymerium.Avalonia/Startup.cs`.
- Window construction, global exception hooks, and startup of lifetime services live in `src/Polymerium.Avalonia/App.axaml.cs`.
- The first navigation goes to `LandingPage`; shell-level state, notifications, OOBE, and update prompts are coordinated from `src/Polymerium.Avalonia/MainWindowContext.cs`.

## Directory Layout

Under `src/Polymerium.Avalonia/`, directories are organized by role. View + ViewModel pairs always live in sibling `Xxx/` + `XxxModels/` folders and are paired by naming convention (see ViewModel Mechanism):

- `Pages/` + `PageModels/` — full-screen pages and their view models (the navigated content).
- `Dialogs/` + `DialogModels/` — modal dialogs (centered, blocking).
- `Modals/` + `ModalModels/` — modal overlays (non-blocking, cover the host).
- `Sidebars/` + `SidebarModels/` — drawer sidebars (slide in from an edge).
- `Toasts/` — transient toast notifications. There is no `ToastsModels/` folder **yet**, so existing toasts are still constructed inline and passed to `OverlayService.PopToast(Toast)`. The mechanism is fully wired, though: `OverlayService.PopToast<TToast>(parameter)` is provided and routes through the same activator as other overlays — add `ToastsModels/` + a `ToastModel` the moment a toast needs a view model.
- `Components/` / `Controls/` / `Widgets/` — reusable Avalonia controls, grouped by scope (larger composite components vs. small atomic widgets).
- `Services/` — application services (navigation, overlay, data, persistence, instance management, etc.).
- `Repositories/` — data access / storage adapters.
- `Snapshots/` — snapshot/version store for instances.
- `Facilities/` — framework-level glue (base classes, activators, mixins, persistence helpers). `ViewModelBase`, `SimpleViewActivator`, `SimpleViewStatePersistence` live here.
- `Converters/` / `Rendering/` / `Themes/` — value converters, custom rendering helpers, and styling/ControlThemes.
- `Models/` — shared data models (DTOs/entities) consumed by views, view models, and services; these get their own files, not nested types.
- `Utilities/` — stateless helpers and extension methods.
- `Migrations/` — database migration definitions.
- `Assets/` / `Properties/` / `Exceptions/` — static assets, `.resx` localization, and domain exception types.

App-level files at the project root: `Program.cs` (entry), `Startup.cs` (DI), `App.axaml(.cs)` (window/lifetime), `MainWindow.axaml(.cs)` + `MainWindowContext.cs` (shell), `Configuration.cs`, `ErrorReporter.cs`, `AppBuilderExtensions.cs`.

## ViewModel Mechanism

Built on **CommunityToolkit.Mvvm** (source-generator-based) + the **Huskui.Avalonia** activation system, with **DynamicData** for reactive collections. There is **no** ReactiveUI and **no** string-based routing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d3ara1n/Polymerium](https://github.com/d3ara1n/Polymerium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
