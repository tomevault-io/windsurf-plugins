---
trigger: always_on
description: Mockolate is a modern, strongly-typed mocking library for .NET, powered by source generators. It enables fast, compile-time validated mocks for interfaces and classes with full IntelliSense support.
---

# Copilot Instructions for Mockolate

## Project Overview

Mockolate is a modern, strongly-typed mocking library for .NET, powered by source generators. It enables fast, compile-time validated mocks for interfaces and classes with full IntelliSense support.

**Key characteristics:**
- Source generator-based (no runtime proxy generation)
- Strongly-typed with compile-time safety
- AOT compatible (NativeAOT and trimming)
- Supports .NET Standard 2.0, .NET 8, .NET 10, and .NET Framework 4.8

## Architecture

The repository is organized into several key components:

### Source Projects
- **Mockolate**: Main library providing the mocking API
- **Mockolate.SourceGenerators**: Source generators that create mock implementations at compile time
- **Mockolate.Analyzers**: Roslyn analyzers for compile-time validation
- **Mockolate.Analyzers.CodeFixers**: Code fix providers for analyzer diagnostics

### Test Projects
- **Mockolate.Tests**: Core unit tests for the library
- **Mockolate.SourceGenerators.Tests**: Tests for source generators
- **Mockolate.Analyzers.Tests**: Tests for analyzers
- **Mockolate.Api.Tests**: API compatibility tests
- **Mockolate.ExampleTests**: Example usage tests
- **Aot/Mockolate.AotCompatibility.TestApp**: AOT compatibility validation

## Build System

The project uses **Nuke Build** for automation. Build scripts are available for all platforms:
- `build.sh` (Linux/macOS)
- `build.ps1` (Windows PowerShell)
- `build.cmd` (Windows Command Prompt)

### Common Build Targets
```bash
./build.sh                    # Default build
./build.sh CodeCoverage       # Build and run tests with coverage
./build.sh ApiChecks          # Run API compatibility tests
./build.sh CodeAnalysis       # Run SonarCloud analysis
./build.sh MutationTests      # Run Stryker mutation tests
./build.sh Benchmarks         # Run performance benchmarks
./build.sh Pack               # Create NuGet packages
```

### Building and Testing
```bash
# Quick build
dotnet build Mockolate.sln

# Run all tests
dotnet test Mockolate.sln

# Run specific test project
dotnet test Tests/Mockolate.Tests/Mockolate.Tests.csproj
```

## Coding Standards

### General Conventions
- Use **tabs** for indentation (tab width: 4)
- Follow `.editorconfig` settings strictly
- All code must be covered by unit tests
- Maintain > 90% code coverage
- Follow conventional commits for commit messages

### C# Conventions
- **DO NOT** use `var` - always use explicit types
- Use expression-bodied members for accessors, properties, methods, and lambdas
- Prefer pattern matching over `is` with cast or `as` with null check
- Use language keywords (`int`, `string`) over BCL types (`Int32`, `String`)
- Sort usings alphabetically with `System` namespaces first
- Do not use `this.` qualification

### Naming Conventions
- Use PascalCase for public members
- Use camelCase with leading underscore prefix for private fields (e.g., `_fieldName`)
- Use meaningful, descriptive names

### Testing Standards
- Use **xUnit** for all tests
- Test class names should end with `Tests` (e.g., `MockabilityAnalyzerTests`)
- Test method names should be descriptive using `When_Should` pattern or similar
- Use `[Fact]` for simple tests, `[Theory]` with `[InlineData]` for parameterized tests
- Arrange-Act-Assert pattern is preferred
- Each test should verify a single behavior

## Source Generator Development

When working on source generators:
- Source generators target .NET Standard 2.0
- Use incremental generators (`IIncrementalGenerator`) for performance
- Test source generators using the `Microsoft.CodeAnalysis.Testing` framework
- Ensure generated code compiles and passes all tests
- Generated code should be readable and follow the same coding standards

## Analyzer Development

When working on analyzers:
- Analyzers target .NET Standard 2.0
- Create corresponding code fixers when appropriate
- Write comprehensive tests for analyzers using `CSharpAnalyzerVerifier`
- Follow Roslyn analyzer guidelines
- Provide clear, actionable diagnostic messages

## API Surface

The library provides several key APIs:

### Mock Creation
- `Mock.Create<T>()` - Create mocks for interfaces or classes
- `Mock.Factory` - Create factory for shared behavior
- `BaseClass.WithConstructorParameters()` - For classes without default constructors

### Setup
- `mock.SetupMock.Method.*` - Configure method behavior
- `mock.SetupMock.Property.*` - Configure property behavior
- `mock.SetupMock.Indexer()` - Configure indexer behavior
- `mock.SetupMock.Delegate(...)` - Configure delegate behavior (for Action, Func, and custom delegates)

### Verification
- `mock.VerifyMock.Invoked.*` - Verify method calls
- `mock.VerifyMock.Invoked(...)` - Verify delegate invocations (for Action, Func, and custom delegates)
- `mock.VerifyMock.Got.*` - Verify property reads
- `mock.VerifyMock.Set.*` - Verify property writes
- `mock.VerifyMock.GotIndexer()` - Verify indexer reads
- `mock.VerifyMock.SetIndexer()` - Verify indexer writes

### Events
- `mock.RaiseOnMock.*` - Raise events on mocks
- `mock.VerifyMock.SubscribedTo.*` - Verify event subscriptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Testably/Mockolate](https://github.com/Testably/Mockolate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
