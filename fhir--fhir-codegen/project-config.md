---
trigger: always_on
description: Canonical, machine-readable conventions for automated agents working in
---

# AGENTS.md

Canonical, machine-readable conventions for automated agents working in
**fhir-codegen**. This file is the single source of truth that the
`.github/skills/dev-*` skills read before naming any build, test, or lint
command.

**Precedence.** This file is authoritative for commands, conventions, and
invariants an agent must follow. [`README.md`](README.md) and
[`docs/`](docs/) are authoritative for rationale, configuration reference,
and operational detail, and are the place to look for the "why". If this
file contradicts the repository itself, the repository wins — fix this file.

---

## What this repository is

`fhir-codegen` ingests FHIR specification packages (NPM-style `hl7.fhir.*`
packages) and exports them into other languages and formats — TypeScript,
C#/Firely, OpenAPI, Ruby, SQLite, FHIR Shorthand, CQL, Info, and a
cross-version mapping pipeline.

The framing fact: this is a **code generator whose output is consumed by
other projects**. Output shape is the product. A change that alters
generated artifacts is a breaking change for downstream consumers even when
the C# compiles cleanly, so generation tests and their expected output carry
more weight here than they would in an ordinary library.

The libraries also ship as NuGet packages (see `fhir-codegen.props`), so
public API changes in `Fhir.CodeGen.*` are breaking changes.

---

## Repository layout

| Path | Contents |
|-|-|
| `src/fhir-codegen/` | `System.CommandLine`-based CLI (`OutputType=Exe`). |
| `src/fhir-codegen-shared/` | Shared Project (`.projitems`) imported by the CLI. |
| `src/Fhir.CodeGen.Common/` | Lightweight POCOs, shared models, polyfills. Dependency-light by design. |
| `src/Fhir.CodeGen.Packages/` | FHIR package cache management (download, resolve, registry lookup). |
| `src/Fhir.CodeGen.CrossVersionLoader/` | Load and reconcile artifacts across R2/R3/R4/R4B/R5. |
| `src/Fhir.CodeGen.MappingLanguage/` | FML (FHIR Mapping Language) parser/abstractions. |
| `src/Fhir.CodeGen.LangSQLite/`, `src/Fhir.CodeGen.SQLiteGenerator/` | SQLite export backend. |
| `src/Fhir.CodeGen.Lib/` | Core engine: loader → normalized model → language exporters. |
| `src/Fhir.CodeGen.Lib/Language/` | **The main extension point** — one `ILanguage` implementation per output format. |
| `src/Fhir.CodeGen.Comparison/` | Package/artifact diffing. |
| `src/Fhir.CodeGen.CrossVersionExporter/` | Produces cross-version artifacts. |
| `src/performance-test-cli/` | Standalone perf tooling. |
| `src/*.Tests/` | xUnit test projects, colocated with their target. |
| `src/Fhir.CodeGen.Lib.Tests/TestData/` | Test fixtures, copied to output via `PreserveNewest`. |
| `docs/articles/`, `docs/specs/` | Narrative docs and per-step pipeline specs. |
| `docfx/` | Documentation site generation. |
| `languageInput/` | Hand-maintained input assets for certain exporters. |

**Ignored paths** (see `.gitignore`): `/scratch`, `/generated`, `/temp`,
`/firely`, `/cytoscape`, `/fhirVersions` contents, `*.sqlite`. Nothing under
`/scratch` is ever committed.

---

## Toolchain pins

- **.NET 9 targeting pack is required.** There is **no `global.json`**, so
  the SDK version is a *floor*, not an exact pin — any SDK that can target
  `net9.0` works. CI pins `DOTNET_VERSION: '9'`
  (`.github/workflows/build-and-test.yml`); a .NET 10 SDK building `net9.0`
  is also known-good locally.
- **Every project targets `net9.0`** except `Fhir.CodeGen.SQLiteGenerator`,
  which targets **`netstandard2.0`**. Do not "fix" that one to `net9.0`.
- `fhir-codegen.props` (imported by the project files) sets
  **`LangVersion 14.0`**, **`Nullable enable`**, **`ImplicitUsings enable`**
  solution-wide. Change these there, not per-project.
- Versions are declared **per-project** in each `.csproj`. There is no
  central package management and no lock file, so a dependency bump must be
  applied consistently across every project that references the package
  (the `Hl7.Fhir.*` family is currently `5.13.3` everywhere).
- **Warnings are not errors.** No project sets `TreatWarningsAsErrors`,
  `EnforceCodeStyleInBuild`, `AnalysisLevel`, or `AnalysisMode`.
- Tests need the **FHIR package cache** populated — see "Test" below.

---

## Build

```powershell
dotnet build fhir-codegen.sln -c Release
```

Scoped to a single project:

```powershell
dotnet build src/Fhir.CodeGen.Lib/Fhir.CodeGen.Lib.csproj -c Release
```

The expected baseline is **0 errors, 1 warning**. The one warning is
pre-existing and unrelated to any current work:

```
src/Fhir.CodeGen.Lib/SqlOnFhir/ViewDefinition.cs(159,40): warning CS3021:
'ViewDefinition.ConstantComponent.Value' does not need a CLSCompliant
attribute because the assembly does not have a CLSCompliant attribute
```

Anything beyond that should be investigated before it is attributed —
confirm against a clean checkout or `HEAD` before calling it a regression.

There is a **single build track**. `fhir-codegen` and `performance-test-cli`
are `Exe`; everything else is a library. No AOT, native, or publish-only
check exists.

---

## Test

**xUnit 2.9.3** with **Shouldly 4.3.0** for assertions — *not*
FluentAssertions. The runner is **VSTest**
(`Microsoft.NET.Test.Sdk 17.14.1` + `xunit.runner.visualstudio 3.1.5`);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FHIR/fhir-codegen](https://github.com/FHIR/fhir-codegen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
