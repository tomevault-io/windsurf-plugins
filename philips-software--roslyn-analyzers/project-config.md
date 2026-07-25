---
trigger: always_on
description: The Philips Roslyn Analyzers repository contains customized Roslyn diagnostic analyzers for C# that provide real-time feedback to developers. This is a .NET 8.0 solution with multiple analyzer projects that compile to NuGet packages.
---

# Philips Roslyn Analyzers
The Philips Roslyn Analyzers repository contains customized Roslyn diagnostic analyzers for C# that provide real-time feedback to developers. This is a .NET 8.0 solution with multiple analyzer projects that compile to NuGet packages.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites
- .NET 8.0 SDK is required and available
- Solution targets both .NET 8.0 and .NET Standard 2.0

### Core Build and Test Commands
Always run these commands in the repository root directory:

```bash
# Clean build artifacts (quick - ~1 second)
dotnet clean

# Restore dependencies (quick - ~1 second if already restored)
dotnet restore

# Build the entire solution -- takes ~1m 21s. NEVER CANCEL. Set timeout to 3+ minutes.
dotnet build --configuration Release

# Run the full test suite -- takes ~48 seconds, runs 1903 tests. NEVER CANCEL. Set timeout to 2+ minutes.
dotnet test --configuration Release --logger "trx;LogFileName=test-results.trx"

# Validate code formatting -- takes ~23 seconds. NEVER CANCEL. Set timeout to 1+ minutes.
dotnet format style --verify-no-changes --no-restore --verbosity detailed
```

### Package Creation
The build process automatically creates NuGet packages in the `./Packages/` directory:
- `Philips.CodeAnalysis.MaintainabilityAnalyzers.*.nupkg`
- `Philips.CodeAnalysis.DuplicateCodeAnalyzer.*.nupkg`  
- `Philips.CodeAnalysis.MoqAnalyzers.*.nupkg`
- `Philips.CodeAnalysis.MsTestAnalyzers.*.nupkg`
- `Philips.CodeAnalysis.SecurityAnalyzers.*.nupkg`

Each package includes both `.nupkg` and `.snupkg` (symbol) packages.

## Validation

### Complete Validation Workflow
Before submitting any changes, ALWAYS run this complete validation sequence:

```bash
# 1. Clean and build
dotnet clean
dotnet build --configuration Release  # NEVER CANCEL: ~1m 21s

# 2. Run tests  
dotnet test --configuration Release --logger "trx;LogFileName=test-results.trx"  # NEVER CANCEL: ~48s

# 3. Check formatting
dotnet format style --verify-no-changes --no-restore --verbosity detailed  # NEVER CANCEL: ~23s

# 4. Run dogfooding process (validate analyzers work on own codebase)
# Use MCP server run_dogfood tool or manual process via CI workflow

# 5. Verify code coverage meets 80% SonarCloud requirement
# Use MCP server analyze_coverage tool to check coverage gaps
```

Total validation time: ~2m 32s + dogfood/coverage checks - NEVER CANCEL these commands.

### Dogfooding Process
This repository uses a "dogfooding" process where the analyzers analyze their own code:
- Build creates analyzer packages with `.Dogfood` suffix
- Analyzers are then applied to the codebase itself
- All analyzer violations must be fixed, not suppressed

### Code Coverage Requirements
SonarCloud enforces **80% code coverage** and will fail the build if this threshold is not met:
- Current coverage is over 90%, so maintain this high standard
- Use the **MCP server `analyze_coverage` tool** to identify coverage gaps and get actionable suggestions
- Tool provides specific test templates and prioritizes areas needing coverage
- Focus testing on error handling, edge cases, and complex logic paths

### Mandatory CI Requirements
The CI process enforces these checks that will cause build failures:
- All tests must pass (1903 tests)
- Code formatting must be perfect (no deviations from .editorconfig)
- **80% code coverage minimum** (enforced by SonarCloud)
- All analyzer warnings must be addressed (no suppressions allowed)
- Dogfood process must complete successfully

## Project Structure

### Key Projects
- **Philips.CodeAnalysis.Common** - Shared utilities and base classes
- **Philips.CodeAnalysis.MaintainabilityAnalyzers** - Code maintainability rules
- **Philips.CodeAnalysis.DuplicateCodeAnalyzer** - Duplicate code detection
- **Philips.CodeAnalysis.MoqAnalyzers** - Moq testing framework rules
- **Philips.CodeAnalysis.MsTestAnalyzers** - MSTest framework rules  
- **Philips.CodeAnalysis.SecurityAnalyzers** - Security-focused rules
- **Philips.CodeAnalysis.Test** - All unit tests (1903 tests)
- **Philips.CodeAnalysis.Benchmark** - Performance benchmarking
- **Philips.CodeAnalysis.AnalyzerPerformance** - Performance analysis tools

### Solution Structure
```
Philips.CodeAnalysis.sln - Main solution file
├── Philips.CodeAnalysis.Common/ - Shared utilities
├── Philips.CodeAnalysis.MaintainabilityAnalyzers/ - Core analyzers  
├── Philips.CodeAnalysis.DuplicateCodeAnalyzer/ - Duplicate detection
├── Philips.CodeAnalysis.MoqAnalyzers/ - Moq-specific rules
├── Philips.CodeAnalysis.MsTestAnalyzers/ - MSTest rules
├── Philips.CodeAnalysis.SecurityAnalyzers/ - Security rules
├── Philips.CodeAnalysis.Test/ - All unit tests
├── Philips.CodeAnalysis.Benchmark/ - Benchmarking
├── Philips.CodeAnalysis.AnalyzerPerformance/ - Performance tools
├── Documentation/ - Rule documentation by category
├── Packages/ - Generated NuGet packages (created during build)
└── .github/workflows/ - CI/CD pipelines
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philips-software/roslyn-analyzers](https://github.com/philips-software/roslyn-analyzers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
