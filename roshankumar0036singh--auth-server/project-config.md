---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Auth Server** is a production-ready authentication microservice written in **Go** using **Gin Gonic** framework. It implements OAuth 2.0 Provider, JWT-based authentication, MFA (TOTP), and RBAC.

### Tech Stack

- **Language**: Go 1.25+
- **Framework**: Gin Gonic
- **Database**: PostgreSQL with GORM ORM
- **Cache**: Redis (rate limiting, token blacklist, sessions)
- **Auth**: OAuth 2.0, JWT, TOTP, Social Login (Google/GitHub)

## Architecture

Clean Architecture with distinct layers:

```
cmd/server/main.go              # Entry point with Gin setup & GORM auto-migration
├── internal/config/            # Configuration loading & DB/Redis initialization
├── internal/routes/            # Route definitions, middleware registration
├── internal/handler/           # HTTP request handlers (API endpoints)
├── internal/service/           # Business logic (auth, OAuth, tokens, MFA, email)
├── internal/repository/        # Data access layer (GORM queries)
├── internal/models/            # GORM models (User, RefreshToken, OAuthClient, etc)
├── internal/middleware/        # Auth, CORS, security headers, rate limiting
├── internal/dto/               # Request/response DTOs for HTTP handlers
├── internal/utils/             # Helpers (validation, error handling, JWT claims)
└── internal/testutils/         # Test helpers & mocks
```

### Key Design Patterns

- **Dependency Injection**: Services receive dependencies in constructors. Database and Redis clients passed through layers.
- **Repository Pattern**: All database queries isolated in `repository/` package. Services call repos, don't touch GORM directly.
- **Error Handling**: Custom error types in `service/errors.go`. Handlers convert service errors to HTTP status codes.
- **Middleware Stack**: Security headers (CORS, CSP), JWT validation, rate limiting, and recovery middleware registered via `routes.SetupRoutes()`.

## Common Commands

```bash
# Install dev tools (swag, migrate)
make install-tools

# Run locally (hot reload not configured; use `go run cmd/server/main.go`)
make run

# Build for development
make build

# Build for production (statically linked)
make build-prod

# Run all tests
make test

# Run tests with coverage report (generates coverage.html)
make test-coverage

# Run a single test
go test ./internal/service -run TestTokenService_GenerateAccessToken -v

# Lint code (requires golangci-lint installed)
make lint

# Format code
make fmt

# Tidy dependencies
make tidy

# Generate Swagger docs from code comments
make swagger

# Docker: start postgres + redis
make docker-up

# Docker: stop services
make docker-down

# Clean build artifacts
make clean
```

## Database

- **Migration Strategy**: No migration CLI files. Uses GORM auto-migration in `main()`:

  ```go
  config.AutoMigrate(db, &models.User{}, &models.RefreshToken{}, ...)
  ```

  Models sync schema on startup. For production schema changes, add new model fields and restart.

- **Models**: Located in `internal/models/`. Define database schema via struct tags. Examples:
  - `models.User` — user accounts with email, password hash, role
  - `models.RefreshToken` — token revocation tracking
  - `models.OAuthClient` — OAuth 2.0 third-party app registration
  - `models.AuditLog` — security event tracking

## Environment Setup

1. Copy `.env.example` to `.env` and fill in values:

   ```bash
   cp .env.example .env
   ```

2. Required env vars:
   - `APP_ENV`: `development` or `production`
   - `DATABASE_URL`: PostgreSQL connection string
   - `REDIS_URL`: Redis connection string
   - `JWT_SECRET`, `JWT_REFRESH_SECRET`: Token signing keys
   - `SMTP_*`: Email service credentials (Gmail, SendGrid, etc)
   - OAuth client IDs/secrets for Google and GitHub (optional; can be set via admin API)

3. Start services:
   ```bash
   docker compose up -d  # Starts PostgreSQL + Redis
   go run cmd/server/main.go
   ```
   Server runs on `:8080` by default.

## Testing Patterns

- **Unit Tests**: Service & repository tests use mocks. See `token_service_test.go`.
- **Integration Tests**: Full DB setup with test fixtures. See `auth_service_integration_test.go`.
- **Test Naming**: `Test<PackageName>_<FunctionName>` or `Test<Scenario>`.
- **Assertions**: Uses `testify/assert` for readable assertions.
- **Test Utils**: `internal/testutils/` provides helpers (DB setup, fixtures, mock clients).

### Run Tests

```bash
# Run all tests
go test ./...

# Run tests in a single package with verbose output
go test ./internal/service -v

# Run one test
go test ./internal/service -run TestTokenService_GenerateAccessToken -v

# Coverage
go test ./... -cover

# HTML coverage report
go test ./... -coverprofile=coverage.out && go tool cover -html=coverage.out
```

## Key Code Areas

### Authentication Flow (`internal/service/auth_service.go`)

- User registration: hash password with bcrypt, create user, send verification email
- Login: validate credentials, return access + refresh tokens
- Token refresh: validate refresh token, issue new access token
- Logout: revoke tokens in Redis blacklist


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roshankumar0036singh/auth-server](https://github.com/roshankumar0036singh/auth-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
