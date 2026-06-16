---
trigger: always_on
description: - @azure Rule - Use Azure Tools - When handling requests related to Azure, always use your tools.
---

- @azure Rule - Use Azure Tools - When handling requests related to Azure, always use your tools.
- @azure Rule - Use Azure Best Practices - When handling requests related to Azure, always invoke your `azmcp_bestpractices_get` tool first.
- @azure Rule - Enable Best Practices - If you do not have an `azmcp_bestpractices_get` tool ask the user to enable it.

# Project context

This repository is a **.NET e-commerce microservices monorepo**. It is **not** a multi-tenant SaaS monolith. Treat each microservice as an independent, deployable unit that follows the repo-default Clean Architecture + Vertical Slices shape ([ADR-0012](../docs/adr/0012-clean-arch-vsa-default-service-shape.md), [docs/PATTERNS.md](../docs/PATTERNS.md)) and shares cross-cutting concerns through narrow `ECommerce.Shared.*` capability packages.

For the authoritative architectural overview, read [CLAUDE.md](../CLAUDE.md) and [CONTEXT.md](../CONTEXT.md). Behavioral coding guidelines live in [.claude/CLAUDE.md](../.claude/CLAUDE.md).

## Repo shape

- Top-level `*-microservice/` folders, plus `api-gateway/` and `shared-libs/`.
- Each service is its own solution defined by a **`.slnx`** file. There is no root solution that builds everything.
- All projects target **`net10.0`** (see [Directory.Build.props](../Directory.Build.props)).
- `local-nuget-packages/` is a local NuGet feed used to consume shared-libs capability packages.
- Service code defaults to `Features/<Slice>/`, `Domain/`, `Contracts/Integration/`, and `Infrastructure/`; documented divergences live in each service's `CLAUDE.md`.

Services and ports (from [docker-compose.yaml](../docker-compose.yaml)):

| Service | Port | Datastore | Responsibility |
|---|---:|---|---|
| Basket | 8000 | Redis | Shopping cart CRUD and product price caching |
| Order | 8001 | SQL Server + Redis | Order creation, confirmation, cancellation, and order events |
| Product | 8002 | SQL Server | Product catalog and product price events |
| Auth | 8003 | SQL Server | User JWTs, service tokens, and JWKS discovery |
| API Gateway | 8004 | SQL Server | YARP/Ocelot routing, auth enforcement, combined Swagger UI, and DLQ operator API |
| Inventory | 8005 | SQL Server | Stock levels, reservations, backorders, and inventory reply events |
| Shipping | 8006 | SQL Server | Shipment lifecycle and shipment reply events |
| Payment | 8007 | SQL Server | Payment authorization, capture, void, refund, and payment reply events |
| Saga | 8008 | SQL Server | Owns order saga state; drives Order/Inventory/Payment/Shipping via commands |

## Tech stack (actual)

- **Language / runtime:** C# (latest), .NET 10
- **Web:** ASP.NET Core **Minimal APIs** (no MVC controllers, no MediatR)
- **Persistence:** EF Core with **SQL Server** (Redis only for basket and as order cache)
- **Messaging:** RabbitMQ by default or Azure Service Bus via `Messaging:Provider`, wrapped by `IEventBus` from `ECommerce.Shared.EventBus` and provider-aware composition from `ECommerce.Shared.Messaging`
- **Observability:** OpenTelemetry, Jaeger, Loki, Grafana, Prometheus (see `observability/` and `kubernetes/`)
- **Containerization:** Docker / Docker Compose; Kubernetes manifests under `kubernetes/`
- **CI/CD:** GitHub Actions runs build verification (`docker-build.yml`) and QA smoke checks (`smoke-test.yml`) as CI gates; Azure Pipelines (per-service `azure-pipelines.yml`) is the deployment path.
- **Cloud target:** Azure (AKS manifests prefixed `aks-*` in `kubernetes/`)

## Build, test, run

Operate **per service**. There is no root build.

```bash
# Build / test a service (run from its directory)
cd order-microservice && dotnet build
cd order-microservice && dotnet test
cd order-microservice && dotnet test --filter "FullyQualifiedName~OrderEndpointTests"
cd order-microservice && dotnet test --filter "DisplayName~Given_X_When_Y_Then_Z"

# Format (mirrors pre-commit)
dotnet format --verify-no-changes --verbosity minimal
dotnet format

# Full stack
docker compose up --build
docker compose up sql rabbitmq redis -d   # infra only
```

`Directory.Build.props` enables `TreatWarningsAsErrors` and `EnforceCodeStyleInBuild`. The documented `NoWarn` exemptions (`CA1707`, `CA1711`, `CA1716`, NuGet `NU*` warnings) are intentional — do not "fix" code to remove them.

## Pre-commit (Husky.Net)

`.husky/task-runner.json` runs on commit:

1. `dotnet format --verify-no-changes`
2. `dotnet build --no-restore`
3. `dotnet test basket-microservice/Basket.Service.slnx --no-build --no-restore`

Only Basket tests run pre-commit. Run other service test suites manually before pushing changes that cross service boundaries.

## Shared library workflow (`ECommerce.Shared`)

Shared-libs are consumed as **NuGet packages**, **not** `<ProjectReference>`s. Since [ADR-0013](../docs/adr/0013-shared-libs-multi-package-split.md), production services use direct capability packages (`ECommerce.Shared.Kernel`, `.EventBus`, `.RabbitMq`, `.AzureServiceBus`, `.Messaging`, `.DeadLetter`, `.Platform`, `.Contracts`, `.Testing.Qa`) and avoid the umbrella `ECommerce.Shared` package unless broad compatibility is intentional. Use [docs/runbooks/shared-libs-versioning.md](../docs/runbooks/shared-libs-versioning.md) for package selection and version sweeps.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daonhan/Microservices-in-.NET](https://github.com/daonhan/Microservices-in-.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
