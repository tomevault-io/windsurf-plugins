---
trigger: always_on
description: The Sandbox Application is a monorepo containing an a .NET backend and a Angular frontend. It has a comprehensive testing strategy including unit, integration, architecture, and end-to-end tests.
---

# Critical System Understanding: Sandbox Application

The Sandbox Application is a monorepo containing an a .NET backend and a Angular frontend. It has a comprehensive testing strategy including unit, integration, architecture, and end-to-end tests.

## Architecture Foundation

**Monorepo**: All backend and frontend code lives in a single repository, enabling easier cross-team collaboration, consistent tooling, and simplified dependency management.

**Modular Monolith** + DDD: Clear bounded contexts. Each module implements IModule with separate schemas, DbContexts, and caching strategies. Always use TUnit for testing.

**BFF Pattern**: Gateway uses YARP to proxy requests, managing authentication server-side. Frontend never touches tokens—only HttpOnly cookies. The gateway automatically transforms cookies to Bearer tokens for API calls.

**Angular Frontend**: Modern Angular 21 app using standalone components, signals, and signal forms. Always Vitest for testing.

**Aspire Orchestration**: Aspire hosts the modular monolith, managing service lifecycles, configurations, and dependencies.

**Observability**: OpenTelemetry collector for metrics, traces, and logs across backend and frontend. Grafana (with loki, tempo, blackbox, prometheus) dashboards for visualization.

### System Architecture

The application follows a modular monolith architecture hosted by .NET Aspire with a BFF (Backend for Frontend) pattern for secure authentication:

```mermaid
graph TB
    subgraph "External Services"
        Keycloak[Keycloak<br/>Identity Provider]
    end

    subgraph "Aspire Orchestration"
        AppHost[Sandbox.AppHost<br/>Aspire Host]

        subgraph "Frontend Layer"
            Angular[Angular App<br/>Standalone Components<br/>Signals & Signal Forms]
        end

        subgraph "Gateway Layer - BFF Pattern"
            Gateway[Sandbox.Gateway<br/>YARP Reverse Proxy<br/>Cookie → Bearer Token]
        end

        subgraph "API Layer"
            ApiService[Sandbox.ApiService<br/>Modular Monolith]

            subgraph "Domain Modules"
                CustomerMgmt[CustomerManagement<br/>Module]
                Billing[Billing<br/>Module]
            end
        end

        subgraph "Data Layer"
            Migrations[Sandbox.Migrations<br/>EF Core Migrations]
            Postgres[(PostgreSQL<br/>Modular Schemas)]
            Redis[(Redis<br/>Distributed Cache<br/>FusionCache L2)]
        end

        subgraph "Observability"
            OTel[OpenTelemetry<br/>Collector]
            Grafana[Grafana Stack<br/>Loki, Tempo<br/>Prometheus]
        end
    end

    subgraph "Shared Libraries"
        SharedKernel[SharedKernel<br/>Vogen Value Objects<br/>Messages & Utilities]
        ServiceDefaults[ServiceDefaults<br/>Common Configs]
    end

    User([User]) -->|HTTPS| Angular
    Angular -->|HttpOnly Cookies| Gateway
    Gateway -->|OIDC/OAuth2| Keycloak
    Gateway -->|Bearer Token| ApiService

    ApiService --> CustomerMgmt
    ApiService --> Billing
    CustomerMgmt --> Postgres
    Billing --> Postgres
    CustomerMgmt -.->|L1/L2 Cache| Redis
    Billing -.->|L1/L2 Cache| Redis

    Migrations -->|Apply Migrations| Postgres

    ApiService -.->|Traces/Metrics/Logs| OTel
    Gateway -.->|Traces/Metrics/Logs| OTel
    Angular -.->|Traces/Metrics/Logs| OTel
    OTel --> Grafana

    AppHost -.->|Orchestrates| Gateway
    AppHost -.->|Orchestrates| ApiService
    AppHost -.->|Orchestrates| Angular
    AppHost -.->|Orchestrates| Migrations
    AppHost -.->|Orchestrates| Postgres
    AppHost -.->|Orchestrates| Redis
    AppHost -.->|Orchestrates| OTel

    CustomerMgmt -.->|Uses| SharedKernel
    Billing -.->|Uses| SharedKernel
    ApiService -.->|Uses| ServiceDefaults
    Gateway -.->|Uses| ServiceDefaults

    style User fill:#e1f5ff
    style Keycloak fill:#fff4e1
    style Angular fill:#dd0031,color:#fff
    style Gateway fill:#512bd4,color:#fff
    style ApiService fill:#512bd4,color:#fff
    style CustomerMgmt fill:#68217a,color:#fff
    style Billing fill:#68217a,color:#fff
    style Postgres fill:#336791,color:#fff
    style Redis fill:#dc382d,color:#fff
    style OTel fill:#f5a800
    style Grafana fill:#f46800,color:#fff
```

**Key Architectural Decisions:**

- **Aspire Orchestration**: Single command (`dotnet run --project ./Sandbox.AppHost`) starts entire stack
- **BFF Pattern**: Gateway handles authentication, transforms cookies to Bearer tokens
- **Modular Monolith**: Each module has its own DbContext, schema, and bounded context
- **Security-First**: Tokens never exposed to frontend, only secure HttpOnly cookies
- **Observability-Ready**: OpenTelemetry integrated at all layers

### Request Flow

The following diagram illustrates how a typical authenticated API request flows through the system using the BFF pattern:

```mermaid
sequenceDiagram
    actor User
    participant Browser as Angular App<br/>(Browser)
    participant Gateway as Sandbox.Gateway<br/>(YARP BFF)
    participant Keycloak as Keycloak<br/>(Identity Provider)
    participant API as Sandbox.ApiService<br/>(Modular Monolith)
    participant Module as Domain Module<br/>(e.g., CustomerManagement)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timdeschryver/Sandbox](https://github.com/timdeschryver/Sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
