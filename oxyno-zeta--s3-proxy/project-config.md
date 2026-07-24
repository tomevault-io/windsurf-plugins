---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Instructions for CLAUDE

- If you see something missing in following sections, add it
- If you see that something have been deleted in project but not in this file, inform and propose a deletion
- If you see that something needs to be updated, update it
- When you are changing, adding or deleting code, run `make code/lint` to ensure linter is passing

## Commands

All commands are run from the project root.

### Build & Lint

```bash
make code/build          # Build binary to bin/s3-proxy (current OS)
make code/build-cross    # Cross-platform build via goreleaser
make code/lint           # Run golangci-lint (default goal)
make code/clean          # Remove build artifacts
make code/generate       # Run go generate ./...
make code/fieldalignment # Fix struct field alignment
```

### Tests

```bash
make test/all            # Run all tests (unit + integration) with coverage + JUnit XML
make test/unit           # Run unit tests only
make test/integration    # Run integration tests only (requires services)
make test/coverage       # Generate HTML coverage report
```

Run a single test:

```bash
go test -tags=unit ./pkg/... -run TestFunctionName
go test -tags=integration ./pkg/... -run TestFunctionName
```

Run a single test inside a testify suite:

```bash
# Pattern: TestSuiteName/TestMethodName
go test -tags=unit ./pkg/... -run TestMySuite/TestFunctionName
go test -tags=integration ./pkg/... -run TestMySuite/TestFunctionName
```

### Services (integration tests)

```bash
make setup/services               # Start OPA + Keycloak (required for integration tests)
make setup/tracing-services       # Start Jaeger
make setup/metrics-services       # Start Prometheus + Grafana
make setup/oauth2-proxy-services  # Start OAuth2-Proxy
make down/services                # Stop all services
```

### Documentation

```bash
make code/docs           # Serve MkDocs documentation locally via Docker
make code/build/docs     # Build static documentation
```

## Technology Stack

### Language & Runtime

|          |                                  |
| -------- | -------------------------------- |
| Language | Go 1.26                          |
| Module   | `github.com/oxyno-zeta/s3-proxy` |

### HTTP & Routing

| Library                    | Role             |
| -------------------------- | ---------------- |
| `github.com/go-chi/chi/v5` | HTTP router      |
| `net/http` (stdlib)        | TLS/HTTPS server |

### AWS

| Library                                          | Role                                                        |
| ------------------------------------------------ | ----------------------------------------------------------- |
| `github.com/aws/aws-sdk-go`                      | S3 client (GetObject, PutObject, DeleteObject, ListObjects) |
| `github.com/aws/aws-sdk-go/service/s3/s3manager` | Multipart upload/download                                   |

### Authentication & Authorization

| Library                        | Role                                  |
| ------------------------------ | ------------------------------------- |
| `github.com/coreos/go-oidc/v3` | OIDC token validation                 |
| `golang.org/x/oauth2`          | OAuth 2.0 / OIDC flows                |
| OPA (Open Policy Agent)        | Policy enforcement (external service) |
| Basic Auth / Header Auth       | Built-in auth providers               |

### Observability

| Library                                 | Role                              |
| --------------------------------------- | --------------------------------- |
| `github.com/prometheus/client_golang`   | Prometheus metrics                |
| `github.com/opentracing/opentracing-go` | Distributed tracing (OpenTracing) |
| `github.com/uber/jaeger-client-go`      | Jaeger tracing backend            |

### Logging & Configuration

| Library                      | Role                        |
| ---------------------------- | --------------------------- |
| `github.com/sirupsen/logrus` | Structured logging          |
| `github.com/spf13/viper`     | YAML config with hot-reload |
| `github.com/spf13/cobra`     | CLI framework               |

### Templating & Utilities

| Library                                  | Role                     |
| ---------------------------------------- | ------------------------ |
| `github.com/Masterminds/sprig/v3`        | Sprig template functions |
| `github.com/go-playground/validator/v10` | Config/input validation  |
| `github.com/thoas/go-funk`               | Functional helpers       |

### Testing

| Library                               | Role                           |
| ------------------------------------- | ------------------------------ |
| `github.com/stretchr/testify`         | Assertions, mocks, suites      |
| `github.com/golang/mock`              | Mock generation (mockgen)      |
| `github.com/gotestyourself/gotestsum` | Test runner + JUnit XML output |

### Linting & Formatting

- **golangci-lint v2** — 40+ linters (`gosec`, `gocritic`, `errname`, `exhaustive`, `bodyclose`, `contextcheck`, …)
- **gofmt** / **goimports** — formatting and import ordering

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oxyno-zeta/s3-proxy](https://github.com/oxyno-zeta/s3-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
