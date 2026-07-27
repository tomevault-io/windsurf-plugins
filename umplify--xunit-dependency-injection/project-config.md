---
trigger: always_on
description: Xunit Dependency Injection is a .NET library that brings Microsoft's dependency injection container to Xunit by leveraging Xunit's fixture pattern. This library enables dependency injection in xUnit tests using familiar Microsoft.Extensions.DependencyInjection patterns.
---

# Xunit Dependency Injection Library for .NET 9.0

Xunit Dependency Injection is a .NET library that brings Microsoft's dependency injection container to Xunit by leveraging Xunit's fixture pattern. This library enables dependency injection in xUnit tests using familiar Microsoft.Extensions.DependencyInjection patterns.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Installation
- **CRITICAL**: This project requires .NET 9.0 SDK (version 9.0.304 or later) and .NET 9.0 runtime (version 9.0.8 or later).
- Install .NET 9.0 SDK:
  ```bash
  # Download the install script
  curl -sSL -o dotnet-install.sh https://dot.net/v1/dotnet-install.sh
  # Download the official SHA-256 checksum (replace URL with actual checksum file if available)
  curl -sSL -o dotnet-install.sh.sha256 https://dot.net/v1/dotnet-install.sh.sha256
  # Verify checksum
  sha256sum -c dotnet-install.sh.sha256
  # If verification passes, install .NET SDK
  bash dotnet-install.sh --version 9.0.304 --install-dir /tmp/dotnet
  export PATH="/tmp/dotnet:$PATH"
  export DOTNET_ROOT="/tmp/dotnet"
  ```
- Verify installation: `dotnet --version` should return `9.0.304` or later
- Verify runtime: `dotnet --list-runtimes` should show `Microsoft.NETCore.App 9.0.8`

### Build and Test Commands
- **Navigate to source directory**: `cd /path/to/xunit-dependency-injection/src`
- **Restore packages**: `dotnet restore` -- takes ~8 seconds. NEVER CANCEL. Set timeout to 30+ seconds.
- **Build library**: `dotnet build --configuration Release` -- takes ~5.5 seconds. NEVER CANCEL. Set timeout to 30+ seconds.
- **Build examples**: The build command automatically builds both the main library and example tests.
- **Run example tests**: 
  ```bash
  cd ../examples/Xunit.Microsoft.DependencyInjection.ExampleTests
  dotnet test --configuration Release
  ```
  -- takes ~10.8 seconds with 43 tests passing. NEVER CANCEL. Set timeout to 60+ seconds.
- **Package library**: `dotnet pack --configuration Release` -- takes ~1.9 seconds. NEVER CANCEL. Set timeout to 30+ seconds.

### Code Quality and Formatting
- **Format code**: `dotnet format Xunit.Microsoft.DependencyInjection.sln` -- fixes whitespace and code style issues per .editorconfig
- **Verify formatting**: `dotnet format Xunit.Microsoft.DependencyInjection.sln --verify-no-changes --verbosity diagnostic`
- **ALWAYS** run `dotnet format` before committing changes to maintain code style consistency
- The project uses .editorconfig with specific C# coding standards including tabs for indentation and CRLF line endings

### Development Workflow
- Always set `PATH="/tmp/dotnet:$PATH"` and `DOTNET_ROOT="/tmp/dotnet"` in your session when working with .NET 9.0
- Test your changes by running the example tests which demonstrate real usage scenarios
- The library targets `net9.0` framework exclusively
- Use Visual Studio Code tasks defined in `.vscode/tasks.json` for build, publish, and watch operations

## Understanding Test Patterns

The library supports multiple dependency injection approaches:

### 1. Traditional Fixture-Based (Fully Supported)
```csharp
public class MyTests : TestBed<TestProjectFixture>
{
    [Fact]
    public async Task TestCalculation()
    {
        var calculator = _fixture.GetService<ICalculator>(_testOutputHelper);
        var result = await calculator.AddAsync(1, 2);
        Assert.Equal(3, result);
    }
}
```

### 2. Property Injection (Recommended - New in 9.2.0+)
```csharp
public class MyTests : TestBedWithDI<TestProjectFixture>
{
    [Inject] private ICalculator Calculator { get; set; } = null!;
    [Inject("Porsche")] private ICarMaker PorscheMaker { get; set; } = null!;
    
    [Fact]
    public async Task TestCalculation()
    {
        var result = await Calculator.AddAsync(1, 2);
        Assert.Equal(3, result);
    }
}
```

### 3. Factory Pattern (Experimental)
```csharp
public class MyTests : TestBed<FactoryTestProjectFixture>
{
    [Fact] 
    public async Task TestConstructorInjection()
    {
        var service = _fixture.CreateTestInstance<SimpleService>(_testOutputHelper);
        var result = await service.CalculateAsync(10, 5);
        Assert.True(result > 0);
    }
}
```

## Validation Scenarios

After making any changes to the library code:
1. **Build validation**: Run `dotnet build --configuration Release` and ensure it completes successfully
2. **Test validation**: Run example tests with `dotnet test --configuration Release` and verify all 43 tests pass
3. **Format validation**: Run `dotnet format` to ensure code follows project standards
4. **Package validation**: Run `dotnet pack --configuration Release` to ensure the library can be packaged

### Manual Testing Scenarios
The example tests demonstrate complete usage patterns:
- **Dependency injection setup**: Tests show how to configure services in `TestProjectFixture`
- **Service resolution**: Tests verify both scoped and singleton service resolution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Umplify/xunit-dependency-injection](https://github.com/Umplify/xunit-dependency-injection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
