---
trigger: always_on
description: Raiden is a Go framework for secure integration with Supabase, providing a unified backend layer for RPC, Edge Functions, and REST APIs. It prevents direct client-to-database calls by providing a secure Go-based server layer that manages all Supabase interactions.
---

# Raiden Framework - Copilot Instructions

## Overview
Raiden is a Go framework for secure integration with Supabase, providing a unified backend layer for RPC, Edge Functions, and REST APIs. It prevents direct client-to-database calls by providing a secure Go-based server layer that manages all Supabase interactions.

## Project Structure

### Module Organization
- **Root**: Go module root with core framework files (`server.go`, `router.go`, `controller.go`, `context.go`, etc.)
- **`cmd/raiden/`**: CLI application entry point and subcommands
  - `main.go`: CLI bootstrap
  - `commands/`: Subcommands (init, start, run, serve, generate, apply, configure, etc.)
- **`pkg/`**: Domain logic organized by capability
  - `connector`: Supabase connection management
  - `pubsub`: Pub/sub functionality
  - `resource`: Schema and resource definitions
  - `controllers`: Controller implementations
  - `mock`: Test mocks for Supabase/network dependencies
  - `logger`, `jwt`, `tracer`, `utils`, etc.
- **`deployments/`**: Operational/deployment assets
- **`docs/`**: Long-form documentation
- **`examples/`**: Runnable sample applications
- **`scripts/`**: Helper automation scripts
- **`build/`**: Build artifacts (created by Makefile)

### Test Organization
Tests live alongside source files with `_test.go` suffix throughout root and `pkg/`. Internal tests use `_internal_test.go` for white-box testing.

## Build, Test, and Development

### Build Commands
```bash
# Cross-compile CLI for Linux, Windows, macOS (amd64)
make build

# Build for arm64 (Linux + macOS only)
make build-arm64

# Run CLI locally without building
go run ./cmd/raiden

# Individual platform builds
GOOS=linux GOARCH=amd64 go build -o build/raiden_linux ./cmd/raiden/main.go
```

### Test Commands
```bash
# Run all tests with coverage
make test

# Run specific package tests (useful for iteration)
go test ./pkg/connector -v
go test ./pkg/... -v

# Run single test
go test ./pkg/connector -run TestConnectorName -v

# Run tests without coverage
go test ./...
```

### Linting
```bash
# Format code (required before commit)
go fmt ./...

# Run golangci-lint
golangci-lint run

# Lint configuration is in .golangci.yml
```

## Architecture & Key Concepts

### Request Flow
1. **HTTP Request** → `Server` (fasthttp-based)
2. **Router** (fasthttp/router) → Route matching based on `RouteType`
3. **Middleware** → Applied to request context
4. **Controller** → Implements HTTP method handlers (Get, Post, Put, Patch, Delete)
   - `BeforeAll` / `AfterAll`: Runs before/after any HTTP method
   - `Before{Method}` / `After{Method}`: Runs before/after specific method
5. **Context** → Request/response interface for controllers
6. **Response** → JSON/error sent back to client

### Route Types
- `RouteTypeCustom`: Custom handler functions
- `RouteTypeFunction`: Edge function routes
- `RouteTypeRest`: RESTful API endpoints (CRUD on models)
- `RouteTypeRpc`: RPC endpoints mapped to database functions
- `RouteTypeRealtime`: WebSocket/realtime endpoints
- `RouteTypeStorage`: Storage bucket operations

### Core Abstractions
- **`Server`**: Main server orchestrator; manages HTTP server, router, scheduler, pub/sub
- **`Router`**: Route registration and dispatching; organized by `RouteType`
- **`Context`**: Request context interface providing access to request/response, config, tracing, RPC execution, pub/sub
- **`Controller`**: Interface for handling HTTP methods with before/after hooks
- **`Config`**: Application configuration loaded from environment variables
- **`Model`**: Database model definitions used for REST endpoints
- **`Rpc`**: RPC function definitions mapped to Supabase functions
- **`Job`**: Scheduled job interface (cron-like)
- **`PubSub`**: Pub/sub interface for Google Pub/Sub integration
- **`Bucket`**: Storage bucket interface

### Supabase Integration
- `pkg/connector`: Manages Supabase client connections
- `pkg/resource`: Defines and manages Supabase resources (tables, functions, policies, roles)
- `pkg/supabase`: Supabase API client
- Models and RPCs are synced with Supabase schema via CLI commands (`generate`, `apply`, `imports`)

## Coding Conventions

### Naming
- **Exported APIs**: PascalCase (e.g., `NewServer`, `ExecuteRpc`)
- **Internal helpers**: lowerCamelCase (e.g., `createRouteGroups`, `handleRequest`)
- **Packages**: lower_snake_case matching directory (e.g., `pkg/pubsub`)
- **Constructors**: Prefix with `New` (e.g., `NewLogger`, `NewConnector`)

### File Organization
- Keep files focused by capability (e.g., `server.go` for server, `router.go` for routing)
- Colocate tests with source (`*_test.go` peers)
- Use `_internal_test.go` for white-box tests that need access to unexported symbols

### Logging
Use structured logging from `pkg/logger`:
```go
var MyLogger = logger.HcLog().Named("raiden.mycomponent")
MyLogger.Info("message", "key", value)
```

### Error Handling
Return errors explicitly; use `Context.SendError()` or `Context.SendErrorWithCode()` in controllers.

## Testing Guidelines

### Test Style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sev-2/raiden](https://github.com/sev-2/raiden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
