---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

gin-scaffold is a production-ready Go web service scaffold built on Gin framework with enterprise-grade features including database support (PostgreSQL, MySQL, SQLite), Redis caching, Kafka messaging, SSE streaming, i18n, and AI provider integrations (OpenAI, Gemini).

## Development Commands

### Running the Application

```bash
# Local development
go run main.go

# Production mode
ENV=prod go run main.go
```

### Testing

```bash
# Run all tests
go test ./...

# Run specific test
go test ./internal/utils/vo/vo_utils_test.go

# Run tests with verbose output
go test -v ./...
```

### Code Quality

```bash
# Format code
gofmt -w .

# Auto-import management
goimports -w .

# Static analysis
go vet ./...

# Generate Wire dependency injection code
cd pkg/wire && wire
```

## Architecture Overview

### Dependency Injection with Wire

The application uses Google Wire for compile-time dependency injection. All dependencies are wired together in `pkg/wire/`:
- `wire.go` defines the dependency graph using `//go:build wireinject`
- `wire_gen.go` is auto-generated (never edit manually)
- `providers.go` contains all provider functions
- The `Container` struct holds all initialized dependencies (managers, controllers, services)

To add new dependencies:
1. Add provider function to `providers.go`
2. Add to `AllProviders` wire set
3. Add field to `Container` struct in `wire.go`
4. Run `wire` to regenerate `wire_gen.go`

### Application Lifecycle

The application follows this initialization sequence (see `cmd/gin_server.go:25`):
1. Load configuration from `configs/config.{local|prod}.yml` based on ENV
2. Wire all dependencies via `wire.NewContainer()`
3. Initialize managers in order: Logger → Database → Redis
4. Setup Gin engine with middleware and routes
5. Start HTTP server with graceful shutdown support

All managers implement `Initialize()` and `Close()` methods and are deferred for proper cleanup.

### Module Organization

The codebase follows a domain-driven structure:

**Infrastructure Layer** (`internal/`):
- `db/` - Database management with GORM (auto-migration, multi-DB support)
- `redis/` - Redis client with connection pooling
- `logger/` - Logrus-based structured logging with rotation
- `queue/` - Kafka producer/consumer
- `sse/` - Server-Sent Events manager
- `i18n/` - Internationalization with go-i18n
- `ai/` - Multi-provider AI service (OpenAI/Gemini) with rate limiting and prompt management

**Domain Layer** (`internal/`):
- `model/` - GORM models with base model pattern
- `types/` - Domain types, constants, error codes
- `utils/` - Shared utilities (errorx, validator, snowflake, etc.)

**Application Layer** (`pkg/`):
- `serve/controller/` - HTTP controllers with DTO validation
- `serve/service/` - Business logic layer
- `serve/mapper/` - Data access layer (optional, use GORM directly if simple)
- `vo/` - View objects for API responses
- `router/` - Route registration organized by modules

### Three-Layer Architecture Pattern

Follow this structure when adding new features:
```
pkg/serve/
├── controller/
│   └── {module}/
│       ├── dto/           # Request DTOs with validation tags
│       └── {module}.go    # HTTP handlers
├── service/
│   └── {module}/
│       ├── {module}.go    # Service interface
│       └── impl/
│           └── {module}.go # Service implementation
└── mapper/
    └── {module}/
        ├── {module}.go    # Mapper interface
        └── impl/
            └── {module}.go # Data access implementation
```

### Configuration Management

Configuration uses Viper with hot-reload support:
- `configs/config.local.yml` - Local development
- `configs/config.prod.yml` - Production
- Config changes are automatically detected and logged
- Access config via `configs.GetConfig()` with RWMutex protection
- Use `UpdateField()` to programmatically modify config

### Error Handling System

The custom error system (`internal/utils/errorx/`) provides:
- Structured errors with status codes and i18n messages
- Stack trace capture for debugging
- Template parameter support for dynamic messages
- Error code registry with range allocation (see `internal/types/errno/README.md`)

Register error codes in `internal/types/errno/`:
```go
errorx.Register(10001, "user not found")
```

Create errors:
```go
return errorx.New(10001)
return errorx.New(10002, errorx.KV("field", "email"))
```

Error code ranges:
- 10000-19999: System errors (next: 10009)

### Middleware Stack

Middleware is registered in `internal/middleware/middleware.go:13`:
1. RequestID - Adds unique request ID
2. Logger - Request/response logging
3. Recovery - Panic recovery
4. CORS - Cross-origin support
5. Custom middleware as needed

### Route Organization

Routes are organized by module in `pkg/router/routes/`:
- Each module has its own routes file (e.g., `test_routes.go`)
- Routes are registered by API version (`/api/v1`, `/api/v2`)
- Registration function pattern: `RegisterXxxRoutes(container, v1, v2)`

## Coding Standards

This project follows strict Go coding conventions documented in `docs/coding-standards.en-US.md`. Key points:

### Naming Conventions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Done-0/gin-scaffold](https://github.com/Done-0/gin-scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
