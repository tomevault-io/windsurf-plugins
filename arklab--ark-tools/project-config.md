---
trigger: always_on
description: - Use structured logging with NLog - NEVER use string interpolation in log messages
---

# AI Agent Instructions for Ark.Reference

## Critical Rules

**MUST:**
- Use structured logging with NLog - NEVER use string interpolation in log messages
- Follow Conventional Commits for all commit messages
- Add XML documentation for all public APIs
- Use `CultureInfo.InvariantCulture` when formatting strings for logging
- Dispose Flurl clients after use
- Use `IArkFlurlClientFactory` instead of `IFlurlClientFactory`
- Run `dotnet build` after making changes to verify compilation
- Follow existing patterns - check `Ping` entity implementation as reference
- Place DTOs in `*.API` project, handlers in `*.Application` project
- Add Reqnroll BDD tests for new features
- Use a single `JsonSerializerContext` for all types serialized by the application (Requests, Queries, Messages)
- Configure `JsonSerializerContext` with Ark defaults using a helper method in the Application layer (e.g., `Ex.CreateCoreApiJsonSerializerOptions()`) instead of creating options inline
- Register `JsonSerializerContext` using `TypeInfoResolver` pattern, not `TypeInfoResolverChain`
- Note: `JsonSerializerOptions` get locked when passed to a `JsonSerializerContext` constructor, preventing reuse for multiple contexts - create separate instances for each context
- **BusinessRuleViolations**: Derive from `Ark.Tools.Core.BusinessRuleViolation.BusinessRuleViolation`, specialize with domain-specific properties (e.g., `BookPrintingProcessAlreadyRunningViolation` with `BookId` property). The class name itself serves as the error code
- **Controller Routing**: Always use explicit routes at the controller class level (e.g., `[Route("bookPrintProcess")]` in camelCase). Never use `[controller]` or other implicit routes. Add `[ApiVersion("1.0")]` or appropriate version on the controller. Use sub-routes on action methods (e.g., `[HttpGet("{id}")]`)
- **Async Methods**: Always use `async` and `await` in async methods, even when the only operation is `return await task`. Do NOT optimize by returning the Task directly - this harms stacktrace clarity in production debugging

**MUST NOT:**
- Add new 3rd party dependencies without explicit approval
- Use `FluentAssertions` (deprecated) - use `AwesomeAssertions` instead
- Use `IFlurlClientFactory` directly - use `IArkFlurlClientFactory`
- Use string interpolation in NLog calls (e.g., `_logger.Info($"...")`)
- Put business logic in Controllers - controllers only call handlers
- Skip validation - all Requests/Queries need FluentValidation validators
- Create separate `JsonSerializerContext` for different layers (API, Messages, etc.) - use one unified context
- Use `JsonSourceGenerationOptions` attributes - configure options via helper method instead
- Use generic `BusinessRuleViolationException` with just a code string - create specialized `BusinessRuleViolation` classes instead
- Use implicit controller routes like `[controller]` - always use explicit routes

## About This Project

Ark.Reference is a monorepo template demonstrating the use of Ark.Tools libraries to build modern .NET web APIs. This project serves as a reference implementation and scaffold for creating new LOB (Line of Business) applications.

**Ark.Reference.Core** is the main/default service, serving as the primary reference implementation.

## Build & Test Commands

### Prerequisites

- .NET SDK 10.0.100 (specified in `global.json`)
- Docker (for running integration tests that require SQL Server and Azurite services)

### Basic Commands

```bash
# Restore NuGet packages
dotnet restore

# Build the solution
dotnet build --no-restore --configuration Debug

# Run all tests
dotnet test

# Build in Release mode
dotnet build --no-restore --configuration Release
```

### Running Tests

Tests require SQL Server and Azurite services running via Docker:

```bash
# Start test dependencies
docker-compose up -d

# Run all tests
dotnet test

# Stop dependencies when done
docker-compose down
```

Integration tests are in `Core/Ark.Reference.Core.Tests/` using Reqnroll (BDD framework).

## Project Structure

```
Ark.ReferenceProject/
├── Ark.Reference.Common/                # Shared services across all services (Audit, etc.)
├── Core/                                # Main/default service (Ark.Reference.Core)
│   ├── Ark.Reference.Core.API/          # API contracts (Queries, Requests, Messages)
│   ├── Ark.Reference.Core.Application/  # Business logic and handlers
│   ├── Ark.Reference.Core.Common/       # Shared DTOs, enums, and constants
│   ├── Ark.Reference.Core.Database/     # SQL Server database project
│   ├── Ark.Reference.Core.Tests/        # Integration tests (Reqnroll)
│   └── Ark.Reference.Core.WebInterface/ # Web API controllers and startup
```

Each service follows clean architecture with API, Application, Common, Database, Tests, and WebInterface layers.

## Coding Standards & Conventions

### Language & Framework

- Target Frameworks: .NET 10.0 (multi-targeting enabled in Directory.Build.props)
- Nullable Reference Types: Enabled across all projects
- Treat Warnings as Errors: True (strict compilation)

### Code Quality

- Code analysis is enforced via:
  - `Microsoft.CodeAnalysis.NetAnalyzers`
  - `Meziantou.Analyzer`
- All public APIs must have XML documentation (`GenerateDocumentationFile: true`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ARKlab/Ark.Tools](https://github.com/ARKlab/Ark.Tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
