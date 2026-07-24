---
trigger: always_on
description: FsToolkit.ErrorHandling is an F# utility library to work with the Result type, enabling clear, simple and powerful error handling. The library provides computation expressions, utility functions, and supports multiple target frameworks including .NET Standard 2.0/2.1, .NET 9.0, and compiles to JavaScript and Python via Fable.
---

# FsToolkit.ErrorHandling

FsToolkit.ErrorHandling is an F# utility library to work with the Result type, enabling clear, simple and powerful error handling. The library provides computation expressions, utility functions, and supports multiple target frameworks including .NET Standard 2.0/2.1, .NET 9.0, and compiles to JavaScript and Python via Fable.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Build the Repository
- **Install .NET SDK**: Download and install .NET 9.0.100 SDK (required for project compilation)
  - Also install .NET 8.0.x runtime (required for build tools)
  - Use `dotnet --version` to verify installation
  - **CRITICAL**: Ensure both SDKs are in PATH before running any build commands
- **Install Node.js**: Install Node.js v18.0.0 or higher (required for Fable JavaScript builds)
  - Current environment has v20.19.5 which works perfectly
  - Use `node --version` to verify installation
- **Install Python**: Install Python 3.10.0 or higher (required for Fable Python builds)
  - Current environment has Python 3.12.3 which works perfectly
  - Use `python3 --version` to verify installation

### Build Commands and Timing
**NEVER CANCEL ANY BUILD COMMAND** - Use appropriate timeouts and wait for completion.

- `./build.sh DotnetRestore` -- restores .NET dependencies. Takes ~2 seconds. NEVER CANCEL.
- `./build.sh Build` -- compiles all target frameworks. Takes ~90 seconds. NEVER CANCEL. Set timeout to 180+ seconds.
- `./build.sh DotnetTest` -- runs F# unit tests. Takes ~12 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
- `./build.sh NpmRestore` -- installs JavaScript dependencies. Takes ~11 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
- `./build.sh NpmTest` -- compiles F# to JavaScript via Fable and runs tests. Takes ~50 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
- `./build.sh PythonTest` -- compiles F# to Python via Fable and runs tests. Takes ~2 minutes. NEVER CANCEL. Set timeout to 300+ seconds.
- `./build.sh RunTests` -- runs ALL tests (F#, JavaScript, Python). Takes ~4.5 minutes. NEVER CANCEL. Set timeout to 600+ seconds.
- `./build.sh FormatCode` -- formats F# code using Fantomas. Takes ~5 seconds. NEVER CANCEL.
- `./build.sh CheckFormatCode` -- validates code formatting. Takes ~5 seconds. NEVER CANCEL.
- `./build.sh Clean` -- cleans bin/obj folders. Takes <1 second.

### Default Build Target
- `./build.sh` (no arguments) -- runs `DotnetPack` target which builds and packages all projects. Takes ~5-7 minutes. NEVER CANCEL. Set timeout to 600+ seconds.

## Validation Requirements

### Pre-commit Validation
Always run these commands before committing changes:
- `./build.sh CheckFormatCode` -- ensures code follows formatting standards
- `./build.sh RunTests` -- runs complete test suite across all platforms (F#, JavaScript, Python)

### Complete Development Workflow Validation
Test the complete workflow with: `./build.sh CheckFormatCode && ./build.sh RunTests`
- Total time: ~4.5-5 minutes
- Must pass all 3422+ tests across F#, JavaScript, and Python platforms
- **NEVER CANCEL** - Set timeout to 600+ seconds

### Manual Testing Scenarios
After making changes, validate functionality by:
1. **Build Validation**: Run `./build.sh Build` to ensure compilation succeeds across all target frameworks
2. **Test Validation**: Run `./build.sh RunTests` to execute the complete test suite (835 JS tests + 2 Python tests + 3406+ F# tests)
3. **Format Validation**: Run `./build.sh CheckFormatCode` to ensure code formatting compliance

### CI Build Requirements
The GitHub Actions CI will fail if:
- Code formatting is incorrect (run `./build.sh FormatCode` to fix)
- Any tests fail in F#, JavaScript, or Python targets
- Build fails on any supported platform (Ubuntu, Windows, macOS)
- Code analysis issues are detected

## Project Structure

### Key Directories
```
src/                           # Source code for all packages
├── FsToolkit.ErrorHandling/          # Core library (Result, AsyncResult, etc.)
├── FsToolkit.ErrorHandling.AsyncSeq/ # AsyncSeq extensions  
├── FsToolkit.ErrorHandling.JobResult/ # JobResult extensions
└── FsToolkit.ErrorHandling.IcedTasks/ # IcedTasks extensions

tests/                         # Test projects
├── FsToolkit.ErrorHandling.Tests/          # Core tests (majority of test coverage)
├── FsToolkit.ErrorHandling.AsyncSeq.Tests/ # AsyncSeq tests (limited Fable support)
├── FsToolkit.ErrorHandling.JobResult.Tests/ # JobResult tests
└── FsToolkit.ErrorHandling.IcedTasks.Tests/ # IcedTasks tests

build/                         # Build scripts and configuration
gitbook/                       # Documentation source
.github/workflows/build.yml    # CI/CD pipeline
```

### Target Frameworks
- **.NET Standard 2.0/2.1**: For broad compatibility
- **.NET 9.0**: Latest .NET support
- **Fable JavaScript**: Compiles to JavaScript for web usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [demystifyfp/FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
