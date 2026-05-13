---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Freetool is an open-source internal tools platform (Retool alternative) built with F# and ASP.NET Core. The app uses **Onion Architecture** with functional design patterns, and its UI is server-rendered from F# Oxpecker views with static assets under `src/Freetool.Api/wwwroot`.

## Shared MCP Skills

Shared internal-tools skills are served by the deployed `internal-tools-mcp` server.

- Codex reads [`.codex/config.toml`](./.codex/config.toml).
- Claude Code reads [`.mcp.json`](./.mcp.json) and [`.claude/settings.json`](./.claude/settings.json).
- No bearer token or local secret bootstrap is required before starting either client.
- Shared internal-tools workflows are now surfaced locally as thin `.agents/skills/*/SKILL.md` stubs that delegate to `internal-tools.use_workflow`.
- If the right shared workflow is not obvious, call `internal-tools.recommend_workflows` first, then call `internal-tools.use_workflow` for the top match before editing.
- Use `app-observability` for owner-safe app telemetry triage through `internal-tools logs`, `internal-tools traces`, `internal-tools metrics`, and `internal-tools alerts` before reaching for backend-specific debugging.
- Before editing EF Core mappings, repositories, or `DbContext` code, load `entity-framework-fsharp` first.
- Before editing schema or migration code, load `db-migrations` first.
- Consult the matching shared stub before infra, deploy, secret, OpenAPI, or review work when the task clearly maps to one of those workflows.
- After loading a primary workflow, also consult related shared stubs such as `domain-driven-design`, `event-sourcing-audit`, and `otel-tracing` when they exist in this repo and the task touches business rules, audit/events, or new request paths.
- Keep `freetool-controller-authoring`, `freetool-iap-auth-architecture`, and `freetool-openfga-hexagonal-architecture` as the full repo-local override skills.
- Use those repo-local skills instead of shared `new-controller`, `iap-auth`, and `openfga` in this repo.

## Essential Commands

### Backend (F#/.NET)
```bash
# Restore dependencies
dotnet restore Freetool.sln

# Build all projects
dotnet build Freetool.sln -c Release

# Run API locally (serves the server-rendered UI and static assets)
dotnet run --project src/Freetool.Api/Freetool.Api.fsproj

# Run all tests
dotnet test Freetool.sln

# Run tests for specific project
dotnet test src/Freetool.Domain/test/Freetool.Domain.Tests.fsproj

# Run specific test by name filter
dotnet test --filter "FullyQualifiedName~NameOfTest"

# Format code
dotnet format Freetool.sln
```

### UI
The UI is served by the ASP.NET Core host from `src/Freetool.Api/src/Ui` and `src/Freetool.Api/wwwroot`; there is no separate browser workspace to install or build.

### Docker
```bash
# Start all services (API, OpenFGA, Aspire Dashboard)
docker compose -f docker-compose.dev.yml up --build

# Start specific service
docker compose -f docker-compose.dev.yml up --build freetool-api

# View logs
docker compose -f docker-compose.dev.yml logs -f freetool-api
```

**Service URLs:**
- API: http://localhost:5002
- OpenAPI UI: http://localhost:5002/openapi
- OTEL/Aspire Dashboard: http://localhost:18888
- OpenFGA: http://localhost:8090
- OpenFGA Playground: http://localhost:3030

## Architecture Overview

### Layered Onion Architecture

The F# backend follows strict dependency inversion - all dependencies point inward toward the domain:

**1. Domain Layer** (`src/Freetool.Domain/`) - Innermost layer, zero dependencies
- **Entities/**: Aggregates with business rules (User, App, Resource, Folder, Space, Run)
  - Each entity is an F# record with a `State` field (business data) and `UncommittedEvents` list
  - Domain methods are pure functions that validate, update state, and collect events
  - Example: `App.updateName` validates the name, updates the aggregate, and adds an `AppUpdatedEvent`
- **ValueObjects/**: Immutable value types with validation (UserId, Email, AppName, etc.)
- **Events/**: Domain events representing business facts (UserCreatedEvent, AppUpdatedEvent, etc.)
- **Services/**: Pure domain services for complex business logic
- **Types.fs**: Core domain types (`DomainError` discriminated union, etc.)
- **EventSourcingAggregate.fs**: Base aggregate pattern for event collection

**2. Application Layer** (`src/Freetool.Application/`) - Orchestration and use cases
- **Commands/**: Command discriminated unions define all operations
  - Pattern: `type UserCommand = CreateUser of ... | UpdateUser of ... | DeleteUser of ...`
  - Result types: `type UserCommandResult = UserResult of UserDto | UsersResult of PagedResult<UserDto> | ...`
- **Handlers/**: Handler modules implement command pattern matching
  - Pattern: Module-based handlers (e.g., `UserHandler`) with `handleCommand` function
  - Each handler matches command cases and orchestrates domain + repository calls
  - Handlers implement `IGenericCommandHandler<TRepository, TCommand, TResult>` interface
- **DTOs/**: Data transfer objects for API boundary (no business logic)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wonderlydotcom/internal-tool-freetool](https://github.com/wonderlydotcom/internal-tool-freetool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
