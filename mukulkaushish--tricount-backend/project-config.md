---
trigger: always_on
description: Server-side Swift backend for the **Tricount** Flutter expense-splitting app.
---

# AGENTS.md — tricount-backend

Server-side Swift backend for the **Tricount** Flutter expense-splitting app.
Built with [Vapor 4](https://vapor.codes), Fluent ORM, MySQL, and JWT auth.

---

## Quick start

```bash
# 1. Resolve packages (must do this first after cloning or adding deps)
swift package resolve

# 2. Set required environment variables (see below)
export JWT_SECRET="your-strong-random-secret-here"

# 3. Run (auto-migrates the DB on startup)
swift run

# Server listens on http://localhost:8080
# Health check: GET /
```

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `JWT_SECRET` | `change-me-in-production-use-env-var` | HMAC-SHA256 signing key — **always override in prod** |
| `MYSQL_HOST` | `127.0.0.1` | MySQL hostname |
| `MYSQL_PORT` | `3306` | MySQL port |
| `MYSQL_USERNAME` | `tricount` | MySQL username |
| `MYSQL_PASSWORD` | `tricount` | MySQL password |
| `MYSQL_DATABASE` | `tricount` | MySQL database name |
| `ROUTE_DOCS_OUTPUT_DIR` | `Generated` locally / `.build/generated-route-docs` in tests | Output directory for the startup-generated `routes.md` and Postman collection files |
| `GOOGLE_CLIENT_ID` | — | Optional Google OAuth audience check |
| `APPLE_APPLICATION_IDENTIFIER` | — | Optional Sign in with Apple audience check (`client_id` / bundle id) |
| `APPLE_BUNDLE_ID` | — | Alias for `APPLE_APPLICATION_IDENTIFIER` |
| `LOG_LEVEL` | `info` | Vapor log level (`debug`, `info`, `warning`, `error`) |

---

## Project layout

```
Sources/TricountBackend/
├── configure.swift          # Thin app bootstrap orchestration
├── routes.swift             # Top-level route registration (/v1/...)
├── entrypoint.swift         # main()
│
├── Configuration/
│   ├── Application+Middleware.swift # Global middleware registration
│   ├── Application+JWT.swift        # JWT signer configuration
│   ├── Application+Database.swift   # MySQL database setup
│   ├── Application+Migrations.swift # Migration registration order
│   └── Application+RouteDocumentation.swift # Startup route-doc output configuration
│
├── Controllers/
│   ├── AuthController.swift    # All /v1/auth/* route handlers
│   └── TodoController.swift    # /v1/todos CRUD routes
│
├── DTOs/
│   ├── AuthDTOs.swift       # Request & response Codable structs
│
├── Documentation/
│   └── RouteDocumentationGenerator.swift # Writes startup-generated routes.md, Postman collection, and HTML docs
│
├── Middleware/
│   ├── ErrorMiddleware.swift      # Global: all errors → standard JSON shape
│   ├── JWTAuthMiddleware.swift    # Verifies Bearer JWT; defines AuthError
│   ├── LoggingMiddleware.swift    # Structured request/response logging
│   └── RateLimitMiddleware.swift  # Sliding-window in-memory rate limiter
│
├── Migrations/
│   ├── 001_CreateTodo.swift       # initial todo table
│   ├── 002_CreateUser.swift       # users table
│   ├── 003_AddUserVerificationFields.swift # verification fields for existing DBs
│   ├── 004_AddAppleIdentifierToUsers.swift # Apple identity linkage for existing DBs
│   ├── 005_CreateEmailVerificationOTP.swift # OTP storage for email verification
│   └── 006_CreateRefreshToken.swift # refresh_tokens table
│
├── Models/
│   ├── User.swift                 # Fluent model
│   ├── EmailVerificationOTP.swift # Email verification OTP model
│   └── RefreshToken.swift         # Fluent model (stores token hash, never raw)
│
├── Payload/
│   └── UserJWTPayload.swift       # JWTPayload struct + req.jwtPayload helper
│
└── Services/
    ├── AuthService.swift          # Login, register, Google/Apple SSO, verify-profile, refresh, logout
    ├── AppleAuthService.swift     # Verifies Apple identity tokens via Apple's JWKS
    ├── AuthValidation.swift       # Shared auth/email normalization + validation
    ├── EmailVerificationOTPDispatcher.swift # OTP delivery abstraction (logs by default)
    └── GoogleAuthService.swift    # Verifies Google ID token via tokeninfo API
```

---

## API endpoints (Phase 2 — Auth)

Base URL: `GET /v1`

Versioned todo routes:
- `GET /v1/todos`
- `POST /v1/todos`
- `DELETE /v1/todos/:todoID`

| Method | Path | Auth | Rate limit |
|---|---|---|---|
| `POST` | `/v1/auth/login` | — | disabled |
| `POST` | `/v1/auth/register` | — | disabled |
| `POST` | `/v1/auth/forgot-password` | — | 3/hour per email |
| `POST` | `/v1/auth/google` | — | disabled |
| `POST` | `/v1/auth/apple` | — | disabled |
| `POST` | `/v1/auth/refresh` | — | disabled |
| `POST` | `/v1/auth/verify-profile/google` | Bearer JWT | disabled |
| `POST` | `/v1/auth/verify-profile/apple` | Bearer JWT | disabled |
| `POST` | `/v1/auth/verify-profile/email/request-otp` | Bearer JWT | disabled |
| `POST` | `/v1/auth/verify-profile/email/confirm` | Bearer JWT | disabled |
| `GET` | `/v1/auth/me` | Bearer JWT | disabled |
| `POST` | `/v1/auth/logout` | Bearer JWT | disabled |

Full contract: `API_CONTRACT.md`

---

## Architecture decisions

### Rate limiting
Actor-based in-memory sliding-window limiter (`RateLimitStore`) applied globally through a
single `RateLimitMiddleware`. The global fallback is `RateLimitPolicy.disabled`, so routes
are unlimited unless they explicitly attach either `RateLimitPolicy.default` or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mukulkaushish) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
