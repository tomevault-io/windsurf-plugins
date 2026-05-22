---
trigger: always_on
description: Universal entry point for any AI coding agent working on the Reporter codebase.
---

# AGENTS.md — AI Agent Quick Reference

Universal entry point for any AI coding agent working on the Reporter codebase.

## Project Identity

**Reporter** is a template-driven report generation service built by Lerian Studio. It manages templates and generates reports in multiple formats (HTML, PDF, CSV, XML, TXT) by connecting to external databases, rendering Pongo2 templates, and storing results in S3-compatible object storage. Asynchronous processing via RabbitMQ enables scalable report generation.

- **Language**: Go 1.26 (module `go 1.26`)
- **Architecture**: Multi-component hexagonal (Manager + Worker) with shared `pkg/`
- **Metadata Store**: MongoDB 8
- **Message Queue**: RabbitMQ 4.0
- **Object Storage**: S3-compatible (SeaweedFS / MinIO / AWS S3)
- **Template Engine**: Pongo2 (Django/Jinja2-like)
- **PDF Generation**: Chromium headless via chromedp
- **License**: Elastic License 2.0

## Quick Start

```bash
make set-env          # Create .env files from .env.example
make up               # Start all services (infra + manager + worker)
make test-unit        # Run unit tests
make lint             # golangci-lint v2
```

Health: `GET http://localhost:4005/health`
Swagger: `http://localhost:4005/swagger/index.html`

## Architecture Overview

Two independently deployable components sharing a common `pkg/` package:

| Component | Role | Port |
|-----------|------|------|
| **Manager** | REST API: template/report/deadline/data-source CRUD, publishes to RabbitMQ | 4005 |
| **Worker** | RabbitMQ consumer: generates reports, renders templates, generates PDFs, stores files | 4006 (health) |

### Bounded Contexts

| Context | Role |
|---------|------|
| **Templates** | Manage template lifecycle (CRUD), store files in S3, validate mapped fields |
| **Reports** | Manage report lifecycle, dispatch to RabbitMQ, track status |
| **Report Generation** | (Worker) Query data, render templates, generate PDFs, store output |
| **Data Sources** | Introspect external database schemas, manage connections |
| **Deadlines** | Schedule recurring report generation with notifications |

### Layer Structure

```
components/
├── manager/
│   ├── internal/
│   │   ├── bootstrap/     # Composition root: config, DI, server, tenant middleware
│   │   ├── adapters/
│   │   │   ├── http/in/   # Fiber handlers, routes, middleware, rate limiting
│   │   │   ├── rabbitmq/  # RabbitMQ producer adapter
│   │   │   └── redis/     # Redis adapter
│   │   └── services/      # Business logic (template/report/deadline/datasource)
│   └── Dockerfile         # golang:1.26-alpine → alpine:3.23 (no Chromium)
├── worker/
│   ├── internal/
│   │   ├── bootstrap/     # Composition root: config, consumer, health server
│   │   ├── adapters/      # RabbitMQ consumer, Redis
│   │   └── services/      # Report generation pipeline, Fetcher integration
│   └── Dockerfile         # golang:1.26-alpine → alpine:3.22 (with Chromium)
└── infra/                 # Docker Compose: MongoDB, RabbitMQ, SeaweedFS, Valkey, KEDA
pkg/
├── model/                 # Domain entities (Report, Template, FilterCondition)
├── constant/              # Error codes (TPL-XXXX), status constants
├── mongodb/               # MongoDB repositories (template, report, deadline)
├── storage/               # S3-compatible object storage adapter
├── pongo/                 # Pongo2 template engine extensions
├── pdf/                   # Chrome headless PDF generation pool
├── datasource/            # DataSourceProvider interface
├── circuit-breaker.go     # External datasource resilience (sony/gobreaker)
└── health-checker.go      # Background monitoring of data source availability
```

## Essential Commands

| Command | Purpose |
|---------|---------|
| `make build` | Build manager and worker binaries |
| `make run` | Run manager locally |
| `make test-unit` | Unit tests with race detector |
| `make test-integration` | Integration tests (testcontainers) |
| `make test-e2e` | E2E full-stack tests |
| `make test-fuzzy` | Fuzz/robustness tests |
| `make test-chaos` | Chaos/resilience tests (Toxiproxy) |
| `make test-property` | Property-based tests |
| `make lint` | golangci-lint v2 with auto-fix |
| `make sec` | gosec + govulncheck |
| `make generate` | go generate (mocks) |
| `make generate-docs` | Regenerate Swagger docs |
| `make up` / `make down` | Docker Compose lifecycle |

## Code Conventions

### Entity Constructors
```go
func NewReportMessage(templateID, reportID uuid.UUID, outputFormat string, ...) (*ReportMessage, error)
```
Validate all invariants. Return `(*T, error)`. Never panic.

### Service Methods
```go
func (uc *UseCase) CreateTemplate(ctx context.Context, input *CreateTemplateInput) (*TemplateOutput, error) {
    logger, tracer, _, _ := libCommons.NewTrackingFromContext(ctx)
    ctx, span := tracer.Start(ctx, "service.template.create")
    defer span.End()
    logger = logging.WithTrace(ctx, logger)
    // ...
}
```
Always start with tracking + span. Enrich logger with trace context.

### Repository Pattern
- Interfaces defined in `pkg/mongodb/*` and `pkg/storage/*`
- MongoDB repositories use `go.mongodb.org/mongo-driver`
- S3 storage via `aws-sdk-go-v2/service/s3`
- Cursor-based pagination (no offset)

### Error Handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LerianStudio/reporter](https://github.com/LerianStudio/reporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
