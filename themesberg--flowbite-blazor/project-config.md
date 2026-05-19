---
trigger: always_on
description: - Flowbite Blazor is a Blazor component library that ports Flowbite React to ASP.NET Blazor 8/9 on top of Tailwind CSS.
---

# Cline and Claude Project Rules for Flowbite Blazor

## Overview
- Flowbite Blazor is a Blazor component library that ports Flowbite React to ASP.NET Blazor 8/9 on top of Tailwind CSS.
- Current status: early development (`v0.0.x-alpha`); expect API and package changes.
- Work from the `develop` branch for new changes and pull requests.

## Projects
- `src/Flowbite/` — core component library.
- `src/Flowbite.ExtendedIcons/` — optional icon packs.
- `src/DemoApp/` — documentation playground; mirror every new component with a demo page.
- Place shared docs under `docs/`, automation in `scripts/`, and ship-ready static assets under each project’s `wwwroot/`.

## Build, Run, and Packaging

Use the Python automation script for all build operations:

### Build & Run Commands
- `python build.py` — Build the solution (default)
- `python build.py build` — Same as above, builds FlowbiteBlazor.sln
- `python build.py watch` — Run DemoApp with hot reload (foreground, Ctrl+C to stop)
- `python build.py run` — Run DemoApp in foreground
- `python build.py start` — Auto-builds, then starts DemoApp in background (http://localhost:5290)
- `python build.py stop` — Stop background DemoApp
- `python build.py status` — Check if DemoApp is running

**Key Behaviors:**
- `build` auto-stops any running DemoApp (prevents file lock errors)
- `start` auto-builds before launching (always runs latest code)
- Tailwind CSS is auto-downloaded to `tools/` on first build

### Package Commands
- `python build.py pack` — Create NuGet packages in `nuget-local/`
- `python build.py publish` — Pack NuGet + publish DemoApp to `dist/`

### Test Commands
- `python build.py test` — Run unit tests (excludes integration tests)
- `python build.py test <filter>` — Run tests matching filter (e.g., `DebouncerTests`)
- `python build.py test-integration` — Run Playwright smoke tests (auto-starts/stops DemoApp)
- `python build.py test-publish` — Run publish to catch pre-rendering errors (validates all pages)
- `python build.py test-all` — Run all tests (unit + publish + integration)

### Log Commands (for debugging)
- `python build.py log` — Show last 50 lines of demoapp.log
- `python build.py log <pattern>` — Search log for regex pattern (case-insensitive)
- `python build.py log --tail <n>` — Show last n lines
- `python build.py log --level error` — Filter by log level (error/warn/info/debug)

### Manual Alternatives (if needed)
- Direct build: `dotnet build FlowbiteBlazor.sln`
- Direct watch: `dotnet watch --project src/DemoApp/DemoApp.csproj`
- Manual Tailwind: `tools/tailwindcss -i src/DemoApp/wwwroot/css/app.css -o src/DemoApp/wwwroot/css/app.min.css --minify --postcss`
- Regenerate docs context: `powershell -ExecutionPolicy Bypass -File Build-LlmsContext.ps1` inside `src/DemoApp/`

## Architecture and Component Patterns
- Base classes live in `src/Flowbite/Base/`:
  - `FlowbiteComponentBase` provides `CombineClasses()`, `MergeClasses()`, and a `Class` parameter.
  - `IconBase` extends the base for SVG icons with aria and stroke control.
  - `OffCanvasComponentBase` manages visibility for drawers, modals, and toasts.
- **CSS Class Composition Patterns:**
  - **PREFER `ElementClass` fluent builder** for component class logic (`src/Flowbite/Utilities/ElementClass.cs`)
  - Use `ElementClass.Empty().Add("class").Add("conditional", when: bool)` for readable conditional classes
  - Pass result to `MergeClasses()` for TailwindMerge.NET conflict resolution
  - Example: `MergeClasses(ElementClass.Empty().Add("px-4").Add("hidden", when: !visible).Add(Class))`
- Components use a two-file pattern: `Component.razor` for markup and `Component.razor.cs` for logic.
- Services for programmatic control (`AddFlowbite*`) reside in `src/Flowbite/Services/`; register them in `Program.cs`.
- DemoApp structure:
  - Pages under `src/DemoApp/Pages/Docs/components/`.
  - Sidebar data in `src/DemoApp/Layout/DocLayoutSidebarData.cs`.
  - AI documentation snippets in `src/DemoApp/wwwroot/llms-docs/sections/`.
- Debug builds link directly to projects; Release builds use locally packed NuGet packages.

## Development Conventions
- Follow `.editorconfig`: 4-space indentation, file-scoped namespaces, PascalCase public APIs, `_camelCase` private fields.
- Keep C# logic in `.cs` files via partial classes; parameters are public properties with `[Parameter]`.
- Use `[CaptureUnmatchedValues]` for additional HTML attributes, `RenderFragment? ChildContent` for slots, and prefer enums for style variations.
- Always apply `@key` when looping components with `@foreach`.
- Use Tailwind utility classes exclusively; ensure dark mode coverage with `dark:` variants and accept a `Class` parameter for custom styling.
- Only use icons from `Flowbite.Icons` or `Flowbite.ExtendedIcons`; add missing glyphs internally.
- Document all public APIs with XML comments.

## UI Assets & Theming Tips
- Tailwind config lives in directory for Flowbite at `src\Flowbite\tailwind.config.js`; PostCSS in `src\Flowbite\postcss.config.js`.
- Tailwind config lives in directory for DemoApp at `src\DemoApp\tailwind.config.js`; PostCSS in `src\DemoApp\postcss.config.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [themesberg/flowbite-blazor](https://github.com/themesberg/flowbite-blazor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
