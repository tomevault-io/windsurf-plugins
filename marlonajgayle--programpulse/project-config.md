---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ProgramPulse is a `.NET 10` solution with **two projects**:

- **`src/ProgramPulse.Api`** — an ASP.NET Core minimal-API backend. Vertical-slice architecture with a CQRS-flavored command/query + handler pattern, a `Result`/`Error` functional-error model, EF Core + SQL Server, ASP.NET Core Identity with JWT-in-cookie auth, multi-tenancy, and a transactional outbox for async work (e.g. emails).
- **`src/ProgramPulse.Web`** — a Blazor **WebAssembly** SPA styled with Tailwind CSS v4 plus a hand-written design system. Talks to the API over HTTP through typed API clients; auth rides on HttpOnly cookies.

There is no test project yet.

## Commands

```bash
# Start dependencies (SQL Server on 1433, Mailpit SMTP on 1025 / web UI on 8025)
docker-compose up -d

# Run the API (see env note below — a plain `dotnet run` needs DATABASE_CONNECTION_STRING)
dotnet run --project src/ProgramPulse.Api                        # http  -> http://localhost:5017
dotnet run --project src/ProgramPulse.Api --launch-profile https # https -> https://localhost:7093

# Run the Blazor SPA (launchSettings sets ASPNETCORE_ENVIRONMENT=Local)
dotnet run --project src/ProgramPulse.Web                        # http  -> http://localhost:5031
dotnet run --project src/ProgramPulse.Web --launch-profile https # https -> https://localhost:7208

# Build / restore
dotnet build
dotnet restore

# Tailwind CSS (run from src/ProgramPulse.Web)
npm run css:build   # Styles/app.css -> wwwroot/css/app.css (minified)
npm run css:watch   # rebuild on change while developing

# EF Core migrations
dotnet ef migrations add <Name> --project src/ProgramPulse.Api
dotnet ef database update --project src/ProgramPulse.Api
```

Migrations are applied automatically on startup by `DbInitializer.UseInitializeDatabaseAsync` (only when pending), which also seeds Identity roles.

API docs (Scalar UI) and OpenAPI are mapped via `MapApiDocumentation` and available in non-Production environments.

### Environment & configuration gotchas

**API connection string** — `PersistenceConfiguration.AddPersistence` reads it differently per environment:
- **Local** (and any env that isn't Development/Staging/Production): from `ConnectionStrings:DatabaseConnection` in `appsettings.Local.json`.
- **Development / Staging / Production**: from the `DATABASE_CONNECTION_STRING` environment variable (throws if missing).

The API's `launchSettings.json` sets `ASPNETCORE_ENVIRONMENT=Development`, so a plain `dotnet run` expects `DATABASE_CONNECTION_STRING` to be set. To use the Docker SQL Server with the appsettings connection string, run with `ASPNETCORE_ENVIRONMENT=Local`. Local email points at Mailpit (`localhost:1025`).

**CORS + cookies** — the API's CORS policy (`WebCorsPolicy` in `Program.cs`) reads `Cors:AllowedOrigins` and falls back to `["https://localhost:7208", "http://localhost:5031"]` (the SPA's two profiles), with `AllowCredentials()`. Auth cookies will not flow if the SPA runs on a different origin than the one allowed here — update `Cors:AllowedOrigins` when changing ports.

**SPA → API base URL** — `src/ProgramPulse.Web/wwwroot/appsettings.json` sets `ApiBaseUrl` (default `https://localhost:7093`, the API's https profile). Read in `Program.cs` into the single `HttpClient`'s `BaseAddress`.

**`Frontend:BaseUrl`** (API side, bound to `FrontendOption`) is the SPA's public origin, used to build user-facing links in emails (e.g. the password-reset link) so they point at the SPA rather than the API. Set per environment.

**Generated CSS** — `src/ProgramPulse.Web/wwwroot/css/app.css` is Tailwind output and is **gitignored**. Never edit it; edit `Styles/app.css`. The Web `.csproj` has a `TailwindBuild` target that runs `npm install` (when `node_modules` is missing) and `npm run css:build` before every build, so `dotnet build` regenerates it.

---

## API architecture (`src/ProgramPulse.Api`)

### Vertical slices (`Features/`)
Each feature folder (e.g. `Features/Kpis/Create`) is self-contained and typically holds:
- **`*Command.cs` / `*Query.cs`** — a `record` request, its `*Response` record(s), plus the `*CommandHandler`/`*QueryHandler` class. Handlers take dependencies via primary constructors and expose `HandleAsync(request, ct)` returning `Result` / `Result<T>`.
- **`*Endpoint.cs`** — a class implementing `IEndpoint`. `MapEndpoint` maps the route, then chains `.HasApiVersion(ApiVersions.V1)`, `.WithValidation<TCommand>()`, `.RequireAuthorization(...)`, `.WithName(...)`, `.WithTags(...)`. The handler is injected as a route parameter and the result is returned via `result.ToHttpResult()`.
- **`*CommandValidator.cs`** — a FluentValidation `AbstractValidator<TCommand>`.

Route ids that identify a **parent** travel in the route and are merged into the command in the endpoint, e.g. `command with { ObjectiveId = objectiveId }` in `CreateKpiEndpoint`. Nested collections are routed under the parent (`objectives/{objectiveId}/kpis`) while single-resource update/delete use the flat route (`kpis/{id}`).

### Wiring (`Program.cs`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marlonajgayle/programpulse](https://github.com/marlonajgayle/programpulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
