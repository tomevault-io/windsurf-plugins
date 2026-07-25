---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

This project uses **Microsoft Testing Platform (MTP)** with the **TUnit** testing framework. Test commands differ significantly from traditional VSTest.

See: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test?tabs=dotnet-test-with-mtp

### Prerequisites

```powershell
# Check .NET installation
dotnet --info

# Restore NuGet packages
dotnet restore Splat.slnx
```

**Note**: This repository uses **SLNX** (XML-based solution format) instead of the legacy SLN format.

### Build Commands

**CRITICAL:** The working folder must be `./src` folder. These commands won't function properly without the correct working folder.

```powershell
# Build the solution
dotnet build Splat.slnx -c Release

# Build with warnings as errors (includes StyleCop violations)
dotnet build Splat.slnx -c Release -warnaserror

# Clean the solution
dotnet clean Splat.slnx
```

### Test Commands (Microsoft Testing Platform)

**CRITICAL:** This repository uses MTP configured in `global.json`. All TUnit-specific arguments must be passed after `--`:

The working folder must be `./src` folder. These commands won't function properly without the correct working folder.

**IMPORTANT:**
- Do NOT use `--no-build` flag when running tests. Always build before testing to ensure all code changes (including test changes) are compiled. Using `--no-build` can cause tests to run against stale binaries and produce misleading results.
- Use `--output Detailed` to see Console.WriteLine output from tests. This must be placed BEFORE any `--` separator:
  ```powershell
  dotnet test --output Detailed -- --treenode-filter "..."
  ```

```powershell
# Run all tests in the solution
dotnet test --solution Splat.slnx -c Release

# Run all tests in a specific project
dotnet test --project tests/Splat.Tests/Splat.Tests.csproj -c Release

# Run a single test method using treenode-filter
# Syntax: /{AssemblyName}/{Namespace}/{ClassName}/{TestMethodName}
dotnet test --project tests/Splat.Tests/Splat.Tests.csproj -- --treenode-filter "/*/*/*/MyTestMethod"

# Run all tests in a specific class
dotnet test --project tests/Splat.Tests/Splat.Tests.csproj -- --treenode-filter "/*/*/MyClassName/*"

# Run tests in a specific namespace
dotnet test --project tests/Splat.Tests/Splat.Tests.csproj -- --treenode-filter "/*/MyNamespace/*/*"

# Filter by test property (e.g., Category)
dotnet test --solution Splat.slnx -- --treenode-filter "/*/*/*/*[Category=Integration]"

# Run tests with code coverage (Microsoft Code Coverage)
dotnet test --solution Splat.slnx -- --coverage --coverage-output-format cobertura

# Run tests with detailed output
dotnet test --solution Splat.slnx -- --output Detailed

# List all available tests without running them
dotnet test --project tests/Splat.Tests/Splat.Tests.csproj -- --list-tests

# Fail fast (stop on first failure)
dotnet test --solution Splat.slnx -- --fail-fast

# Control parallel test execution
dotnet test --solution Splat.slnx -- --maximum-parallel-tests 4

# Generate TRX report
dotnet test --solution Splat.slnx -- --report-trx

# Disable logo for cleaner output
dotnet test --project tests/Splat.Tests/Splat.Tests.csproj -- --disable-logo

# Combine options: coverage + TRX report + detailed output
dotnet test --solution Splat.slnx -- --coverage --coverage-output-format cobertura --report-trx --output Detailed
```

**Alternative: Using `dotnet run` for single project**
```powershell
# Run tests using dotnet run (easier for passing flags)
dotnet run --project tests/Splat.Tests/Splat.Tests.csproj -c Release -- --treenode-filter "/*/*/*/MyTest"

# Disable logo for cleaner output
dotnet run --project tests/Splat.Tests/Splat.Tests.csproj -- --disable-logo --treenode-filter "/*/*/*/Test1"
```

### TUnit Treenode-Filter Syntax

The `--treenode-filter` follows the pattern: `/{AssemblyName}/{Namespace}/{ClassName}/{TestMethodName}`

**Examples:**
- Single test: `--treenode-filter "/*/*/*/MyTestMethod"`
- All tests in class: `--treenode-filter "/*/*/MyClassName/*"`
- All tests in namespace: `--treenode-filter "/*/MyNamespace/*/*"`
- Filter by property: `--treenode-filter "/*/*/*/*[Category=Integration]"`
- Multiple wildcards: `--treenode-filter "/*/*/MyTests*/*"`

**Note:** Use single asterisks (`*`) to match segments. Double asterisks (`/**`) are not supported in treenode-filter.

### Key TUnit Command-Line Flags

- `--treenode-filter` - Filter tests by path pattern or properties (syntax: `/{Assembly}/{Namespace}/{Class}/{Method}`)
- `--list-tests` - Display available tests without running
- `--fail-fast` - Stop after first failure
- `--maximum-parallel-tests` - Limit concurrent execution (default: processor count)
- `--coverage` - Enable Microsoft Code Coverage
- `--coverage-output-format` - Set coverage format (cobertura, xml, coverage)
- `--report-trx` - Generate TRX format reports
- `--output` - Control verbosity (Normal or Detailed)
- `--no-progress` - Suppress progress reporting
- `--disable-logo` - Remove TUnit logo display
- `--diagnostic` - Enable diagnostic logging (Trace level)
- `--timeout` - Set global test timeout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactiveui/splat](https://github.com/reactiveui/splat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
