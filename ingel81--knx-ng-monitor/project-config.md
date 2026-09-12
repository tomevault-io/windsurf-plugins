---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KNX-NG-Monitor is a KNX bus monitoring tool with a web UI that displays, historizes and presents KNX telegrams in real time. It ships as a Docker image or as a single self-contained binary. As of v0.1.0 the parser supports ETS 4 / 5 / 6, password-protected projects (incl. ETS6 PBKDF2/AES wrapping) and `.knxkeys` keyring decryption.

## Tech Stack

### Backend
- **.NET 9** (ASP.NET Core Web API) — see `backend/global.json` (none — uses installed SDK)
- **Entity Framework Core 9** with SQLite (`Microsoft.EntityFrameworkCore.Sqlite`)
- **SignalR** for real-time telegram broadcasting
- **[Knx.Falcon.Sdk](https://www.nuget.org/packages/Knx.Falcon.Sdk)** for KNX bus integration
- **JWT auth** (access + refresh tokens, secret auto-generated)
- **SharpZipLib 1.4.2** for AES-encrypted ZIP entries (ETS6-Password + KNX Secure)

### Frontend
- **Angular 20** (standalone components)
- **Angular Material** (dialogs, forms, theming)
- **AG-Grid Community** (live-view, virtual scrolling)
- **RxJS**, SignalR client, Angular CDK virtual scrolling

### Database
- **SQLite** — embedded, single file under `./data/knxmonitor.db`

## Project Structure

```
knx-ng-monitor/
├── backend/
│   ├── KnxMonitor.Api/                   # ASP.NET Core Web API + SignalR Hubs + Program.cs (DI)
│   ├── KnxMonitor.Core/                  # Domain layer (Entities, Interfaces, DTOs, Enums)
│   ├── KnxMonitor.Infrastructure/        # EF Core, repositories, KNX bus, project import, adapter to ProjectParser
│   ├── KnxMonitor.ProjectParser/         # Standalone parser library (ETS 4/5/6 loaders, FeatureDetector, KeyringReader, ZipHandler)
│   ├── KnxMonitor.ProjectParser.Tests/   # xUnit tests (206, ~99 % line cov), uses Xunit.SkippableFact for proprietary fixtures
│   └── KnxMonitor.ParserTool/            # CLI: `parse` / `detect` with --password / --keyring / --keyring-password
├── frontend/
│   └── src/app/
│       ├── core/                         # Singleton services (auth, signalr, project)
│       ├── features/                     # Feature modules (login, live-view, projects/import-wizard, settings)
│       └── shared/                       # Reusable components, models, layout
├── docs/
│   ├── samples/xknxproject/              # Public test fixtures (MIT, mirrored from XKNX/xknxproject)
│   ├── samples/own/                      # Private fixtures (gitignored)
│   ├── PARSER_LIBRARY_PLAN_PART1/2.md    # Historic library-extraction plan
│   ├── PARSER_LIBRARY_PROGRESS.md        # Implementation status of the parser library
│   ├── SAMPLE_TESTS.md                   # Manual / CLI test recipes per sample
│   └── ai/{PROJECT_PLAN,RELEASE_PLAN,DOKUMENTATION,TODO}.md
├── scripts/                              # build.sh / run-release.sh (+ .ps1 / .bat) — local prod-build mirror of CI
├── test-all-samples.sh / .ps1            # CLI smoke-test against every sample (skips own/ when missing)
├── Dockerfile                            # Multi-stage build (Node → SDK → debian:12-slim, ~120 MB)
└── docker-compose.yml
```

## Architecture Principles

### Backend (Clean Architecture)
- **Core** — Entities, Interfaces, DTOs (no dependencies)
- **ProjectParser** — Pure library, no EF / no Infrastructure dependency. Reusable as NuGet.
- **Infrastructure** — EF Core, repositories, KNX bus, adapter from `IKnxProjectParserService` → library `IProjectParser`. Holds `ProjectImportService` (job state machine, two-stage wizard, auto-activate).
- **Api** — Controllers, SignalR Hubs, JWT middleware, Program.cs DI registration.
- **Patterns** — Repository, Dependency Injection, async/await throughout.

### Frontend
- Standalone components (no NgModules), Angular Material + AG-Grid
- Reactive programming with RxJS
- Smart/dumb component split, route guards, JWT HTTP interceptor, lazy loading per feature

## Development Commands

### Backend (.NET)
```bash
cd backend
dotnet restore
dotnet build
dotnet run --project KnxMonitor.Api                                    # http://localhost:8080
# API-Referenz (nur Development): http://localhost:8080/scalar/v1 — Token oben rechts einsetzen,
# dann lassen sich die geschützten Endpunkte direkt ausprobieren. Rohes Dokument: /openapi/v1.json.
# Beim Build wird es zusätzlich nach docs/api/openapi.json geschrieben (versioniert, nie von Hand pflegen).

# Tests — parser library carries the bulk of the coverage
dotnet test KnxMonitor.ProjectParser.Tests/KnxMonitor.ProjectParser.Tests.csproj
dotnet test KnxMonitor.Infrastructure.Tests/KnxMonitor.Infrastructure.Tests.csproj

# Coverage (cobertura → coverage-tmp/, ignored by git)
dotnet test KnxMonitor.ProjectParser.Tests/KnxMonitor.ProjectParser.Tests.csproj \
  --collect:"XPlat Code Coverage" --results-directory ../coverage-tmp

# EF migrations (run from repo root)
dotnet ef migrations add <Name> --project backend/KnxMonitor.Infrastructure --startup-project backend/KnxMonitor.Api
dotnet ef database update      --project backend/KnxMonitor.Infrastructure --startup-project backend/KnxMonitor.Api
```

### Frontend (Angular)
```bash
cd frontend
npm install

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingel81/knx-ng-monitor](https://github.com/ingel81/knx-ng-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
