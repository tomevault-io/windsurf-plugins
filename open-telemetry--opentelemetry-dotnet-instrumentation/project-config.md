---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## What this repository is

OpenTelemetry .NET Automatic Instrumentation: adds OpenTelemetry to .NET and
.NET Framework applications without source code changes, via the .NET host
startup hook and/or a native CLR Profiler that rewrites IL at runtime.

## Build and test

Builds are driven by [Nuke](https://github.com/nuke-build/nuke). `./build.cmd`
(Windows) / `./build.sh` (Linux, macOS) bootstrap `dotnet nuke`; run
`dotnet tool restore` first if using `dotnet nuke` directly. The default target
is `BuildTracer`; main artifact is `bin/tracer-home`.

```cmd
dotnet nuke --help                 :: list all targets and parameters
./build.cmd                        :: BuildTracer - native + managed src -> bin/tracer-home
./build.cmd BuildNativeWorkflow    :: native only
./build.cmd Workflow               :: full build + all tests (what CI approximates)
./build.cmd NativeTests            :: build and run C++ unit tests
./build.cmd ManagedTests           :: build and run managed unit + integration tests
./build.cmd TestWorkflow           :: NativeTests + ManagedTests, assumes BuildTracer already ran
./build.cmd BuildNuGetPackages     :: needs CI artifacts, see docs/developing.md
```

Useful parameters (apply to the test targets):

- `--test-project <substring>` - select test projects by name substring.
- `--test-name <substring>` - becomes an xunit `FullyQualifiedName~` filter.
- `--test-target-framework net8.0|net9.0|net10.0|net462|net472|...` - otherwise
  every TFM the project supports is run.
- `--containers linux|windows|windows-only|none` - integration tests are
  tagged `[Trait("Containers", "Linux"|"Windows|Any")]`; use `none` to skip every
  test needing Docker.
- `--test-count N` - repeat runs, for flakiness checks.
- `--configuration Debug`, `--platform x86|x64|ARM64`, `--skip <target>`.

Running one integration test:

```cmd
./build.cmd BuildTracer
./build.cmd --target ManagedTests --test-project IntegrationTests --test-name MongoDBTests --test-target-framework net8.0
```

Integration tests can also be run straight from an IDE / `dotnet test` once
`bin/tracer-home` exists, but then only the latest library version of each
instrumented package is exercised (Nuke runs the full version matrix).

Test logs and profiler logs land in `test-artifacts/`.

### Lint and format

- `dotnet format .\OpenTelemetry.AutoInstrumentation.sln --verify-no-changes` -
  exactly what the `dotnet format` CI check runs. Warnings are errors
  (`TreatWarningsAsErrors`), analysis level is `latest-All`, StyleCop is on
  solution-wide, nullable + implicit usings are enabled.
- `./scripts/format-native.sh` (after `./scripts/download-clang-tools.sh`) for
  C++.
- `dotnet nuke InstallDocumentationTools ValidateDocumentation` for markdownlint
  & cspell; `nuke MarkdownLintFix` auto-fixes. Requires Node.js.

### Local telemetry backend

`docker compose -f dev/docker-compose.yaml up` starts a Collector + Jaeger
(<http://localhost:16686/search>, metrics on `:8889`, files in `dev/log`).
`OTEL_DOTNET_AUTO_HOME=bin/tracer-home . ./instrument.sh` exports the profiler
env vars into the current shell, or `./instrument.sh dotnet MyApp.dll` to launch
an instrumented app directly. `examples/playground` is a scratch app for this.

## Architecture

Read `docs/design.md` for the full picture. The moving parts:

- `src/OpenTelemetry.AutoInstrumentation.Native` - C++ CLR Profiler
  (`cor_profiler.cpp`, `il_rewriter*`, `method_rewriter.cpp`, plus the
  CallTarget token helpers). Receives instrumentation definitions from managed
  code, requests ReJIT for target methods, and rewrites their IL. On .NET
  Framework it also injects the Loader at startup and performs assembly
  reference redirection.
- `src/OpenTelemetry.AutoInstrumentation.StartupHook` - .NET-only entry point
  (`StartupHook.Initialize()`), loaded via `DOTNET_STARTUP_HOOKS`.
- `src/OpenTelemetry.AutoInstrumentation.Loader` - creates
  `Loader.Startup`, hooks `AssemblyResolve`/`AssemblyLoadContext` so SDK and
  instrumentation assemblies resolve, then reflectively calls
  `Instrumentation.Initialize` in the managed profiler.
- `src/OpenTelemetry.AutoInstrumentation` - the managed profiler: SDK setup
  (`Instrumentation.cs`, `Configurations/`), CallTarget infrastructure
  (`CallTarget/`), DuckTyping (`DuckTyping/`), bytecode instrumentations
  (`Instrumentations/`), plugin hooks (`Plugins/`), vendored code (`Vendors/`).
- `src/OpenTelemetry.AutoInstrumentation.Assemblies{,.NetFramework}` - pull in
  the OpenTelemetry SDK + instrumentation library packages that get copied into
  the distribution; `...AdditionalDeps` produces the additional-deps files.
- `src/SourceGenerators` - `InstrumentationDefinitionsGenerator` scans
  `[InstrumentMethod]` attributes and emits the `InstrumentationDefinitions`
  partial (`GetDefinitionsArray`) that is handed to the native side as
  `NativeCallTargetDefinition[]`. Generated output is under
  `src/OpenTelemetry.AutoInstrumentation/Generated/<tfm>/`.
- `src/OpenTelemetry.AutoInstrumentation.PluginApi`, `nuget/` - public plugin
  API and the NuGet package layouts (`OpenTelemetry.AutoInstrumentation`,
  `...Runtime.Native`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-telemetry/opentelemetry-dotnet-instrumentation](https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
