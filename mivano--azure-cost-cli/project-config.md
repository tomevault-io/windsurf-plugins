---
trigger: always_on
description: Azure Cost CLI is a .NET command-line tool that retrieves Azure cost information via the Azure Cost Management API. It is distributed as a .NET global tool (`azure-cost-cli`) and supports multiple output formats.
---

# AGENTS.md

## Project Overview

Azure Cost CLI is a .NET command-line tool that retrieves Azure cost information via the Azure Cost Management API. It is distributed as a .NET global tool (`azure-cost-cli`) and supports multiple output formats.

## Architecture

```
src/
├── Program.cs                  # Entry point, DI setup, Spectre.Console.Cli command registration
├── Commands/                   # CLI commands (one subfolder per command)
│   ├── CommandHelpers.cs       # Shared validation: subscription resolution, timeframe, version
│   ├── AccumulatedCost/        # Default command: accumulated cost overview
│   ├── DailyCost/              # Daily costs grouped by dimension
│   ├── CostByResource/         # Cost breakdown by resource
│   ├── CostByTag/              # Cost breakdown by tag key(s)
│   ├── Budgets/                # Azure budget information
│   ├── DetectAnomaly/          # Anomaly detection on cost data
│   ├── Diff/                   # Compare costs between two timeframes
│   ├── Regions/                # List Azure regions
│   ├── WhatIf/                 # What-if cost scenarios (region/dev-test)
│   ├── Threshold/              # CI/CD cost threshold checks (daily-change, forecast-deviation, service-spike, weekly-average)
│   └── Prices/                 # Price catalog lookup
├── CostApi/                    # Azure API integration layer
│   ├── AzureCostApiRetriever.cs    # Calls Azure Cost Management REST API
│   ├── AzurePriceRetriever.cs      # Calls Azure Retail Prices API
│   ├── AzureRegionsRetriever.cs    # Calls Azure Regions API
│   └── PollyPolicyExtensions.cs    # Retry/resilience policies (Polly)
├── OutputFormatters/           # Output rendering (Console, JSON, Text, Markdown, CSV)
│   └── OutputFormatterFactory.cs   # Factory for formatter dictionary (used by all commands)
└── Infrastructure/             # DI registrar/resolver, extensions, type converters
```

Each command inherits from `AsyncCommand<TSettings>` (Spectre.Console.Cli) and follows a consistent pattern:
- A `*Settings.cs` class defines CLI options/arguments
- A `*Command.cs` class implements the command logic
- `Validate()` uses `CommandHelpers` for subscription resolution and timeframe validation
- Formatters are initialized via `OutputFormatterFactory.Create()`
- Commands call `ICostRetriever` / `IPriceRetriever` / `IRegionsRetriever` (injected via DI)
- Results are rendered through the appropriate `OutputFormatter`

## Prerequisites

- .NET 8.0 SDK or .NET 10.0 SDK
- Azure CLI authenticated (`az login`) for cost data access
- An Azure subscription with Cost Management API access

## Build

```bash
# Build the solution (from repo root)
dotnet build src/azure-cost-cli.sln

# Build a specific target framework
dotnet build src/azure-cost-cli.csproj --framework net10.0
```

The project targets both `net8.0` and `net10.0`. Warnings are treated as errors for NuGet dependency resolution (`NU1605`).

## Test

```bash
# Run all tests (both frameworks)
dotnet test src/azure-cost-cli.sln

# Run tests for a specific framework
dotnet test tests/AzureCostCli.Tests/AzureCostCli.Tests.csproj --framework net10.0
```

Tests use **xUnit**, **Moq** for mocking, and **Shouldly** for assertions. Tests cover:
- Command validation logic (date ranges, settings, subscription resolution)
- Shared infrastructure (CommandHelpers, OutputFormatterFactory)
- Cost data model parsing (API response deserialization)
- Output formatters (JSON, Text, Markdown, CSV rendering)
- Polly resilience policy behavior

## Run

```bash
# Run from source (specify framework since the project multi-targets)
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- [command] [options]

# Show all available commands and options
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- --help

# Examples
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- accumulatedCost -o json
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- accumulatedCost -o text
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- dailyCosts --dimension MeterCategory
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- costByResource -o csv
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- budgets -o json
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- detectAnomalies -o text
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- threshold daily-change --percentage 20 --fail-on-threshold
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- threshold service-spike --fixed-amount 200 -o json
dotnet run --project src/azure-cost-cli.csproj --framework net10.0 -- threshold weekly-average --fixed-amount 50 --fail-on-threshold
```

### Install as global tool (from source)

```bash
dotnet pack src/azure-cost-cli.csproj -c Release
dotnet tool install --global --add-source src/nupkg azure-cost-cli

# Then run with:
azure-cost accumulatedCost -o json
```

## Available Commands

| Command | Description |
|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mivano/azure-cost-cli](https://github.com/mivano/azure-cost-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
