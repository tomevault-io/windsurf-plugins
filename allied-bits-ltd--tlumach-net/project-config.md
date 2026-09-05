---
trigger: always_on
description: **Tlumach.NET** (`AlliedBits.Tlumach` on NuGet) is a translation and localization library for .NET applications. It supports multiple file formats, runtime language switching, XAML framework integrations, and Roslyn-based compile-time code generation for type-safe translation access.
---

# Tlumach.NET — CLAUDE.md

## Project Overview

**Tlumach.NET** (`AlliedBits.Tlumach` on NuGet) is a translation and localization library for .NET applications. It supports multiple file formats, runtime language switching, XAML framework integrations, and Roslyn-based compile-time code generation for type-safe translation access.

**Target platforms:** Desktop (WPF, WinUI, WinForms, UWP), Mobile (MAUI, Avalonia), Web (Razor/Blazor), Console, and Server.

---

## Repository Layout

```
src/
  Tlumach.Main.sln              # Core packages only — use this for most work
  Tlumach.sln                   # Full solution including XAML framework integrations
  Tlumach/                      # Core library (TranslationManager, public API)
  Tlumach.Base/                 # Parsers, TranslationEntry, TranslationConfiguration
  Tlumach.Generator/            # Roslyn incremental code generator
  Tlumach.Extensions.Localization/  # Microsoft.Extensions.Localization adapter
  Tlumach.WPF/                  # WPF-specific integration
  Tlumach.WinUI/                # WinUI-specific integration
  Tlumach.MAUI/                 # MAUI-specific integration
  Tlumach.Avalonia/             # Avalonia-specific integration
  Tlumach.UWP/                  # UWP-specific integration
  Shared/                       # Shared MSBuild props and StyleCop config
tests/
  Tlumach.Tests.sln
  Tlumach.Tests/                # Main xUnit test suite
  Tlumach.GeneratorTests/       # Generator-specific tests
samples/                        # One sample project per supported platform/scenario
docs/                           # DocFX documentation source
```

---

## Build

```bash
# Core packages only (preferred for day-to-day work)
dotnet build src/Tlumach.Main.sln

# Full solution including XAML frameworks (requires platform SDKs)
dotnet build src/Tlumach.sln

# Release build
dotnet build src/Tlumach.Main.sln -c Release
```

Build configurations: `Debug`, `Release`, `SignedRelease`.
The `Release` configuration enables deterministic output (`ContinuousIntegrationBuild=true`) and embeds source files.

---

## Tests

```bash
# Run main test suite (as the CI pipeline does)
dotnet test tests/Tlumach.Tests/Tlumach.Tests.csproj -c Release

# Whole test solution (also works)
dotnet test tests/Tlumach.Tests.sln
```

Test files are in `tests/Tlumach.Tests/`. Each file format has its own `*ParserTests.cs`. Test data (sample translation files) are embedded resources under `tests/Tlumach.Tests/TestData/`.

---

## CI/CD

GitHub Actions workflow: `.github/workflows/build-test.yml`

- Trigger: push/PR to `main` or `release/*`
- Runner: `ubuntu-latest`, .NET 10.0.x
- Steps: build `Tlumach.Main.sln`, then run tests with `dotnet run`

---

## Key Architecture

### Core Components

| Component | Location | Purpose |
|---|---|---|
| `TranslationManager` | `src/Tlumach/TranslationManager.cs` | Central service; manages cultures, caching, events |
| `TranslationEntry` | `src/Tlumach.Base/TranslationEntry.cs` | Immutable translation unit (key, text, metadata) |
| `TranslationConfiguration` | `src/Tlumach.Base/TranslationConfiguration.cs` | Declarative `.cfg` file model |
| Parser base classes | `src/Tlumach.Base/` | `BaseParser`, `BaseJsonParser`, `BaseKeyValueParser`, `BaseTableParser`, `BaseXMLParser` |
| Concrete parsers | `src/Tlumach.Base/` | `JsonParser`, `ArbParser`, `IniParser`, `TomlParser`, `CsvParser`, `TsvParser`, `ResxParser` |
| Writer base classes | `src/Tlumach.Writers/` | `BaseWriter`, `BaseJsonWriter`, `BaseKeyValueWriter`, `BaseTableWriter`, `BaseXmlWriter` |
| Concrete writers | `src/Tlumach.Writers/` | `JsonWriter`, `IniWriter`, `TomlWriter`, `CsvWriter`, `TsvWriter`, `ResxWriter` |
| Code generator | `src/Tlumach.Generator/Generator.cs` | Roslyn incremental generator producing typed translation classes |
| ICU/placeholder engine | `src/Tlumach.Base/IcuFragment.cs` | Handles `{name}`, `{0}`, `plural`, `select`, `date`, etc. |

### Patterns

- **Strategy** — Parsers are registered with `Use()` and selected per format.
- **Observer** — `TranslationManager` exposes events: `CultureChanged`, `OnTranslationFileNotFound`, `OnReferenceNotResolved`, `OnPlaceholderValueNeeded`.
- **Incremental generation** — `Tlumach.Generator` uses Roslyn's incremental generator API; it targets `Microsoft.CodeAnalysis.CSharp` v5.0.0 for broad SDK compatibility.

### Supported File Formats

JSON, ARB, INI, TOML, CSV, TSV, ResX.

---

## Versioning

Managed by **Nerdbank.GitVersioning** (`version.json`).
Current version: `1.3.0.0-alpha`.
Release branches follow the pattern `release/v{version}`.

---

## Code Style

- Indentation: 4 spaces (enforced by `.editorconfig`)
- Line endings: CRLF (LF for shell scripts)
- Encoding: UTF-8
- Analyzers active in all builds: Roslynator, StyleCop, SonarAnalyzer, Meziantou, NetAnalyzers
- Analysis mode: `AllEnabledByDefault` — expect warnings to be treated as guidance, many suppressed selectively via `.editorconfig`
- StyleCop file headers use the company name defined in `Shared/stylecop.json`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Allied-Bits-Ltd/tlumach-net](https://github.com/Allied-Bits-Ltd/tlumach-net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
