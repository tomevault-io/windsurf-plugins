---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Running the Full Stack (Recommended)
```bash
cd src/AppointMe.Aspire && dotnet run
```
This starts all services via .NET Aspire: SQL Server, Azure Service Bus Emulator, Keycloak, Mailpit, API, and Frontend.

### Backend Only
```bash
dotnet build AppointMe.sln        # Build entire solution
dotnet run --project src/AppointMe.Api  # Run API
```

### Frontend Only
```bash
cd src/AppointMe.Frontend
yarn install
yarn dev          # Development server (port 5173)
yarn build        # Production build
yarn lint         # ESLint
yarn generate:api # Regenerate API client from OpenAPI spec (see /regenerate-api skill)
```

### Regenerating the frontend API client after backend contract changes

After any change that alters the OpenAPI surface — new/renamed/removed endpoint, changed `*Request.cs`/`*Response.cs` shape, changed route or HTTP verb, changed auth/permission attributes on an endpoint — invoke the `/regenerate-api` skill **in the same task**. It waits for the API to be reachable on `https://localhost:7233`, runs `yarn generate:api`, and reports the diff on `src/AppointMe.Frontend/src/api/`. The backend must be restarted (Aspire dashboard → restart `appointme-api`, or relaunch `dotnet run` in `src/AppointMe.Aspire`) so orval reads the new contract — a reachable-but-stale backend silently produces a stale client.

Do **not** invoke `/regenerate-api` for backend-only changes that don't affect the contract (internal handlers, domain events, EF config, migrations, tests).

### Testing
```bash
dotnet test                                    # All tests
dotnet test src/CRM/AppointMe.Crm.Tests        # Single test project
dotnet test --filter "FullyQualifiedName~TestName"  # Single test
```

## Architecture

This is a **modular monolithic** .NET 10 application following Domain-Driven Design principles.

### Module Structure
```
src/
├── AppointMe.Api/           # ASP.NET Core API entry point
├── AppointMe.Aspire/        # .NET Aspire orchestrator for local dev
├── AppointMe.Shared/        # Shared domain abstractions and value objects
├── Identity/AppointMe.Identity/           # User auth module (Keycloak)
├── Organizations/AppointMe.Organizations/ # Companies & employees
├── CRM/AppointMe.Crm/                     # Customer management
├── Booking/AppointMe.Booking.Contracts/   # Booking contracts (TBD)
└── AppointMe.Frontend/      # React/Vite/TypeScript frontend
```

### Key Patterns

**Bounded Contexts**: Each module has its own `DbContext` with separate database schema:
- `IdentityDbContext` (schema: "identity")
- `OrganizationsDbContext` (schema: "organizations")
- `CrmDbContext` (schema: "crm")

**Vertical Slice Architecture**: Inside each module, code is organized by feature/use case rather than by technical layer. Each slice owns everything required to serve that use case and lives in its own folder named after the action (verb-first, CRUD-style for projections).

Example layout:
```
Customers/
├── Customer.cs                       # aggregate root
├── CustomerId.cs
├── RegisterCustomer/                 # one slice per use case
│   ├── RegisterCustomer.cs           # domain factory (extension on aggregate)
│   ├── RegisterCustomerCommand.cs
│   ├── RegisterCustomerCommandHandler.cs
│   ├── RegisterCustomerEndpoint.cs
│   └── RegisterCustomerRequest.cs
├── UpdateCustomer/
├── DeleteCustomer/
├── GetCustomers/                     # queries are slices too
└── Database/                         # shared EF config / Dapper queries
```

Slice rules:
- Event handlers live in the slice whose domain operation they invoke (e.g. `CustomerRegisteredEventHandler` sits in `CreateAttendee/` because it calls `Attendee.Create`), **not** in a shared `Handlers/` folder.
- Domain operations on aggregates are expressed as extension methods co-located with the slice (`extension(Aggregate)` for factories, `extension(Aggregate instance)` for mutators) — see `RegisterCustomer.cs`, `UpdateCustomer.cs`, `DeleteCustomer.cs`.
- Use CRUD-style verbs (`Create`, `Update`, `Delete`) for projection/synchronization slices, and domain-specific verbs (`Register`, `Schedule`, `Cancel`) for slices that own business intent on the source aggregate.
- Shared infrastructure (EF type configurations, Dapper queries reused across slices, ID types, the aggregate itself) lives alongside the slices in the parent feature folder, not inside any one slice.

**Endpoint Convention**: Implement `IEndpoint` interface. Endpoints are auto-discovered via Scrutor.

**Module Registration**: Each module exposes extension methods:
```csharp
.AddIdentityModule()
.AddCompaniesModule()
.AddCrmModule()
```

**Domain Events**: Aggregate roots inherit from `AggregateRoot` and raise `IDomainEvent` events. Wolverine handles async messaging.

**CQRS Pattern**:
- Writes use Entity Framework Core
- Reads use Dapper with `IDbConnectionFactory`

**Authentication**: Hybrid auth scheme (JWT Bearer for API calls, Cookies for browser flows) with Keycloak.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bravodev-hub/appointme](https://github.com/bravodev-hub/appointme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
