---
trigger: always_on
description: These guidelines are tailored to this repository. They consolidate prior guidance, remove duplication, and keep only what’s actionable.
---

# Copilot Instructions

These guidelines are tailored to this repository. They consolidate prior guidance, remove duplication, and keep only what’s actionable.

## Project Overview

- **Backend**: .NET 10 Web API (C#) using FastEndpoints in a modular monolith architecture
- **Frontend**: React 19 (TypeScript) with Vite, TanStack Query, React Router, Tailwind CSS, and Shadcn UI
- **Data**: Entity Framework Core 10 with SQL Server
- **Orchestration**: .NET Aspire 13 (AppHost) for local orchestration
- **Testing**: MSTest (Microsoft.Testing.Platform) + AwesomeAssertions + Moq; plus integration and acceptance test projects (Reqnroll)
- **Observability**: Serilog for logging, OpenTelemetry for distributed tracing and metrics

## Repository Layout (high level)

- `src/apps/`: Deployable apps (e.g., the backend Web API)
- `src/aspire/`: Aspire AppHost + ServiceDefaults
- `src/frontend/`: npm workspaces (app + shared UI library)
- `src/modules/`: Business modules (Characters, Elements)
- `src/platform/`: Shared platform + reusable components
- `src/tests/`: Integration and acceptance tests
- `src/source-generators/`: Roslyn source generators + tests

## C# Practices

- Prefer immutability; minimize mutable state. Use explicit access modifiers.
- Async/await for I/O. Log for diagnostics, not control flow.
- Apply SOLID principles. Use DI for all services/abstractions via constructor injection.
- Use records for simple data containers and value objects.
- Enable nullable reference types and rely on them:
  - Use `?` for optional members.
  - Don't add manual null checks for non-nullable DI parameters.
  - Add explicit null checks only when null would cause runtime issues beyond compiler analysis.
- Summary/XML comments for public APIs; use `<inheritdoc />` when implementing documented interfaces.
- Use string interpolation; avoid magic values (prefer constants/enums).
- Prefer pattern matching/switch expressions, LINQ, and collection initializers.
- Dispose `IDisposable` properly (use `using` statements/declarations).
- Follow naming conventions: PascalCase for types/public members, camelCase for local variables/private fields.

## Architecture: Modular Monolith

- Organize by business capability with clear module boundaries:
  - **Elements Module**: Game data (classes, abilities, features, rules)
  - **Characters Module**: Character creation and management
  - **Platform Layer**: Shared infrastructure (hosting, logging, data, eventing)
- Each module follows internal layering:
  - `Domain`: Entities, value objects, aggregates, domain logic
  - `Data`: Repository interfaces and abstractions
  - `Data.EntityFramework`: EF Core configurations, DbContext, migrations
  - `Data.EntityFramework.MigrationService`: Migration worker used by Aspire at startup
  - `Data.EntityFramework.EventProcessing`: Outbox/event processing persistence and workers
  - `Endpoints`: FastEndpoints for API exposure
  - `Integration`: Public contracts for inter-module communication
- Strict boundaries: modules interact via interfaces or domain events; avoid leaking internals.
- Favor internal visibility; expose only what’s necessary.
- Use feature folders/namespaces for related functionality.
- Keep a minimal, stable shared kernel in the Platform layer.
- Apply DDD concepts where beneficial (entities, aggregates, value objects).
- Write integration tests for module interactions and endpoint behavior.

## Aspire 13 (local orchestration)

- **Local Development**:
  - Aspire AppHost entrypoint is `src/aspire/Starlights.AppHost/AppHost.cs`.
  - Use Aspire service discovery for resource connections (SQL Server, backend endpoints).
  - SQL Server container runs on static port `61070` for consistency.
  - Migration workers run at startup (in run mode) and the backend waits for completion.
  - Frontend is started by Aspire as a JavaScript app; it can also be exposed via a Dev Tunnel (explicit start).
  - Monitor via the Aspire dashboard (logs/metrics/traces).

## Entity Framework Core + SQL Server

- **Configuration**: All entities must have explicit `IEntityTypeConfiguration<T>` implementations in respective `*.Data.EntityFramework` projects.
- **Query Performance**:
  - Add indexes for frequently queried columns
  - Use efficient LINQ queries; avoid client evaluation and N+1 queries
  - Project with `.Select()` to load only needed columns
  - Use `AsNoTracking()` for read-only queries
- **Data Transfer**:
  - Load only needed data
  - Avoid cartesian explosion in joins
  - Batch operations where possible
  - Paginate large result sets
- **Runtime**:
  - Disable change tracking when not needed
  - Consider compiled queries for hot paths
  - Use value converters and shadow properties judiciously
- **Caching**: Cache hot data appropriately; plan cache invalidation strategy.
- **Diagnostics**: Enable EF logging; inspect generated SQL; use SQL Profiler/App Insights; measure before optimizing.
- **Migrations**:
  - Use EF Core migrations for schema changes
  - Add migrations in respective module `*.Data.EntityFramework` projects
  - Migration workers run automatically at startup via Aspire
- **General**:
  - Use transactions for multi-step operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swdriessen/starlights](https://github.com/swdriessen/starlights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
