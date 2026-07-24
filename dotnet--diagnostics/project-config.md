---
trigger: always_on
description: This document provides standard guidance for AI agents and automated tools working on the .NET Diagnostics repository. Following these guidelines will help ensure consistency and quality across contributions.
---

# Agent Guidelines for .NET Diagnostics Repository

This document provides standard guidance for AI agents and automated tools working on the .NET Diagnostics repository. Following these guidelines will help ensure consistency and quality across contributions.

## Repository Overview

The .NET Diagnostics repository contains diagnostic tools and libraries for .NET Core, including:

- **SOS**: The Son of Strike debugger extension
- **dotnet-dump**: Dump collection and analysis utility
- **dotnet-gcdump**: Heap analysis tool
- **dotnet-trace**: Event collection tool
- **dotnet-counters**: Performance counter monitoring tool
- **Diagnostic libraries**: Client libraries and services for diagnostics

## Build System

### Building the Repository

The repository uses a cross-platform build system:

**Linux/macOS:**
```bash
./build.sh
```

**Windows:**
```cmd
Build.cmd
```

### Build Scripts Location
- Main build scripts: `build.sh` / `Build.cmd` at repository root
- Actual build logic: `eng/build.sh` / `eng/Build.cmd`
- Build configuration: `build.proj`, `Directory.Build.props`, `Directory.Build.targets`

### Common Build Options
- `-configuration <Debug|Release>`: Build configuration (default: Debug)
- `-architecture <x64|x86|arm|arm64>`: Target architecture
- `-restore`: Restore dependencies before building
- `-build`: Build the repository (incremental, only changed projects)
- `-rebuild`: Clean and rebuild (required after changing .props/.targets files)
- `-bl`: Requests a binlog.
- `-test`: Run tests after building

### Build Output Locations

Understanding where build outputs are placed is essential for verification and debugging:

- **Managed Build outputs**: `artifacts/bin/<ProjectName>/<Configuration>/<TargetFramework>/`
- **SOS Build outputs**: `artifacts/bin/<OS>.<Architecture>.<Configuration>`
- **Test results when using global test script**: `artifacts/TestResults/`
- **Build logs**: `artifacts/log/` (including `Build.binlog` for detailed analysis)
- **NuGet packages**: `artifacts/packages/<Configuration>/`
- **Temporary files**: `artifacts/tmp/`
- **Intermediate files**: `artifacts/obj/` (such as obj files, generated files, etc.)

### Quick Build Commands

After a full build of the repo has been done, some commands can be used to iterate faster on changes:

### For changes under src/Tools:

```bash
# Build the relevant tool
dotnet build src/Tools/dotnet-dump/dotnet-dump.csproj

# Build without restoring (faster if dependencies haven't changed)
dotnet build --no-restore
```

### For changes under to native files:

```bash
# Build the native components to verify compilation works
./build.sh -skipmanaged

# Do a full test run:
./build -test
```

## Testing

### Running All Tests

**Linux/macOS:**
```bash
./test.sh
```

**Windows:**
```cmd
Test.cmd
```

The test script runs all tests in the repository. **Important**: `test.sh` calls `eng/build.sh -test -skipmanaged -skipnative`, which means it only runs tests without rebuilding. Always build first if you've made code changes.

### Test Organization

Test projects are usually located in `src/tests/` with the following structure:

- **Tool and libraries tests**: `*.UnitTests.csproj` or `*.Tests.csproj` under the appropriate tool's folder in `src/tests`.
- Changes with native dependencies (SOS, DBGShim, dotnet-sos, dotnet-dump) are better tested with the global test script.

### Running Specific Tests

```bash
# Run tests for a specific project
dotnet test src/tests/Microsoft.Diagnostics.DebugServices.UnitTests/

# Run tests with detailed output
dotnet test --logger "console;verbosity=detailed"

# Run a specific test by name
dotnet test --filter "FullyQualifiedName~TestMethodName"
```

## Project Structure

```
/src
├── Microsoft.Diagnostics.DebugServices        # Debug service interfaces
├── Microsoft.Diagnostics.DebugServices.Implementation  # Debug service implementations
├── Microsoft.Diagnostics.ExtensionCommands    # SOS extension commands
├── Microsoft.Diagnostics.Monitoring           # Monitoring libraries
├── Microsoft.Diagnostics.NETCore.Client       # Diagnostic client library
├── Microsoft.Diagnostics.Repl                 # REPL infrastructure
├── Microsoft.FileFormats                      # File format parsers
├── Microsoft.SymbolStore                      # Symbol store implementation
├── SOS                                        # SOS debugger extension
├── Tools                                      # Command-line tools (dump, trace, counters, gcdump)
├── tests                                      # Test projects
└── shared                                     # Shared native code

/documentation                                  # Documentation files
/eng                                           # Engineering/build infrastructure
```

## Coding Standards

### C# Code Style

The repository follows standard .NET coding conventions defined in the `.editorconfig` file at the root. This is a **must** for C# code. Ensure your changes conform to these settings:

1. **Indentation**: 4 spaces (no tabs)
2. **Line endings**: LF on Linux/macOS, CRLF on Windows (EditorConfig enforces this)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/diagnostics](https://github.com/dotnet/diagnostics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
