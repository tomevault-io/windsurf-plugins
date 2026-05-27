---
trigger: always_on
description: - Install the Aspire workload before using the AppHost: `dotnet workload install aspire`
---


# Copilot Instructions

## Build, test, and lint commands

### Backend and solution
- Install the Aspire workload before using the AppHost: `dotnet workload install aspire`
- Restore packages: `dotnet restore`
- Build the solution: `dotnet build --configuration Release`
- Use `dotnet build` as the .NET lint gate. `Directory.Build.props` enables `AnalysisMode=All`, `EnforceCodeStyleInBuild`, `TreatWarningsAsErrors`, and SonarAnalyzer across all `.csproj` files.
- Run all tests: `dotnet test --configuration Release`
- Run one test project: `dotnet test Tests\Domain.UnitTests\Domain.UnitTests.csproj --configuration Release`
- Run a single test class or method: `dotnet test Tests\Domain.UnitTests\Domain.UnitTests.csproj --configuration Release --filter "FullyQualifiedName~Domain.UnitTests.BookTests"`
- Run only the integration suite: `dotnet test Tests\CleanArchitecture.IntegrationTests\CleanArchitecture.IntegrationTests.csproj --configuration Release --filter "FullyQualifiedName~CleanArchitecture.IntegrationTests.BookIntegrationTests"`
- Run the full local stack through Aspire: `dotnet run --project CleanArchitecture.Aspire\CleanArchitecture.AppHost\CleanArchitecture.AppHost.csproj`
- Run only the API: `dotnet run --project CleanArchitecture.Presentation\API\CleanArchitecture.Api.csproj`
- Add a migration: `dotnet ef migrations add <Name> --project CleanArchitecture.Infrastructure.Persistence --startup-project CleanArchitecture.Presentation\API`
- Apply migrations manually: `dotnet ef database update --project CleanArchitecture.Infrastructure.Persistence --startup-project CleanArchitecture.Presentation\API`

### Admin app
- Work from `CleanArchitecture.Presentation\admin`
- Install dependencies: `pnpm install`
- Start the admin app: `pnpm dev`
- Build the admin app: `pnpm build`
- Lint the admin app: `pnpm lint` (eslint with `--max-warnings=0`)
- Regenerate API hooks and Zod schemas: `pnpm generate` (runs orval against `http://localhost:5049/openapi/v1.json`)

## High-level architecture

- `CleanArchitecture.sln` is split into Core (`CleanArchitecture.Domain`, `CleanArchitecture.Application`), Infrastructure (`CleanArchitecture.Infrastructure`, `CleanArchitecture.Infrastructure.Persistence`), Presentation (`CleanArchitecture.Presentation\API`, `CleanArchitecture.Presentation\admin`), Aspire (`CleanArchitecture.AppHost`, `CleanArchitecture.ServiceDefaults`), and four test projects.
- `CleanArchitecture.Presentation\API\Program.cs` is the backend composition root. It adds Aspire service defaults first, then wires the Application, Infrastructure, Infrastructure.Persistence, and Presentation layers through their `Add*Services()` extension methods.
- The API is a versioned Minimal API. Endpoints are grouped under `/api/v{version:apiVersion}` and then mapped by feature extension classes such as `BookEndpoints`.
- `CleanArchitecture.Infrastructure.Persistence` owns EF Core concerns: `ApplicationDbContext`, entity configurations, migrations, SaveChanges interceptors (`AuditableEntityInterceptor` for timestamps, `DispatchDomainEventsInterceptor` for publishing domain events after save), and the unit-of-work implementation.
- `CleanArchitecture.Aspire\CleanArchitecture.AppHost\AppHost.cs` is the real local-dev entry point. It has three environment modes: **Development** starts Postgres + PgAdmin, Keycloak with realm import, the API, and the Next.js admin app. **Testing** starts only the API and an ephemeral Postgres database (no Keycloak; `TestAuthHandler` grants all roles). **Production** targets Azure Container Apps, PostgreSQL Flexible Server, Key Vault, and Application Insights.
- `CleanArchitecture.Aspire\CleanArchitecture.ServiceDefaults\Extensions.cs` adds the shared cross-cutting runtime behavior: Serilog, OpenTelemetry, service discovery, resilience handlers, and health endpoints.
- `CleanArchitecture.Presentation\admin` is a Next.js 16 App Router admin app acting as a **Backend for Frontend (BFF)**. Only the frontend and Keycloak admin UI are publicly exposed; the .NET API remains private. The admin uses NextAuth with Keycloak, TanStack Query for server state, and orval-generated API clients/Zod schemas under `src\lib\api` and `src\lib\api\zod`.

## Key conventions

### Architecture and layer boundaries
- Preserve the layer boundaries enforced by `Tests\Architecture.UnitTests\ArchitectureTests.cs`. Domain → no project references. Application → Domain only. Infrastructure → Application (+ Domain). Presentation → Application + Infrastructure.
- Backend use cases follow CQRS with the **Mediator** source-generator package (not MediatR) and FluentValidation auto-registration from `CleanArchitecture.Application\DependencyInjection.cs`. New features should follow the existing `Entities\<Aggregate>\Commands\<Verb>\` and `Entities\<Aggregate>\Queries\<Verb>\` layout, each containing a command/query record, handler, and validator.

### Domain model
- Domain entities inherit from `Entity` → `AggregateRoot` (or their auditable variants `EntityAuditable` → `AggregateRootAuditable` which add `CreatedDate`/`UpdatedDate`). `Entity` provides identity-based equality and a `DomainEvents` collection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iPazooki/CleanArchitecture](https://github.com/iPazooki/CleanArchitecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
