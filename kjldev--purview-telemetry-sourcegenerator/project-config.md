---
trigger: always_on
description: Purview Telemetry Source Generator is a .NET incremental source generator that generates [`ActivitySource`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.activitysource), [`ILogger`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger), and [`Metrics`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.metrics) based telemetry from methods you define on an interface.
---

# Purview Telemetry Source Generator

Purview Telemetry Source Generator is a .NET incremental source generator that generates [`ActivitySource`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.activitysource), [`ILogger`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger), and [`Metrics`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.metrics) based telemetry from methods you define on an interface.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites

Install these exact dependencies in order:

- Install .NET 10.0.200 SDK: `curl -sSL https://dot.net/v1/dotnet-install.sh | bash -s -- --version 10.0.200`
- Install .NET 10.0 runtime: `curl -sSL https://dot.net/v1/dotnet-install.sh | bash -s -- --runtime dotnet --version 10.0.0`
- Install Bun: `curl -fsSL https://bun.sh/install | bash`
- Set environment variables: `export PATH=$HOME/.bun/bin:$HOME/.dotnet:$PATH && export DOTNET_ROOT=$HOME/.dotnet`

### Bootstrap, Build, and Test

Bootstrap and build the repository:

- `just build` -- builds the main source generator and integration tests. Takes 26 seconds. NEVER CANCEL. Set timeout to 60+ minutes.
- `just test` -- runs 282 integration tests. Takes 42 seconds. NEVER CANCEL. Set timeout to 60+ minutes.
- `just format` -- formats code according to .editorconfig rules. Takes 21 seconds. NEVER CANCEL. Set timeout to 30+ minutes.

Alternative direct commands (use environment variables above):

- `dotnet build ./src/Purview.Telemetry.SourceGenerator.slnx --configuration Release`
- `dotnet test ./src/Purview.Telemetry.SourceGenerator.slnx --configuration Release`
- `dotnet format ./src/`

### Build and Test Sample Application

The sample application demonstrates the source generator in action:

- `cd samples/SampleApp && dotnet build --configuration Release` -- takes 19 seconds. NEVER CANCEL. Set timeout to 30+ minutes.
- `cd samples/SampleApp && dotnet test --configuration Release` -- runs 8 tests, takes 3 seconds.

### Package Creation

- `just pack` -- creates NuGet package with current version from package.json
- `just version` -- displays current version (currently 4.0.0)

## Validation

### Manual Validation Requirements

Always manually validate changes to the source generator:

- ALWAYS run `just build && just test` after making any changes to the source generator code
- ALWAYS build and test the sample application: `cd samples/SampleApp && dotnet build --configuration Release && dotnet test --configuration Release`
- ALWAYS run `just format` before committing to ensure code formatting compliance
- Test actual source generator functionality by examining generated files in the sample project (EmitCompilerGeneratedFiles is enabled)

### Functional Testing Scenarios

Test these scenarios when modifying the source generator:

- **Interface to Implementation Generation**: Modify an interface in `samples/SampleApp/SampleApp.Host/APIs/` and verify generated telemetry code appears
- **Activity Generation**: Test ActivitySource generation by adding methods with activity attributes
- **Logging Generation**: Test ILogger generation by adding methods with logging attributes
- **Metrics Generation**: Test metrics generation by adding methods with metrics attributes
- **Integration Test Coverage**: Verify new functionality is covered by tests in `src/Purview.Telemetry.SourceGenerator.IntegrationTests/`

### CI Validation

Always run these validation steps before committing:

- `just format` (takes 21 seconds)
- `just build` (takes 26 seconds)
- `just test` (takes 42 seconds)
- Sample app build and test (takes 22 seconds total)

The CI pipeline (`./.github/workflows/ci.yml`) runs the same dotnet restore → build → test workflow.

## Common Tasks

### Project Structure

```
src/
├── Purview.Telemetry.SourceGenerator/          # Main source generator library
├── Purview.Telemetry.SourceGenerator.IntegrationTests/  # 282 integration tests
├── Purview.Telemetry.SourceGenerator.slnx      # Main solution
└── global.json                                 # Pins to .NET 10.0.200

samples/
└── SampleApp/                                  # .NET Aspire demo application
    ├── SampleApp.AppHost/                      # Aspire AppHost
    ├── SampleApp.Host/                         # Main web API
    ├── SampleApp.ServiceDefaults/              # Shared service config
    ├── SampleApp.UnitTests/                    # Sample app tests
    └── SampleApp.slnx                          # Sample solution

benchmarks/
└── Purview.Telemetry.Benchmarks/               # BenchmarkDotNet benchmark project

.build/
└── update-version.ts                           # Version management script


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kjldev/purview-telemetry-sourcegenerator](https://github.com/kjldev/purview-telemetry-sourcegenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
