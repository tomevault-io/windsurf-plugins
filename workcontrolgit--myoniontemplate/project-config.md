---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

MyOnion is a .NET 10 clean architecture reference API template that demonstrates Domain-Driven Design patterns. The solution showcases a custom lightweight mediator, CQRS pattern, FluentValidation, Mapster for mapping, and EF Core with the Repository/Specification pattern. The project is packaged as a VSIX template for Visual Studio 2022.

## Build and Development Commands

### Local Development
```powershell
# Restore dependencies
dotnet restore MyOnion.sln

# Build the solution (use Release before PRs)
dotnet build MyOnion.sln -c Release

# Run the API with hot reload
dotnet watch run --project MyOnion/src/MyOnion.WebApi/MyOnion.WebApi.csproj

# Apply code formatting
dotnet format MyOnion.sln
```

The API will be available at `https://localhost:5001/swagger` with health checks at `/health`.

### Docker Development
```powershell
# Create development HTTPS certificate
./MyOnion/scripts/Create-DevCert.ps1 -Password "devpassword"

# Start API + SQL Server (requires .env file - copy from .env.example)
docker compose --project-directory MyOnion up --build
```

Access the containerized API at `https://localhost:44378/swagger/index.html`.

### VSIX Template Generation
```powershell
# Build VSIX package (outputs to Desktop by default)
./MyOnion/scripts/Build-OnionTemplate.ps1 -Configuration Release

# Install template locally for testing (copies to Visual Studio templates directory)
./MyOnion/scripts/Install-OnionTemplateLocally.ps1 -Configuration Release
```

The build script:
- Copies all source/test projects to `MyOnion/artifacts/TemplateOnionAPI/`
- Performs token replacement (`MyOnion` → `$safeprojectname$`, namespace updates)
- Normalizes project references (`..\..\src\` → `..\`)
- Generates `MyTemplate.vstemplate` XML for each project
- Creates `TemplateOnionAPI.zip` in `MyOnion/vsix/VSIXTemplateOnionAPI/ProjectTemplates/`
- Builds VSIX using MSBuild if `-SkipVsix` not specified

## Architecture

### Clean Architecture Layers

The solution follows onion architecture with strict dependency flow:

```
WebApi → Infrastructure.Persistence, Infrastructure.Shared, Application → Domain
```

**MyOnion.Domain** (`MyOnion/src/MyOnion.Domain/`)
- Pure domain layer with zero external dependencies
- Contains entities (`Department`, `Employee`, `Position`, `SalaryRange`)
- Value objects (`PersonName`, `DepartmentName`, `PositionTitle`) - sealed, immutable classes with validation
- Base classes: `BaseEntity` (Guid ID), `AuditableBaseEntity` (Created/LastModified tracking)
- Domain events (`EmployeeChangedEvent`, `DepartmentChangedEvent`, etc.)

**MyOnion.Application** (`MyOnion/src/MyOnion.Application/`)
- Business logic layer implementing CQRS pattern
- Custom lightweight mediator (`Messaging/Mediator.cs`)
- Feature-based organization under `Features/<FeatureName>/Commands|Queries/`
- FluentValidation validators co-located with commands/queries
- Mapster mapping profiles in `Mappings/`
- Pipeline behaviors for cross-cutting concerns (validation, caching)
- Repository and specification interfaces

**MyOnion.Infrastructure.Persistence** (`MyOnion/src/MyOnion.Infrastructure.Persistence/`)
- EF Core `ApplicationDbContext` with query tracking disabled by default
- Generic repository implementing `IGenericRepositoryAsync<T>`
- Specialized repositories for domain entities
- Value object ownership configuration (e.g., `OwnsOne()` for `PersonName`)
- Database migrations and seed data

**MyOnion.Infrastructure.Shared** (`MyOnion/src/MyOnion.Infrastructure.Shared/`)
- Cross-cutting services: `IDateTimeService`, `IEmailService`, `IMockDataService`
- Registered with transient lifetime

**MyOnion.WebApi** (`MyOnion/src/MyOnion.WebApi/`)
- ASP.NET Core host with controllers in `Controllers/v1/`
- Service registration extensions in `Extensions/`
- Middleware pipeline: error handling → request timing → Swagger → authentication → CORS → health checks
- Feature flags for authentication (`FeatureManagement:AuthEnabled`) and database mode (`FeatureManagement:UseInMemoryDatabase`)
- JWT authentication configured via `Sts:ServerUrl`, `Sts:Audience`, `Sts:ValidIssuer` (optional when auth disabled)

### Custom Mediator Implementation

**Location:** `MyOnion/src/MyOnion.Application/Messaging/`

The solution uses a custom lightweight mediator instead of MediatR:

```csharp
public interface IRequest<TResponse> { }
public interface IRequestHandler<TRequest, TResponse> where TRequest : IRequest<TResponse>
{
    Task<TResponse> HandleAsync(TRequest request, CancellationToken cancellationToken);
}
public interface IPipelineBehavior<TRequest, TResponse> where TRequest : IRequest<TResponse>
{
    Task<TResponse> HandleAsync(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken);
}
```

**Pipeline Execution:**
- Mediator resolves handler via `IServiceProvider`
- Retrieves all `IPipelineBehavior<,>` registrations
- Builds chain of responsibility in **reverse order**
- Behaviors can intercept, validate, cache, or transform requests

**Registered Behaviors:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/workcontrolgit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
