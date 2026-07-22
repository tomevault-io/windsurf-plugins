---
trigger: always_on
description: Instructions for AI coding agents working in this repository.
---

# AGENTS.md

Instructions for AI coding agents working in this repository.

## Project Overview

Go RESTful API starter kit built with Gin, GORM, JWT, Redis, MongoDB,
2FA, email verification, and password recovery. Module path:
`github.com/pilinux/gorest`. Requires Go 1.25.0+ (`go.mod` declares
`go 1.25.0`; CI tests against Go 1.25.x and 1.26.x).

## Build and Run Commands

### Build

```bash
go build -v ./...
```

### Tidy Dependencies

```bash
go mod tidy
```

### Format

```bash
go fmt ./...
```

### Vet

```bash
go vet -v ./...
```

### Lint

```bash
golangci-lint run ./...
revive ./...
```

CI uses golangci-lint v2.12.2 with `--timeout 5m --verbose`.

### Test - All

```bash
go test -v -cover ./...
```

### Test - Single Package

```bash
go test -v -cover ./lib/...
go test -v -cover ./config/...
```

### Test - Single Test Function

```bash
go test -v -run TestHashPass ./lib/...
go test -v -run TestFirewall ./lib/middleware/...
```

### Test Environment

Tests require environment variables. In CI these come from secrets.
Locally, source `setTestEnv.sh` before running tests:

```bash
source setTestEnv.sh
```

### Cross-Platform Vet (CI runs all six)

```bash
GOOS=linux GOARCH=amd64 go vet -v ./...
GOOS=darwin GOARCH=arm64 go vet -v ./...
```

## Project Structure

- `config/` - Configuration loading from `.env` files
- `controller/` - HTTP request handlers (thin layer, calls handler)
- `handler/` - Business logic layer (called by controller)
- `service/` - Utility services (auth, security, email)
- `database/` - DB connection management and models
- `database/model/` - Shared data models (Auth, HTTPResponse)
- `database/migrate/` - GORM auto-migration
- `lib/` - Core utility library (hashing, encryption, validation)
- `lib/middleware/` - Gin middleware (JWT, CORS, firewall, rate limit)
- `lib/renderer/` - HTTP response rendering (JSON and HTML)
- `lib/server/` - Graceful server shutdown
- `example/` - Legacy example app
- `example2/` - Recommended example app (interface-driven, DI)

### example2/ Structure (Recommended)

```text
example2/
├── cmd/app/main.go          # Entry point
└── internal/
    ├── database/
    │   ├── migrate/migrate.go  # Auto-migration
    │   └── model/models.go     # App-specific GORM models
    ├── handler/                # HTTP handlers (Gin context binding)
    ├── repo/                   # Repository pattern (data access)
    ├── router/router.go        # Route definitions + middleware setup
    └── service/                # Business logic
```

### Layer Responsibilities

| Layer | Package | Responsibility |
| ----- | ------- | -------------- |
| Controller | `controller/` | Bind request, call handler, render response |
| Handler | `handler/` | Business logic, validation, returns `(HTTPResponse, int)` |
| Service | `service/` | Shared utilities (auth, email, crypto, JWT blacklist) |
| Repository | `example2/internal/repo/` | Data access abstraction (interface-driven) |
| Database | `database/` | Connection management (RDBMS, Redis, MongoDB) |
| Config | `config/` | Load `.env`, expose `GetConfig()`, feature checks |

### Controller-Handler 1:1 Mapping

Every controller function in `controller/` is a thin wrapper that binds
the HTTP request and calls the corresponding function in `handler/`.
Controllers use `renderer.Render()` to send the response. There are
19 exported functions in each package with a 1:1 correspondence. Every
handler takes a `context.Context` as its first parameter; controllers pass
`c.Request.Context()` so handlers honor client cancellation/deadlines:

| Controller | Handler | Description |
| ---------- | ------- | ----------- |
| `CreateUserAuth` | `CreateUserAuth(ctx, auth model.Auth)` | User registration |
| `Login` | `Login(ctx, payload model.AuthPayload)` | User login |
| `Refresh` | `Refresh(ctx, claims middleware.MyCustomClaims)` | Refresh JWT |
| `Logout` | `Logout(ctx, jtiAccess, jtiRefresh string, expAccess, expRefresh int64)` | Invalidate tokens |
| `PasswordForgot` | `PasswordForgot(ctx, authPayload model.AuthPayload)` | Send recovery email |
| `PasswordRecover` | `PasswordRecover(ctx, authPayload model.AuthPayload)` | Reset password |
| `PasswordUpdate` | `PasswordUpdate(ctx, claims, authPayload)` | Change password |
| `Setup2FA` | `Setup2FA(ctx, claims, authPayload)` | Generate 2FA secret |
| `Activate2FA` | `Activate2FA(ctx, claims, authPayload)` | Enable 2FA |
| `Validate2FA` | `Validate2FA(ctx, claims, authPayload)` | Verify OTP |
| `Deactivate2FA` | `Deactivate2FA(ctx, claims, authPayload)` | Disable 2FA |
| `CreateBackup2FA` | `CreateBackup2FA(ctx, claims, authPayload)` | Generate backup codes |
| `ValidateBackup2FA` | `ValidateBackup2FA(ctx, claims, authPayload)` | Use backup code |
| `VerifyEmail` | `VerifyEmail(ctx, payload model.AuthPayload)` | Verify email |
| `CreateVerificationEmail` | `CreateVerificationEmail(ctx, payload model.AuthPayload)` | Resend verification |
| `VerifyUpdatedEmail` | `VerifyUpdatedEmail(ctx, payload model.AuthPayload)` | Verify email change |
| `UpdateEmail` | `UpdateEmail(ctx, claims, req model.TempEmail)` | Submit new email |
| `GetUnverifiedEmail` | `GetUnverifiedEmail(ctx, claims)` | Get pending email |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pilinux/gorest](https://github.com/pilinux/gorest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
