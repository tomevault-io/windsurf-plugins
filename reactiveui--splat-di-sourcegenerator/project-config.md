---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Splat DI Source Generator: Dependency Injection Code Generation

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup
- **CRITICAL**: Requires .NET 9.0 SDK (not .NET 8.0). Install with:
  ```bash
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 9.0
  export PATH="$HOME/.dotnet:$PATH"
  ```
- **Platform Support**: This project **builds fully only on Windows**. Linux/macOS have partial support due to ILRepack tooling requirements.
- **Development Tools**: Visual Studio 2022 or VS Code with C# extension.

### Code Style and Analysis Enforcement
- **EditorConfig Compliance**: Repository uses comprehensive `.editorconfig` with 500+ rules for C# formatting, naming conventions, and code analysis
- **StyleCop Analyzers**: Enforces consistent C# code style with `stylecop.analyzers` (v1.2.0-beta.556)
- **Roslynator Analyzers**: Additional code quality rules with `Roslynator.Analyzers` (v4.14.0)
- **Analysis Level**: Set to `latest` with enhanced .NET analyzers enabled
- **CRITICAL**: All code must comply with **ReactiveUI contribution guidelines**: https://www.reactiveui.net/contribute/index.html

### Code Formatting (Fast - Always Run)
- **ALWAYS** run formatting before committing:
  ```bash
  cd src
  dotnet format whitespace --verify-no-changes
  dotnet format style --verify-no-changes
  ```
  Time: **2-5 seconds per command**.

## Project Overview

### What is Splat DI Source Generator?
This project is a source generator that produces Splat-based registrations for both constructor and property injection. It eliminates the need for reflection by using C# Source Generation to create dependency injection registrations at compile time.

### Key Features
- **Constructor Injection**: Automatic registration generation based on constructor parameters
- **Property Injection**: Support for `[DependencyInjectionProperty]` attribute on properties
- **Lazy Singleton Support**: `RegisterLazySingleton<TInterface, TImplementation>()` for singleton instances
- **No Reflection**: Full native speed through compile-time code generation
- **Splat Integration**: Seamless integration with the Splat service location framework

## Common Development Tasks

### Source Generator Development
1. **Follow Roslyn Source Generator best practices** - see Microsoft documentation on source generators
2. **Ensure StyleCop compliance** - all code must pass StyleCop analyzers (SA* rules)
3. **Run code analysis** - `dotnet build` must complete without analyzer warnings
4. **Add unit tests** - use Microsoft.CodeAnalysis.Testing for source generator tests
5. **Update documentation** - especially for public APIs with XML doc comments
6. **Test generated code** - verify the output compiles and behaves correctly

### Adding New Features
1. **Follow coding standards** - see ReactiveUI guidelines: https://www.reactiveui.net/contribute/index.html
2. **Ensure cross-platform compatibility** - while builds require Windows, generated code should work everywhere
3. **Add comprehensive tests** - test both the generator and the generated code
4. **Update README.md** - document new attributes or registration methods
5. **Consider performance** - source generators run during compilation

### Testing Source Generators
- Use `Microsoft.CodeAnalysis.Testing` framework for testing source generators
- Test both successful generation and error cases
- Verify generated code compiles and produces expected registrations
- Test edge cases like multiple constructors, missing dependencies, etc.

## CI/CD Integration

### GitHub Actions (Windows-based)
- Uses `reactiveui/actions-common` workflow
- Requires Windows runner for full build due to ILRepack tooling
- Installs all workloads automatically
- Runs comprehensive test suite and uploads coverage

### Local Development
- **Use** Linux/macOS for quick iteration on core source generator logic
- **Format code** before every commit
- **Test generated output** when changing generation logic
- **Full builds require Windows** due to IL merging requirements

## Troubleshooting

### Common Issues
1. **"ILRepack not found" errors**: Platform limitation - use Windows for full builds
2. **"Invalid framework identifier" errors**: Use explicit `-p:TargetFramework=netstandard2.0`
3. **Source generator not running**: Clean and rebuild, ensure generator is referenced correctly
4. **Build hangs**: Normal for large builds - wait up to 45 minutes
5. **Test failures**: May be platform-specific - verify on Windows

### Quick Fixes
- **Format issues**: Run `dotnet format whitespace` and `dotnet format style`
- **StyleCop violations**: Check `.editorconfig` rules and `src/stylecop.json` configuration
- **Analyzer warnings**: Build with `--verbosity normal` to see detailed analyzer messages
- **Missing XML documentation**: All public APIs require XML doc comments per StyleCop rules
- **Package restore issues**: Clear NuGet cache with `dotnet nuget locals all --clear`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactiveui/Splat.DI.SourceGenerator](https://github.com/reactiveui/Splat.DI.SourceGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
