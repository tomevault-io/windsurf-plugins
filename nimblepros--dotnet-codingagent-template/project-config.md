---
trigger: always_on
description: This is a placeholder. Review the README.md and /src files for details about the repository purpose and structure and update this overview accordingly.
---

# Agent Instructions

## Project Overview

This is a placeholder. Review the README.md and /src files for details about the repository purpose and structure and update this overview accordingly.

## Target Framework & Language

- **Framework**: .NET 10.0 (`net10.0`)
- **Language Version**: Latest C# features enabled
- **Nullable Reference Types**: Enabled project-wide
- **Implicit Usings**: Enabled
- **Treat Warnings as Errors**: Enabled globally

## Project Structure

```
├── src/              # Source code projects
├── tests/            # Test projects
├── Directory.Build.props    # Shared properties for all projects
├── Directory.Packages.props # Central Package Management (CPM) configuration
└── Template.slnx            # Solution file (XML format)
```

## Build Configuration

### Central Package Management

- Uses **MSBuild Central Package Management** (`ManagePackageVersionsCentrally=true`)
- All package versions are managed in [Directory.Packages.props](Directory.Packages.props)
- Individual projects reference packages WITHOUT version numbers in their `.csproj` files
- When adding a new package: Add `<PackageVersion Include="PackageName" Version="x.y.z" />` to [Directory.Packages.props](Directory.Packages.props)

### Compiler Settings

- **Treat Warnings As Errors**: Enabled (`TreatWarningsAsErrors=true`)
- **XML Documentation**: Warning 1591 (missing XML comments) is suppressed via `NoWarn`
- All projects automatically inherit settings from [Directory.Build.props](Directory.Build.props)

## Code Style Conventions

### File-Scoped Namespaces (Enforced)

```csharp
namespace MyProject.Feature;  // ✅ Use this (warning if not used)

// NOT this:
namespace MyProject.Feature   // ❌ Avoid
{
}
```

### Naming Conventions

- **Private fields**: Use underscore prefix with camelCase (`_myField`)
- **Constants**: Use PascalCase
- **Local variables**: Use `var` for built-in types and when type is apparent
- **Interfaces**: Prefix interfaces with `I` (e.g., `IRepository`)
- **Async Methods**: Suffix async methods with `Async` (e.g., `GetByIdAsync`)
- **Code Blocks**: Always use `{}` for blocks except single-line exits (e.g., `return`, `throw`)

### Code Formatting

- **Indentation**: 2 spaces (not tabs) for all `.cs` files
- **Line endings**: CRLF (Windows)
- **Encoding**: UTF-8 with BOM for code files
- **Braces**: Always use braces for control flow (`csharp_prefer_braces = true`)
- **Opening braces**: On new line (Allman style - `csharp_new_line_before_open_brace = all`)
- **Namespace directives**: File-scoped, outside namespace (enforced as warning)

### Modern C# Preferences

- **Primary constructors**: Preferred; ALWAYS assign to readonly fields
- **Top-level statements**: Preferred for program entry points
- **Pattern matching**: Prefer over `is` with cast and `as` with null check
- **Expression-bodied members**: Use for properties, indexers, accessors (but NOT for methods)

### Code Convention Enforcement

- Run `dotnet format` to auto-format code according to conventions
- Use analyzers from [Directory.Build.props](Directory.Build.props) to catch violations during build
- Consider using NSDepCop for architectural rules

### Configuration Management

- Use `appsettings.json` for default configuration
- Use `appsettings.{Environment}.json` for environment-specific overrides
- Use `IConfiguration` and `IOptions<T>` for accessing configuration in code
- Create strongly-typed settings classes for complex configurations
- Use `User Secrets` for sensitive data in development
- Never hardcode sensitive information in source code

## Logging

- Use `Microsoft.Extensions.Logging` for logging
- Log at appropriate levels: `Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical`
- Include relevant context in log messages
- Use Structured logging where possible
- Avoid logging sensitive information
- Include correlation IDs for tracing requests across services when possible

## Guard Clauses

- Use `Ardalis.GuardClauses` for input validation in non-UI code
- Prefer `FluentValidation` for complex validation scenarios, especially in UI layers
- Invalid inputs in UI layers should result in user-friendly error messages rather than exceptions
- Invalid inputs in non-UI layers should throw appropriate exceptions using guard clauses
- HTTP methods should return suitable status codes for non-success results (e.g., 400 Bad Request)

## Security

- Follow OWASP guidelines for secure coding practices
- Validate and sanitize all user inputs
- Use parameterized queries or ORM features to prevent SQL injection
- Implement proper authentication and authorization mechanisms
- Use HTTPS for all communications

## Coding Practices

- Write unit testable code wherever possible
- Offer to add tests to generated code if applicable - place such tests in the `tests/` folder
- Only test code that you control and that has business logic; do not test framework code
- Keep classes focused on a single responsibility
- Use Dependency Injection for managing dependencies
- Prefer composition over inheritance
- Use async/await for I/O-bound operations
- Keep methods short and focused; ideally under 20 lines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NimblePros/dotnet-codingagent-template](https://github.com/NimblePros/dotnet-codingagent-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
