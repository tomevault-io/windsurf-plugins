---
trigger: always_on
description: SG4MVC is a **Roslyn incremental source generator** for ASP.NET Core MVC/Razor Pages. It analyzes a project at compile-time and generates strongly-typed static helper classes that replace magic strings in route and view references:
---

# SG4MVC Copilot Instructions

## What This Project Is

SG4MVC is a **Roslyn incremental source generator** for ASP.NET Core MVC/Razor Pages. It analyzes a project at compile-time and generates strongly-typed static helper classes that replace magic strings in route and view references:

```csharp
// Without SG4MVC
Html.ActionLink("Details", "Dinners", new { id = 123 })

// With SG4MVC
Html.ActionLink(MVC.Dinners.Details(123))
```

There are two NuGet packages:
- **`Sg4Mvc`** (`src/Sg4Mvc/`) — runtime library (tag helpers, action result wrappers, model unbinders)
- **`Sg4Mvc.Generator`** (`src/Sg4Mvc.Generator/`) — the source generator itself (targets `netstandard2.0`, shipped as an analyzer)

Activation in a consuming project requires `[assembly: GenerateSg4Mvc]`.

## Build and Test

```sh
dotnet build
dotnet test

# Single test by fully qualified name
dotnet test --filter "Sg4Mvc.Test.ControllerDefinitionTests.FullyQualifiedName_Default"

# Single test class
dotnet test --filter "ClassName=ClassBuilderTests"

# Pack NuGet packages
dotnet pack --configuration Release
```

## Architecture

### Source Generation Pipeline (`Sg4Mvc.Generator`)

```
Compilation
  → ControllerTargetFilter / PageTargetFilter / AdditionalTextsTargetFilter
  → *DefinitionTransform   (build ControllerDefinition / PageDefinition models)
  → DataGroupingService    (correlate controllers ↔ views ↔ static files)
  → GeneratorServiceFactory
  → Sg4MvcGeneratorService (generate partial controllers + MVC/MVCPages/Links helpers)
  → CodeGen layer          (ClassBuilder → MethodBuilder → BodyBuilder → SyntaxFactory)
```

The entry point is `Generator` (implements `IIncrementalGenerator`, attributed `[Generator(LanguageNames.CSharp)]`).

### Key Abstractions

| Interface | Role |
|-----------|------|
| `IControllerGeneratorService` | Generates partial controller class + `MVC.*` helpers |
| `IPageGeneratorService` | Generates `MVCPages.*` helpers |
| `IStaticFileGeneratorService` | Generates `Links.*` helpers |
| `IViewLocator` / `IPageViewLocator` | Discovers `.cshtml` files |
| `IControllerRewriterService` | Rewrites action method bodies |
| `IModelUnbinder` / `IModelUnbinderProvider` | Convert models → route values (runtime) |

### Generated Output Shape

For each controller, the generator emits a `<FullyQualifiedName>.generated.cs` file containing:
1. A `partial` class matching the original controller
2. A nested static class under `MVC.<ControllerName>` (or `MVCPages.<PageName>`) with strongly-typed action calls, `Views`, and `Actions` sub-classes

### Settings

`Settings` is a POCO loaded via `AnalyzerConfigOptionsProvider`. Key properties:

| Property | Default | Effect |
|----------|---------|--------|
| `HelpersPrefix` | `"MVC"` | Top-level controller helper class name |
| `PageHelpersPrefix` | `"MVCPages"` | Top-level page helper class name |
| `Sg4MvcNamespace` | `"Sg4Mvc"` | Root namespace for generated files |
| `LinksNamespace` | `"Links"` | Static file helper class name |
| `StaticFilesPath` | `"wwwroot"` | Where to scan for static files |
| `FeatureFolders` | `false` | Enable feature-folder view discovery |

## Key Conventions

### Code Style (enforced via `.editorconfig`)
- CRLF line endings
- Private fields: `_camelCase`
- Constants and statics: `PascalCase`

### CodeGen Layer (`src/Sg4Mvc.Generator/CodeGen/`)
Use the fluent builder classes—never write raw `SyntaxFactory` calls directly in generator services:
```csharp
new ClassBuilder("HomeController")
    .WithModifiers("public", "partial")
    .WithMethod(new MethodBuilder("Index").WithReturnType("IActionResult") ...)
    .Build();
```

### Test Structure
- Unit tests live in `test/Sg4Mvc.Test/` and mirror the source namespace structure
- Integration tests live in `test/AspNetSimple.Test/` and exercise the full generation pipeline against sample apps
- Use **xUnit** (`[Fact]` / `[Theory]` / `[InlineData]`) and **Moq** for mocking

### View Discovery Patterns
- **Default**: `Views/<Controller>/`, `Areas/<Area>/Views/<Controller>/`
- **Feature folders**: `Features/<Feature>/Views/` (opt-in via `Settings.FeatureFolders`)
- Extend by implementing `IViewLocator` / `IPageViewLocator`

### Excluding from Generation
- `[Sg4MvcExclude]` on a controller or action suppresses generation for that target
- `[NonAction]` on a method causes it to be skipped
- Non-public and abstract controllers are ignored automatically

### Incremental Generator Performance
Filters (`*TargetFilter`) and transforms (`*DefinitionTransform`) must be **pure functions** — no side effects, no I/O — so Roslyn can cache and replay them safely.

## Performance Benchmarking

The project includes a BenchmarkDotNet suite in `test/Sg4Mvc.Benchmarks/` for measuring generator performance and detecting regressions.

### Benchmark Classes

| Class | What it measures |
|-------|-----------------|
| `FullPipelineBenchmarks` | End-to-end generator execution via `CSharpGeneratorDriver` at 3 scale profiles (Small/Medium/Large) |
| `CodeGenBenchmarks` | Individual phases: `GeneratePartialController`, `GenerateSg4Controller`, `CodeFileBuilder.Build` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SG4MVC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
