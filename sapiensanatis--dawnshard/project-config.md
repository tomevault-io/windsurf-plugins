---
trigger: always_on
description: Dawnshard is a game server emulator for [Dragalia Lost](https://en.wikipedia.org/wiki/Dragalia_Lost). This document gives a coding agent the context needed to work efficiently in this repository.
---

# Agent Instructions for Dawnshard

Dawnshard is a game server emulator for [Dragalia Lost](https://en.wikipedia.org/wiki/Dragalia_Lost). This document gives a coding agent the context needed to work efficiently in this repository.

## Repository Structure

```
Dawnshard/
├── DragaliaAPI/              # Main ASP.NET Core 10 game server (primary focus)
│   ├── DragaliaAPI/          # Application entrypoint, controllers, services
│   ├── DragaliaAPI.Database/ # EF Core DbContext, entities, migrations, repositories
│   ├── DragaliaAPI.Shared/   # Shared types, MessagePack models, source generators
│   ├── DragaliaAPI.Test/     # Unit tests
│   ├── DragaliaAPI.Integration.Test/ # Integration tests (uses real DB + Redis)
│   └── DragaliaAPI.Database.Test/   # Database/ORM tests
├── PhotonStateManager/       # Microservice for Photon co-op room state (ASP.NET Core)
├── PhotonPlugin/             # Photon Server plugin (C# DLL, not an ASP.NET app)
├── MaintenanceWorker/        # Cloudflare Worker written in Rust/WASM
├── Website/                  # SvelteKit 2 + Svelte 5 + Tailwind CSS frontend
├── Aspire/                   # .NET Aspire AppHost for local development orchestration
├── Shared/                   # Types shared between DragaliaAPI and PhotonPlugin
├── .editorconfig             # Enforced C# code style rules
├── Directory.Packages.props  # Centralized NuGet package versions (do not add versions in .csproj)
└── global.json               # Pins .NET SDK version to 10.0.100
```

## Technology Stack

- **Runtime:** .NET 10 / C# (backend), Node.js + pnpm (frontend)
- **Web framework:** ASP.NET Core 10
- **Database:** PostgreSQL 17 via Entity Framework Core 10 + Npgsql
- **Cache/Session:** Redis 7 (redis-stack image)
- **Serialization:** MessagePack (game protocol), JSON (REST/admin)
- **Background jobs:** Hangfire
- **Logging:** Serilog (structured, with Seq sink)
- **Frontend:** SvelteKit 2 (`@sveltejs/kit` 2.x), Svelte 5, Tailwind CSS, shadcn-svelte, TypeScript
- **Local orchestration:** .NET Aspire (starts DB, cache, API, and frontend together)
- **Containerization:** Docker (multi-stage, chiseled ASP.NET base images)
- **Testing:** xUnit v3, FluentAssertions, Playwright (E2E)

## Building the Project

### Prerequisites

- .NET SDK 10.0.100 (pinned in `global.json`)
- Docker Desktop (required for integration tests and Aspire local dev)
- Node.js + pnpm (for `Website/`)
- .NET Aspire workload: `dotnet workload install aspire`

### DragaliaAPI

```bash
# Restore and build
dotnet build DragaliaAPI.slnx
```

### Website

```bash
cd Website
pnpm install
pnpm build
pnpm dev   # development server
```

### Docker image

```bash
docker build -f DragaliaAPI/DragaliaAPI/Dockerfile -t dragalia-api .
```

## Running Tests

### C# tests

```bash
# All tests
dotnet test --solution DragaliaAPI.Linux.slnf -c Release

# Single project
dotnet test --project DragaliaAPI/DragaliaAPI.Test/DragaliaAPI.Test.csproj -c Release

# Integration tests (requires Docker for Testcontainers)
dotnet test --project DragaliaAPI/DragaliaAPI.Integration.Test/DragaliaAPI.Integration.Test.csproj -c Release

# Run specific integration test class (accepts * as wildcard)
dotnet test --project DragaliaAPI/DragaliaAPI.Integration.Test/DragaliaAPI.Integration.Test.csproj -c Release --filter-class DragaliaAPI.Integration.Test.Features.Event.EventSummonTest

# Run specific integration test method (accepts * as wildcard)
dotnet test --project DragaliaAPI/DragaliaAPI.Integration.Test/DragaliaAPI.Integration.Test.csproj -c Release --filter-method DragaliaAPI.Integration.Test.Features.Event.EventSummonTest.GetData_ReturnsBoxSummonData
```

Test framework is **xUnit v3** using the Microsoft Testing Platform runner. Assertions use **FluentAssertions**. **Moq is no longer used** — new tests should be written as integration tests rather than unit tests.

### Website tests

```bash
cd Website
pnpm test        # Playwright E2E tests
pnpm lint        # ESLint + Prettier check
```

## Linting and Formatting

### C# — CSharpier

The project uses [CSharpier](https://csharpier.com/) (not `dotnet format`) as the canonical C# formatter. CI will fail if formatting is off.

```bash
# Check formatting
dotnet tool restore
dotnet csharpier check .

# Auto-fix formatting
dotnet csharpier format .
```

> Always run `dotnet csharpier .` before committing C# changes.

### Website — ESLint + Prettier

```bash
cd Website
pnpm lint        # check
pnpm format      # auto-fix
```

## C# Code Conventions

These rules are enforced by `.editorconfig` and will produce compiler warnings/errors if violated:

- **No `var`** for built-in types; use explicit types everywhere.
- **File-scoped namespaces** (`namespace Foo.Bar;` not `namespace Foo.Bar { ... }`).
- **Braces required** on all control flow statements.
- **Readonly fields** where possible.
- **Pattern matching** preferred over `is`/`as` casts.
- **Null-coalescing / null-propagation** required (`?.`, `??`).
- **No unused parameters** — all parameters must be used.
- **`this.` qualifier** is mandatory for field access (silent preference; follow existing code).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SapiensAnatis/Dawnshard](https://github.com/SapiensAnatis/Dawnshard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
