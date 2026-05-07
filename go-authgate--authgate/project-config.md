---
trigger: always_on
description: Enables secure authentication without embedding client secrets.
---

# CLAUDE.md

This file provides guidance to [Claude Code](https://claude.ai/code) when working with code in this repository.

## Project Overview

AuthGate is an OAuth 2.0 authorization server built with Go and Gin. It supports:

- **Device Authorization Grant (RFC 8628)** - For CLI tools, IoT devices, and headless environments
- **Authorization Code Flow (RFC 6749) with PKCE (RFC 7636)** - For web and mobile applications

Enables secure authentication without embedding client secrets.

## Common Commands

```bash
# Build (IMPORTANT: make generate is required before building)
make generate           # Generate templ templates and Swagger docs (REQUIRED before build)
make build              # Build to bin/authgate with version info in LDFLAGS

# Run
./bin/authgate -v       # Show version information
./bin/authgate -h       # Show help
./bin/authgate server   # Start the OAuth server

# Development
make dev                # Hot reload development mode (watches .go, .templ, .env, .css, .js)
make generate           # Compile .templ templates to Go code (REQUIRED)
make swagger            # Generate OpenAPI/Swagger documentation

# Test & Lint
make test               # Run tests with coverage report (outputs coverage.txt)
make coverage           # View test coverage in browser
make lint               # Run golangci-lint (auto-installs if missing)
make fmt                # Format code with golangci-lint fmt

# Cross-compile (outputs to release/<os>/<arch>/)
make build_linux_amd64  # CGO_ENABLED=0 for static binary
make build_linux_arm64  # CGO_ENABLED=0 for static binary

# Clean
make clean              # Remove bin/, release/, coverage.txt, generated templ files

# Docker
docker build -f docker/Dockerfile -t authgate .
```

## Architecture

### Device Authorization Flow (RFC 8628)

1. CLI calls `POST /oauth/device/code` → receives device_code + user_code + verification_uri
2. User visits `/device` in browser, must login first if not authenticated
3. User submits user_code via `POST /device/verify` → device code marked as authorized
4. CLI polls `POST /oauth/token` with device_code every 5s → receives access_token + refresh_token
5. When access_token expires, CLI uses `grant_type=refresh_token` to get new token

### Authorization Code Flow (RFC 6749)

In addition to Device Code Flow, AuthGate supports Authorization Code Flow with PKCE for web and mobile applications:

1. App redirects user to `GET /oauth/authorize` with client_id, redirect_uri, state, code_challenge (PKCE)
2. User logs in (if needed) and sees consent screen at `/oauth/authorize`
3. User approves → `POST /oauth/authorize` → redirects to redirect_uri with authorization code
4. App exchanges code for tokens via `POST /oauth/token` (with code_verifier for PKCE, or client_secret for confidential clients)

**User Consent Management**

- Users can review granted apps at `/account/authorizations`
- Each authorization grants specific scopes to one app
- Users can revoke per-app access (revokes all associated tokens)
- Admins can force re-authentication for all users of a client via `/admin/clients/:id/revoke-all`

**Client Types**

- **Confidential clients**: Web apps with server-side backends that can securely store client_secret
- **Public clients**: SPAs and mobile apps that use PKCE instead of client_secret

### Technology Stack

- **Web Framework**: [Gin](https://gin-gonic.com/) - Fast HTTP router
- **Templates**: [templ](https://templ.guide/) - Type-safe Go templates (requires `make generate` before build)
- **ORM**: [GORM](https://gorm.io/) - Database abstraction
- **Database**: SQLite (default) / PostgreSQL
- **Sessions**: Encrypted cookies with [gin-contrib/sessions](https://github.com/gin-contrib/sessions)
- **JWT**: [golang-jwt/jwt](https://github.com/golang-jwt/jwt)
- **API Documentation**: Swagger/OpenAPI via [swaggo/swag](https://github.com/swaggo/swag)
- **Hot Reload**: [air](https://github.com/air-verse/air) for development (`make dev`)

### Project Structure (Layers)

- `main.go` - Wires up store → auth providers → token providers → services → handlers
- `internal/config/` - Environment configuration management
- `internal/store/` - GORM-based data access layer (SQLite/PostgreSQL), uses map-based driver factory
- `internal/auth/` - Authentication providers (local, HTTP API, OAuth providers)
- `internal/token/` - Token provider (local JWT)
- `internal/services/` - Business logic layer (user, device, authorization, token, client, audit services)
- `internal/handlers/` - HTTP request handlers for all endpoints
- `internal/models/` - GORM database models (User, OAuthApplication, UserAuthorization, DeviceCode, AuthorizationCode, AccessToken, OAuthConnection, AuditLog)
- `internal/middleware/` - Gin middleware (auth, CSRF, rate limiting, metrics auth, CORS)
- `internal/metrics/` - Prometheus metrics collection and caching (supports memory, Redis, Redis-aside)
- `internal/cache/` - Cache implementations (memory, Redis, Redis-aside)
- `internal/client/` - HTTP client with exponential backoff retry
- `internal/templates/` - templ templates (`_.templ` → `_templ.go` via `make generate`)
- `internal/util/` - Utility functions (crypto, context helpers)
- `internal/version/` - Version information injected at build time


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-authgate/authgate](https://github.com/go-authgate/authgate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
