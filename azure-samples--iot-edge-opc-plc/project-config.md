---
trigger: always_on
description: OPC UA PLC server simulator for Azure IoT Edge. Generates random data, anomalies,
---

# AGENTS.md - Coding Agent Guidelines for iot-edge-opc-plc

## Project Overview

OPC UA PLC server simulator for Azure IoT Edge. Generates random data, anomalies,
alarms, and supports user-defined nodes. Built with C# 12 / .NET 10.0, using the
OPC Foundation UA SDK and ASP.NET Core for web hosting.

## Build Commands

```bash
# Build solution (Release)
dotnet build opcplc.sln -c Release

# Build solution (Debug)
dotnet build opcplc.sln -c Debug

# Run all tests
dotnet test tests/opc-plc-tests.csproj -c Release

# Run a single test by fully-qualified name
dotnet test tests/opc-plc-tests.csproj -c Release --filter "FullyQualifiedName~OpcPlc.Tests.SimulatorNodesTests.Telemetry_StepUp"

# Run a single test by name
dotnet test tests/opc-plc-tests.csproj -c Release --filter "Name=Telemetry_StepUp"

# Run all tests in a single test class
dotnet test tests/opc-plc-tests.csproj -c Release --filter "FullyQualifiedName~OpcPlc.Tests.SimulatorNodesTests"

# Run the server locally
dotnet run --project src/opc-plc.csproj -- --pn=50000 --autoaccept

# Docker build (release)
docker build -f Dockerfile.release -t iotedge/opc-plc .
```

**Important**: `TreatWarningsAsErrors` is enabled globally. All warnings must be fixed.

## Project Structure

```
src/                          # Main application
  opc-plc.csproj              # Target: net10.0, LangVersion: Preview
  Program.cs                  # Entry point
  OpcPlcServer.cs             # Server orchestration, plugin loading
  PlcServer.cs                # OPC UA StandardServer override
  PlcNodeManager.cs           # OPC UA node manager
  PlcSimulation.cs            # Simulation engine
  TimeService.cs              # Virtual time (mocked in tests)
  PluginNodes/                # Plugin node implementations (IPluginNodes)
    Models/IPluginNodes.cs    # Plugin interface
    PluginNodeBase.cs         # Base class with primary constructor
  Configuration/              # CLI options, OPC UA config
  Helpers/                    # Metrics, OTEL, CLI helpers
  DeterministicAlarms/        # Deterministic alarm system
  Boilers/                    # Boiler simulation models
tests/                        # Integration tests (NUnit)
  opc-plc-tests.csproj        # Test project
  PlcSimulatorFixture.cs      # Starts real OPC PLC server with mocked time
  SimulatorTestsBase.cs       # Base class for read/write tests
  MonitoringTestsBase.cs      # Base class for subscription/event tests
tools/scripts/                # PowerShell CI/build scripts
```

## Code Style Guidelines

### Formatting (enforced by .editorconfig)
- **Indent**: 4 spaces for C#, 2 spaces for XML/JSON/YAML
- **Braces**: Allman style (opening brace on new line for types, methods, control blocks)
- **Line length**: 120 characters max
- **Line endings**: LF (`end_of_line = lf`)
- **Final newline**: Required
- **Trim trailing whitespace**: Yes
- **Charset**: UTF-8

### Namespaces and Imports
- **File-scoped namespaces**: `namespace OpcPlc.PluginNodes;`
- **Usings go inside the namespace**, after the file-scoped namespace declaration
- **Sort alphabetically** as a single group (no `System.*` first, no group separation)

```csharp
namespace OpcPlc.PluginNodes;

using Microsoft.Extensions.Logging;
using Opc.Ua;
using OpcPlc.Helpers;
using System.Collections.Generic;
```

### Naming Conventions
| Element | Convention | Example |
|---------|-----------|---------|
| Private instance fields | `_camelCase` (underscore prefix) | `_cancellationTokenSource` |
| Private constants | PascalCase | `DefaultMinThreads` |
| Public constants | PascalCase | `PlcShutdownWaitSeconds` |
| Local variables / parameters | camelCase | `nodeCount`, `cancellationToken` |
| Methods, Properties, Classes | PascalCase | `StartAsync`, `NodeCount` |
| Async methods | Must end in `Async` | `CreateSessionAsync` |
| Interfaces | `I` prefix | `IPluginNodes`, `ITimer` |

### Type Usage
- **`var`**: Use when type is apparent from the right side; use explicit types for built-in types (`string`, `int`, `uint`)
- **Pattern matching**: Prefer `is not null` over `!= null`; use switch expressions
- **C# 12 features in use**: Primary constructors, collection expressions `[...]`, file-scoped namespaces
- **Nullable reference types**: NOT globally enabled; null checks are manual

### Async/Await
- **Always use `.ConfigureAwait(false)`** on every `await` (CA2007 is a warning)
- **Always suffix async methods with `Async`**
- **Pass `CancellationToken`** through async call chains

```csharp
await StartPlcServerAsync(cancellationToken).ConfigureAwait(false);
```

### Error Handling
- Use **exception filters** (`catch ... when`) for specific status codes:
```csharp
catch (ServiceResultException ex) when (ex.StatusCode == StatusCodes.BadServerHalted)
{
    LogCreateSessionWhileHalted();
    return new ResponseHeader { ServiceResult = StatusCodes.BadServerHalted };
}
```
- Pattern: **metrics + log + rethrow** for unknown exceptions
- Bare `catch` blocks are acceptable for non-critical paths (shutdown, IP resolution) with a comment
- Guard clauses: `_field = param ?? throw new ArgumentNullException(nameof(param));`

### Logging
- Use **`[LoggerMessage]` source-generated logging** for performance-critical paths (partial methods on partial classes):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/iot-edge-opc-plc](https://github.com/Azure-Samples/iot-edge-opc-plc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
