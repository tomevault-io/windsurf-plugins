---
trigger: always_on
description: A .NET solution for ingesting FHIR specification packages and exporting them into other languages/formats (TypeScript, C#/Firely, OpenAPI, Ruby, SQLite, FHIR Shorthand, CQL, Info, etc.).
---

# Copilot Instructions for fhir-codegen

A .NET solution for ingesting FHIR specification packages and exporting them into other languages/formats (TypeScript, C#/Firely, OpenAPI, Ruby, SQLite, FHIR Shorthand, CQL, Info, etc.).

## Toolchain

- **.NET 9 SDK** is required. Despite what `README.md` says, every project targets `net9.0` (see `fhir-codegen.props` and `*.csproj` files). The CI workflow pins `DOTNET_VERSION: '9'`.
- **C# `<LangVersion>14.0</LangVersion>`**, `Nullable enable`, `ImplicitUsings enable` are set globally in `fhir-codegen.props`.
- Solution file: `fhir-codegen.sln` at the repo root.
- Style is enforced via `.editorconfig` and `stylecop.json`. Notable: 4-space indent, CRLF, accessibility modifiers required, copyright header on every `.cs` file:
  ```csharp
  // <copyright file="Foo.cs" company="Microsoft Corporation">
  //     Copyright (c) Microsoft Corporation. All rights reserved.
  //     Licensed under the MIT License (MIT). See LICENSE in the repo root for license information.
  // </copyright>
  ```

## Build / Test / Run

```powershell
# Restore & build everything
dotnet build fhir-codegen.sln -c Release

# Full test suite (matches CI; skips tests that need external FHIR IG repos)
dotnet test --configuration Release --framework net9.0 --filter "RequiresExternalRepo!=true"

# Run a single test class or method (xUnit + Shouldly)
dotnet test src/Fhir.CodeGen.Lib.Tests/Fhir.CodeGen.Lib.Tests.csproj --filter "FullyQualifiedName~GenerationTests"
dotnet test src/Fhir.CodeGen.Lib.Tests/Fhir.CodeGen.Lib.Tests.csproj --filter "FullyQualifiedName=Fhir.CodeGen.Lib.Tests.GenerationTests.MyTest"

# Run the CLI from source
dotnet run --project src/fhir-codegen/fhir-codegen.csproj -- generate TypeScript -p hl7.fhir.r4.core --output-path ./R4.ts
```

Tests use **xUnit** + **Shouldly** (not FluentAssertions). The `RequiresExternalRepo=true` trait gates tests that clone the HL7 cross-version repos and is skipped in CI.

Tests rely on FHIR packages being present under `~/.fhir`. CI seeds the cache via `firely.terminal` (`fhir install hl7.fhir.r{2..5}.core ...`); locally, run `fhir-codegen` against the packages first or install `firely.terminal` to pre-populate the cache.

## Solution architecture

Projects under `src/` form a layered pipeline:

| Project | Role |
|---|---|
| `Fhir.CodeGen.Common` | Lightweight POCOs / shared models / polyfills. Minimal dependencies — keep it that way; other libs depend on it. |
| `Fhir.CodeGen.Packages` | FHIR package cache management (download, resolve, registry lookup). |
| `Fhir.CodeGen.CrossVersionLoader` | Load and reconcile artifacts across FHIR versions (R2/R3/R4/R4B/R5). |
| `Fhir.CodeGen.MappingLanguage` | FML (FHIR Mapping Language) parser/abstractions. |
| `Fhir.CodeGen.LangSQLite` / `Fhir.CodeGen.SQLiteGenerator` | SQLite export backend. |
| `Fhir.CodeGen.Lib` | Core engine: loader → normalized model → language exporters. Depends on Packages, CrossVersionLoader, MappingLanguage, LangSQLite, Common. |
| `Fhir.CodeGen.Comparison` | Package/artifact diffing. |
| `Fhir.CodeGen.CrossVersionExporter` | Produces cross-version artifacts. |
| `fhir-codegen` | `System.CommandLine`-based CLI. Shares code with `fhir-codegen-shared` (a Shared Project, imported via `.projitems`). |
| `performance-test-cli` | Standalone tooling. |

Tests live alongside their target: `Fhir.CodeGen.Lib.Tests`, `Fhir.CodeGen.MappingLanguage.Tests`, `Fhir.CodeGen.Packages.Tests`. Test data is under `Fhir.CodeGen.Lib.Tests/TestData/` and copied to output via `<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>`.

## Key conventions

### Language exporters (the main extension point)
- All exporters live under `src/Fhir.CodeGen.Lib/Language/{Name}.cs` (or `Language/{Name}/`) and implement `ILanguage`.
- They are auto-discovered by `LanguageManager` via reflection at static-init time — **do not** register them manually; just add the type.
- Each exporter exposes a nested `*Options : ConfigGenerate` class. Options are surfaced to the CLI via `[ConfigOption(ArgName = "--foo", Description = "...")]` attributes plus a matching `ConfigurationOption` static (with a `System.CommandLine.Option<T>`). Both must stay in sync.
- See `TypeScript.cs` and `Firely/`, `OpenApi/`, `Info/`, `Ruby/`, `SQLite/`, `Shorthand/`, `Cql/` as canonical examples.

### Multi-version FHIR via extern aliases
Different FHIR major versions ship as separate `Hl7.Fhir.*` assemblies whose types collide. The solution uses MSBuild aliasing so they can coexist:
- `fhir-codegen.csproj` aliases `Hl7.Fhir.STU3 → coreR3`, `Hl7.Fhir.R4 → coreR4` (see the `AddPackageAliases` target).
- `Fhir.CodeGen.Lib.Tests.csproj` uses aliases `stu3`, `r4`, `r4b`.
- When you reference these versioned types, use `extern alias` and qualify accordingly. Don't add a top-level `using Hl7.Fhir.Model;` for these.

### Coding style preferences (from custom instructions)
- Prefer **explicit types** in C#; use `var` only when the type is obvious from the right-hand side.
- Use **collection expressions `[]`** for empty initializers, not `new List<T>()` / `new()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FHIR/fhir-codegen](https://github.com/FHIR/fhir-codegen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
