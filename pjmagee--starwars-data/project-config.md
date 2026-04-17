---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

All projects live under `src/` with the solution at `src/StarWarsData.slnx`. Requires .NET 10 SDK (see `src/global.json`).

```bash
# Build everything
dotnet build src/StarWarsData.slnx

# Run the Aspire orchestrator (starts API + Frontend)
dotnet run --project src/StarWarsData.AppHost

# Run just the API
dotnet run --project src/StarWarsData.ApiService

# Run just the frontend
dotnet run --project src/StarWarsData.Frontend

# Run just the admin app
dotnet run --project src/StarWarsData.Admin
```

### Aspire MCP

When the AppHost is running, use the **Aspire MCP server** (`mcp__aspire__*`) to interact with the running application:

- `list_resources` — see all running services and their status
- `list_console_logs` / `list_structured_logs` — read service logs and diagnose issues
- `list_traces` / `list_trace_structured_logs` — inspect distributed traces
- `execute_resource_command` — restart services or trigger the HTTP commands defined in the AppHost (ETL pipeline phases, index creation, etc.)

The AppHost defines HTTP commands on the **admin** resource for every ETL pipeline phase (1a–8). These are visible in the Aspire dashboard and executable via the MCP.

## Tests

Tests use xUnit with Testcontainers for MongoDB (requires Docker running). There are no unit tests — all tests are integration tests that spin up real MongoDB containers.

```bash
# Run all tests
dotnet test src/StarWarsData.Tests

# Run a single test class
dotnet test src/StarWarsData.Tests --filter "FullyQualifiedName~RecordServiceTests"

# Run a single test method
dotnet test src/StarWarsData.Tests --filter "FullyQualifiedName~RecordServiceTests.Search_returns_matching_pages"
```

Test fixtures: `ApiFixture` (shared MongoDB container with seed data for most tests, xUnit collection `"Api"`) and `MongoFixture` (for relationship graph tests, collection `"Mongo"`). Both use `IAsyncLifetime` to start/stop Testcontainers.

## Architecture

**Aspire-orchestrated app** with five runtime components:

- **AppHost** — .NET Aspire orchestrator. Wires MongoDB connection, OpenAI key, and defines HTTP commands for the ETL pipeline phases (visible in the Aspire dashboard). Also manages a MongoDB MCP sidecar container.
- **ApiService** — ASP.NET Core API. Hosts the AI agent (via Microsoft.Agents.AI + AGUI), MCP client for MongoDB tools, and feature endpoints. Organized into `Features/` folders: `CharacterTimelines`, `Chat`, `GalaxyMap`, `KnowledgeGraph`, `Pages`, `RPG`, `Search`, `Timeline`, `User`.
- **Admin** — Blazor Interactive Server app (MudBlazor UI) for ETL pipeline management, Hangfire background jobs, and admin dashboards. Organized into `Features/` folders: `Admin`, `KnowledgeGraph`, `Search`. Also has `Components/Pages/`, `Components/Layout/`, `Components/Shared/`.
- **Frontend** — Blazor Interactive Server with MudBlazor UI. Authenticates via Keycloak OIDC. Uses `Components/Pages/`, `Components/Layout/`, `Components/Shared/`.
- **MongoDbMigrations** — Run-once container (`mongo:latest`) that executes `mongosh migrate.js` against the target database. Tracked in a `migrations` collection — re-runs are idempotent. Scripts live in `src/StarWarsData.MongoDbMigrations/`.

**Shared libraries:**

- **Models** — Entities (`Page`, `Infobox`, `TimelineEvent`, `RelationshipEdge`, `CharacterTimeline`, `ArticleChunk`, etc.), DTOs, and `SettingsOptions` configuration.
- **Services** — All business logic organized by feature: `AI/Toolkits`, `Admin`, `CharacterTimelines`, `Chat`, `GalaxyMap`, `KnowledgeGraph`, `Pages`, `RPG`, `Search`, `Shared`, `Timeline`, `User`.
- **ServiceDefaults** — Aspire service defaults (OpenTelemetry, resilience).

**Folder convention:** ApiService, Admin, and Services use feature-based folder organization (`Features/<FeatureName>/` or `<FeatureName>/`), not layer-based (no `Controllers/`, `Repositories/` top-level folders).

## Key Patterns

**AI stack**: Microsoft.Extensions.AI (`IChatClient`) + Microsoft.Agents.AI (`AIAgent`, `AITool`) + OpenAI SDK. No Semantic Kernel — do not add SK packages.

**AI Agent pipeline** (in `ApiService/Program.cs`): Topic guardrail classifier -> AI agent with tool registry (ComponentToolkit, DataExplorerToolkit, GraphRAGToolkit, WikiSearchProvider, MongoDB MCP tools) -> AGUI streaming endpoint at `/kernel/stream`.

**MongoDB**: External self-hosted server (not Aspire-managed). Connection string assembled from parameters in AppHost. Single database configured via `SettingsOptions.DatabaseName` with namespaced collections: `raw.*`, `timeline.*`, `kg.*`, `search.*`, `genai.*`, `chat.*`, `territory.*`, `galaxy.*`, `admin.*`, `hangfire.*`. Hangfire collections live in the same database, namespaced by the `Prefix` option (default `"hangfire"`). Collection names are defined in the `Collections` static class in `Settings.cs`. Two database environments: `starwars-dev` (default, used in local/dev — safe for experimentation) and `starwars` (production). The default in code is `starwars-dev`; production overrides via `appsettings.json` or env var `Settings__DatabaseName`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pjmagee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
