---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Build and Run
- `make build` - Build the API server binary
- `make build-worker` - Build the worker server binary
- `make build-migration` - Build the migration server binary
- `make build-grpc` - Build the gRPC server binary
- `make build-all` - Build all servers
- `make run` - Run the API server directly
- `make run-worker` - Run the worker server directly
- `make run-migration` - Run database migrations
- `make run-seed` - Seed database with initial data
- `make run-drop` - Drop all database tables
- `make run-grpc` - Run the gRPC server
- `go run .` - Alternative way to run API server
- `go run ./cmd/worker` - Alternative way to run worker server

### Testing
- `make test` - Run all tests
- `make test-coverage` - Run tests with coverage report
- `make test-unit` - Run unit tests only
- `make test-integration` - Run integration tests only
- `make test-repo` - Run repository layer tests
- `make test-service` - Run service layer tests
- `make test-handler` - Run handler layer tests
- `make test-worker` - Run worker layer tests
- `make test-user` - Run user domain tests
- `make test-payment` - Run payment domain tests
- `make test-verbose` - Run tests with verbose output

### Code Quality & Linting
- `make lint` - Run golangci-lint (includes nil detection)
- `make lint-fix` - Run golangci-lint with auto-fix
- `make lint-verbose` - Run golangci-lint with verbose output
- `make lint-new` - Lint only new/changed files
- `make lint-linter LINTER=name` - Run specific linter
- `make lint-nil-info` - Show enabled nil detection linters
- `make format` - Format code with go fmt
- `make format-strict` - Format with stricter rules (gofumpt + goimports)

### Dependencies
- `make deps` - Download and tidy dependencies
- `go mod tidy` - Tidy go modules

### Development
- `make dev-setup` - Setup development environment
- `make tools` - Install development tools
- `make quality` - Run comprehensive quality checks
- `make pre-commit` - Run pre-commit checks
- `make install-hooks` - Install pre-commit hooks
- `make ci` - Run CI checks (linting, tests, build)
- `make clean` - Clean build artifacts

### Proto Generation
- `make proto-gen` - Generate gRPC code from proto files
- `make proto-clean` - Clean generated proto files
- `make proto-tools` - Install proto generation tools

### Swagger/OpenAPI Documentation
- `make swagger-gen` - Generate Swagger/OpenAPI documentation
- `make swagger-clean` - Clean generated swagger files
- `make swagger-tools` - Install swagger generation tools

### Docker
- `make docker-build` - Build Docker image
- `make docker-run` - Run Docker container

## Architecture

This is a Go web application following **Domain-Driven Design (DDD)** principles with **NestJS-like architecture patterns**. Built with modern Go practices, microservice architecture, and comprehensive background job processing.

### Multi-Server Architecture

The application follows a **multi-server architecture** where different concerns are separated into independent, deployable servers:

| Server | Purpose | Entry Point | Default Port |
|--------|---------|-------------|--------------|
| **API Server** | HTTP REST API | `main.go` | 8080 |
| **Worker Server** | Background job processing | `cmd/worker/main.go` | - |
| **Migration Server** | Database operations | `cmd/migration/main.go` | - |
| **gRPC Server** | gRPC services (User & Payment) | `cmd/grpc/main.go` | 9090 |

### Directory Structure
```
vibe-ddd-golang/
├── cmd/                                  # Application entry points
│   ├── api/main.go                       # API server startup
│   ├── worker/main.go                    # Worker server startup
│   ├── migration/main.go                 # Database migration server
│   └── grpc/main.go                      # gRPC server startup
├── internal/                             # Private application code
│   ├── application/                      # Domain layer (DDD)
│   │   ├── payment/                      # Payment domain
│   │   │   ├── dto/                      # Data Transfer Objects
│   │   │   ├── entity/                   # Domain entities
│   │   │   ├── repository/               # Data access layer
│   │   │   ├── service/                  # Business logic layer
│   │   │   ├── handler/                  # HTTP layer
│   │   │   ├── worker/                   # Background processing
│   │   │   └── module.go                 # Domain DI configuration
│   │   └── user/                         # User domain
│   │       ├── dto/user.dto.go           # User DTOs
│   │       ├── entity/user.entity.go     # User entity
│   │       ├── repository/user.repo.go   # User repository
│   │       ├── service/user.service.go   # User services
│   │       ├── handler/user.handler.go   # User endpoints
│   │       └── module.go                 # User DI config
│   ├── server/                           # Server implementations
│   │   ├── api/                          # HTTP API server
│   │   ├── worker/                       # Background worker server
│   │   ├── migration/                    # Database migration server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rizkydwicmt/vibe-ddd-golang](https://github.com/rizkydwicmt/vibe-ddd-golang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
