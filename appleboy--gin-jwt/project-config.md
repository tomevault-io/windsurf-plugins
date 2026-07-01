---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**gin-jwt** is a JWT authentication middleware for the Gin web framework. It provides RFC 6749 compliant OAuth 2.0 refresh tokens with pluggable storage backends (in-memory, Redis with client-side caching). The library supports direct token generation, multiple JWT providers via dynamic key functions, and comprehensive cookie/header token management.

## Development Commands

### Testing

```bash
# Run all tests with coverage
make test

# Run tests with race detection and coverage report
go test -v -race -coverprofile=coverage.out -covermode=atomic ./...

# Generate HTML coverage report
make coverage

# Run specific test
go test -v -run TestFunctionName ./...

# Run tests in a specific package
go test -v ./store/...
```

### Code Quality

```bash
# Format code (uses golangci-lint fmt with gofmt, gofumpt, goimports, golines)
make format

# Run linter
make lint

# Run go vet
make vet

# Clean test cache and coverage files
make clean
```

### Development Setup

```bash
# Install required development tools (golangci-lint)
make install-tools
```

### Running Examples

```bash
# Basic authentication example
go run _example/basic/server.go

# OAuth SSO integration example
go run _example/oauth_sso/server.go

# Token generator example (direct token creation)
go run _example/token_generator/main.go

# Redis store examples
go run _example/redis_simple/main.go
go run _example/redis_store/main.go
go run _example/redis_tls/main.go

# Authorization example
go run _example/authorization/main.go
```

## Architecture

### Core Components

**auth_jwt.go**: Main middleware implementation containing `GinJWTMiddleware` struct and HTTP handlers (LoginHandler, RefreshHandler, LogoutHandler, MiddlewareFunc). Handles JWT token creation, validation, and the complete authentication flow.

**core/**: Core interfaces and types

- `core/store.go`: Defines `TokenStore` interface for refresh token storage backends
- `core/token.go`: `Token` struct representing JWT token pairs with metadata

**store/**: Refresh token storage implementations

- `store/memory.go`: Thread-safe in-memory store (single-instance applications)
- `store/redis.go`: Redis-based store with rueidis client-side caching (distributed systems)
- `store/factory.go`: Factory functions for creating stores

**auth_jwt_redis.go**: Redis store configuration via functional options pattern (EnableRedisStore, WithRedisAddr, WithRedisAuth, WithRedisTLS, etc.)

### Key Architectural Patterns

**RFC 6749 OAuth 2.0 Compliance**: Uses distinct opaque refresh tokens (not JWT) stored server-side with automatic rotation on refresh for enhanced security.

**Pluggable Storage Backend**: `core.TokenStore` interface allows swapping between in-memory, Redis, or custom storage implementations without changing middleware code.

**Token Generator Pattern**: Direct token creation via `TokenGenerator()` and `TokenGeneratorWithRevocation()` methods bypasses HTTP middleware for programmatic authentication, service-to-service communication, and testing.

**Dynamic Key Function**: `KeyFunc` callback enables multi-provider JWT validation by inspecting tokens before validation and choosing appropriate signing keys/methods dynamically. This supports hybrid authentication (internal + external providers like Azure AD, Auth0).

**Functional Options for Redis**: `EnableRedisStore()` with options like `WithRedisAddr()`, `WithRedisAuth()`, `WithRedisTLS()` provides flexible configuration.

### Authentication Flow

1. **Login** (`LoginHandler`): Authenticator validates credentials → PayloadFunc creates claims → JWT access token + opaque refresh token generated → Both stored (token in store, JWT signed) → Sent via headers/cookies
2. **Protected Routes** (`MiddlewareFunc`): Token extracted from header/query/cookie → JWT validated → IdentityHandler extracts identity → Authorizer checks permissions → Request proceeds or Unauthorized called
3. **Refresh** (`RefreshHandler`): Refresh token extracted from cookie/form/query/JSON → Server-side validation → Old token revoked → New access + refresh tokens generated (token rotation) → Sent via headers/cookies
4. **Logout** (`LogoutHandler`): Refresh token extracted → Revoked from store → Cookies cleared → LogoutResponse called

### Token Storage Strategy

**Access Tokens (JWT)**: Stateless, signed, short-lived (default 1 hour), contain claims for authorization, validated cryptographically.

**Refresh Tokens (Opaque)**: Stateful, stored server-side in `TokenStore`, long-lived (default 7 days), used only to obtain new access tokens, automatically rotated on use.

**Why Separate?**: Prevents JWT refresh token vulnerabilities, enables immediate revocation, follows OAuth 2.0 security best practices, allows scalable distributed token management via Redis.

## Testing Strategy

**Unit Tests**: All main files have corresponding `*_test.go` files. Tests use `testify` for assertions and `gofight` for HTTP testing.

**Integration Tests**: Redis tests use `testcontainers-go` to spin up real Redis instances, ensuring actual Redis behavior is tested.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appleboy/gin-jwt](https://github.com/appleboy/gin-jwt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
