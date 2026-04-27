---
trigger: always_on
description: A Docker-first, event-driven enterprise demo application focused around a generic "model," deployed to Azure via Terraform (and respective CLIs). The system demonstrates enterprise patterns: SSO, async job processing, observability, and repeatable IaC deployments.
---

# CLAUDE.md — Enterprise App

## Project Overview

A Docker-first, event-driven enterprise demo application focused around a generic "model," deployed to Azure via Terraform (and respective CLIs). The system demonstrates enterprise patterns: SSO, async job processing, observability, and repeatable IaC deployments.

The development occurs within a VS Code-based Devcontainer, as defined in `.devcontainer`. Any missing CLIs, dependencies, should be updated accordingly in setup script `.devcontainer/scripts/setup-env.sh`.

### Architecture

- **Angular 20 SPA** (`ui/`) — client UI, hosted on Azure Static Web Apps
- **ASP.NET Core .NET 10 REST API** (`api/`) — system-of-record, EF Core + Postgres, publishes commands to RabbitMQ
- **Python 3 Data Engine** (`data-engine`) - companion service for numerical computations and data-related workflows, jobs, etc.; transmits data via RabbitMQ
- **RabbitMQ** — message broker for async job workflows
- **PostgreSQL** — relational data store, managed via EF Core migrations
- **Azure Container Apps** — runtime for API and RabbitMQ containers
- **Terraform** (`infra/`) — all Azure infrastructure as code

### Interaction Flow

```
User → Angular SPA → (Bearer token) → ASP.NET Core API → PostgreSQL
                                            ↓ publish
                                        RabbitMQ
                                            ↓ consume
                                    (future workers)
```

## Project Repository Structure

```
/workspace
├── api/                        # ASP.NET Core .NET 10 REST API
│   ├── src/
│   │   ├── EA.Api/             # Web API project (controllers, endpoints)
│   │   ├── EA.Domain/          # Domain models, interfaces, value objects
│   │   ├── EA.Infrastructure/  # EF Core DbContext, migrations, RabbitMQ integration
│   │   └── EA.Contracts/       # Shared DTOs, message contracts, JSON schemas
│   ├── tests/
│   │   ├── EA.Api.Tests/            # Unit tests
│   │   └── EA.Api.IntegrationTests/ # Testcontainers-based integration tests
│   ├── Dockerfile              # Multi-stage: SDK build → aspnet runtime
│   ├── Dockerfile.migrations   # EF Core migration bundle image
│   └── EA.sln
├── ui/                         # Angular 20 SPA
│   ├── src/
│   ├── Dockerfile              # Multi-stage: node build → nginx (local parity only)
│   ├── angular.json
│   └── package.json
├── infra/                      # Terraform (Azure)
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── backend.tf
│   └── modules/
│       ├── container-apps/
│       ├── postgres/
│       ├── static-web-app/
│       ├── container-registry/
│       ├── key-vault/
│       └── observability/
├── deploy/
│   ├── compose.yaml            # Full-stack local dev (API + Postgres + RabbitMQ + UI)
│   └── compose.override.yaml   # Dev overrides (hot reload, debug ports)
├── .github/
│   └── workflows/
│       ├── ci.yml              # Build, test, push images
│       ├── deploy.yml          # Terraform plan/apply
│       └── swa-deploy.yml      # Static Web Apps deploy
├── schemas/                    # JSON Schema message contracts (source of truth)
├── docs/                       # Architecture decisions, runbooks, API docs
└── CLAUDE.md                   # This file
```

## Technology Stack & Versions

| Layer | Technology | Version | Notes |
|---|---|---|---|
| API runtime | .NET | 10 | `mcr.microsoft.com/dotnet/sdk:10.0` / `aspnet:10.0` |
| API framework | ASP.NET Core | 10 | Minimal APIs preferred; controllers acceptable |
| ORM | EF Core + Npgsql | latest stable | Npgsql.EntityFrameworkCore.PostgreSQL |
| Messaging (.NET) | MassTransit | latest stable | RabbitMQ transport, outbox, sagas |
| Frontend | Angular | 20 | Standalone components, signals preferred |
| Frontend auth | MSAL Angular | latest | Auth code flow + PKCE via Entra ID |
| Database | PostgreSQL | 16 | Azure Flexible Server in cloud; `postgres:16` locally |
| Message broker | RabbitMQ | 4 | `rabbitmq:4-management` image |
| IaC | Terraform | ≥1.9 | AzureRM provider, azurerm backend |
| Containers | Docker | Compose v2 | Multi-stage builds, BuildKit |
| CI/CD | GitHub Actions | — | OIDC to Azure, Buildx, matrix builds |
| Observability | OpenTelemetry | — | Azure Monitor distro for .NET |

## Development Workflow

### Local Development (Docker-first)

```bash
# Start full stack
docker compose -f deploy/compose.yaml up --build

# API:        http://localhost:8000
# UI:         http://localhost:4200
# RabbitMQ:   http://localhost:15672 (guest/guest)
# Postgres:   localhost:5432 (demo/demo/demo)
```

### Running Tests

```bash
# API unit tests
dotnet test api/tests/EA.Api.Tests/

# API integration tests (requires Docker for Testcontainers)
dotnet test api/tests/EA.Api.IntegrationTests/

# UI tests
cd ui && npm test
```

### Database Migrations

```bash
# Add a migration
cd api/src/EA.Infrastructure
dotnet ef migrations add <MigrationName> -s ../EA.Api

# Apply locally
dotnet ef database update -s ../EA.Api

# Generate idempotent SQL script (for CI/CD)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/awpala) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-12 -->
