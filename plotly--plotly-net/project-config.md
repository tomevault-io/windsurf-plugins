---
trigger: always_on
description: Guidance for AI coding agents working in the Plotly.NET repo. This is a first draft — please update it as conventions evolve.
---

# AGENTS.md

Guidance for AI coding agents working in the Plotly.NET repo. This is a first draft — please update it as conventions evolve.

## What this repo is

Plotly.NET is an interactive charting library for .NET, built on top of plotly.js. The core is written in F# and wraps the plotly.js JSON schema with multiple API layers (high-level type-safe `Chart` API down to low-level object manipulation). See [README.md](README.md) for user-facing docs and the [F1000Research paper](https://doi.org/10.12688/f1000research.123971.1) for design rationale.

Currently targeted plotly.js version: **2.27.1** (bundled at [src/Plotly.NET/plotly-2.27.1.min.js](src/Plotly.NET/plotly-2.27.1.min.js)).

## Packages (monorepo layout)

All shipped packages live under [src/](src/):

- **[Plotly.NET](src/Plotly.NET/)** — core F# library. Everything else depends on this.
- **[Plotly.NET.CSharp](src/Plotly.NET.CSharp/)** — idiomatic C# wrapper around the core API.
- **[Plotly.NET.Interactive](src/Plotly.NET.Interactive/)** — formatter extensions for .NET Interactive / Polyglot Notebooks.
- **[Plotly.NET.ImageExport](src/Plotly.NET.ImageExport/)** — static image (PNG/SVG/JPEG/PDF) rendering.
- **[Plotly.NET.Verso](src/Plotly.NET.Verso/)** — Verso integration.

Inside [src/Plotly.NET/](src/Plotly.NET/), the core is organized by concern:

- [ChartAPI/](src/Plotly.NET/ChartAPI/) — high-level `Chart` API (`Chart2D`, `Chart3D`, `ChartMap`, `ChartPolar`, etc.) and `GenericChart`.
- [Traces/](src/Plotly.NET/Traces/) — trace types (`Trace2D`, `Trace3D`, `TraceGeo`, …) matching plotly.js trace categories.
- [Layout/](src/Plotly.NET/Layout/) — layout objects (axes, legends, annotations, shapes, …).
- [CommonAbstractions/](src/Plotly.NET/CommonAbstractions/) — shared enums, style objects, and types used across traces/layouts.
- [Config/](src/Plotly.NET/Config/), [DisplayOptions/](src/Plotly.NET/DisplayOptions/), [Templates/](src/Plotly.NET/Templates/) — render-time configuration.
- [Globals.fs](src/Plotly.NET/Globals.fs), [Defaults.fs](src/Plotly.NET/Defaults.fs), [InternalUtils.fs](src/Plotly.NET/InternalUtils.fs) — globals and helpers.

## Build system — use FAKE, not `dotnet build` directly

The build is a FAKE script defined as an executable F# project at [build/build.fsproj](build/build.fsproj). It controls project ordering, version injection from `RELEASE_NOTES.md`, packing, and test execution. **Do not use `dotnet build` directly** — always go through the build project:

```shell
# Windows
./build.cmd                  # default target: Build
./build.cmd runTestsCore     # run a specific target

# Linux / macOS
./build.sh
./build.sh runTestsAll

# Or invoke the build project directly
dotnet run --project ./build/build.fsproj
dotnet run --project ./build/build.fsproj -- RunTestsExtensionLibs
```

Key build files:

- [build/ProjectInfo.fs](build/ProjectInfo.fs) — registry of all projects (library + test). **New projects must be added here**, both in the `projects` list and in the relevant test-project lists.
- [build/Build.fs](build/Build.fs) — main build targets, `sourceFiles` glob. New projects may also need to be registered in the glob.
- [build/TestTasks.fs](build/TestTasks.fs) — test build and run targets (see below).
- [build/PackageTasks.fs](build/PackageTasks.fs), [build/ReleaseTasks.fs](build/ReleaseTasks.fs) — pack and release pipeline.
- [build/DocumentationTasks.fs](build/DocumentationTasks.fs) — docs build targets.

Every shipped project needs its own `RELEASE_NOTES.md` — the build parses the top entry to derive the package version and assembly version.

## Tests

See [tests/README.md](tests/README.md) for the authoritative overview. Short version:

### Frameworks
- F# test projects: **Expecto** (majority) or xUnit.
- C# test projects: **xUnit**.
- JS test projects: **Mocha** (run in node; primarily validate generated JSON against `Plotly.validate`).

### Layout ([tests/](tests/))
- [Common/](tests/Common/) — `FSharpTestBase` and `CSharpTestBase` class libraries containing shared `TestUtils`, `TestCharts`, and helpers (`substringIsInChart`, `chartGeneratedContains`, `getFullPlotlyJS`, …). Test projects reference these.
- [CoreTests/](tests/CoreTests/) — largest suite. Tests HTML generation, JSON serialization, object validity for the core `Plotly.NET` library. Includes `CoreTests.fsproj` and `CSharpInteroperabilityTests.csproj`.
- [ExtensionLibsTests/](tests/ExtensionLibsTests/) — tests for `Plotly.NET.CSharp`, `Plotly.NET.ImageExport`, etc.
- [InteractiveTests/](tests/InteractiveTests/) — tests for `Plotly.NET.Interactive`.
- [JSTests/](tests/JSTests/) — node/Mocha tests validating generated JSON with plotly.js.
- [VersoTests/](tests/VersoTests/) — tests for `Plotly.NET.Verso`.
- [ConsoleApps/](tests/ConsoleApps/) — manual-test playgrounds, not unit tests.

### Running tests

```shell
./build.cmd runTestsAll              # everything
./build.cmd runTestsCore             # CoreTests
./build.cmd runTestsExtensionLibs    # extension lib tests
./build.cmd runTestsJSTests          # JS / Mocha tests
```

### Writing tests
- Use `FSharpTestBase` helpers: `substringIsInChart`, `chartGeneratedContains`, `getFullPlotlyJS`, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plotly/Plotly.NET](https://github.com/plotly/Plotly.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
