---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Liturgical Calendar API is a PSR-7/15/17 compliant REST API written in PHP 8.4+ that generates the Roman Catholic liturgical calendar for any given year.
It calculates mobile festivities and determines the precedence of solemnities, feasts, and memorials.
The API serves calendar data for nations, dioceses, or groups of dioceses in various formats: JSON, YAML, XML, or ICS.

**Key characteristics:**

- Data is based on official sources (Roman Missal editions, Magisterial documents, Dicastery Decrees)
- Historically accurate: calendars for past years reflect rules as they existed at that time
- Supports multiple languages via gettext
- PSR-7 compliant HTTP message handling with PSR-15 middleware architecture

## Development Commands

### Starting the API Server

The API requires at least 6 PHP workers since some routes make internal requests to other routes:

```bash
# Using composer (recommended)
composer start

# Using the script directly
./start-server.sh

# Manual approach with environment
PHP_CLI_SERVER_WORKERS=6 php -S localhost:8000 -t public
```

**Stop the server:**

```bash
composer stop
# or
./stop-server.sh
```

**Environment configuration:** Copy `.env.example` to `.env.local` and configure:

- `API_PROTOCOL` (http|https)
- `API_HOST` (localhost in dev)
- `API_PORT` (8000 in dev)
- `API_BASE_PATH` (empty in dev, e.g. /api/dev in production)
- `APP_ENV` (development|test|staging|production) - **Required in non-localhost environments**
  - `development` / `test`: Allow default password if `ADMIN_PASSWORD_HASH` is unset (for testing convenience)
  - `staging` / `production`: Require `ADMIN_PASSWORD_HASH` to be configured (throws exception if missing)
  - Invalid/unset values throw exception (fail-closed security)

**JWT Authentication configuration (required for protected endpoints):**

- `JWT_SECRET` - Secret key for signing tokens (minimum 32 characters, generate with `php -r "echo bin2hex(random_bytes(32));"`)
- `JWT_ALGORITHM` - Algorithm for signing (default: HS256)
- `JWT_EXPIRY` - Access token expiry in seconds (default: 3600 = 1 hour), must be positive
- `JWT_REFRESH_EXPIRY` - Refresh token expiry in seconds (default: 604800 = 7 days), must be positive
- `ADMIN_USERNAME` - Admin username for authentication (default: admin)
- `ADMIN_PASSWORD_HASH` - Argon2id password hash (generate with `password_hash('password', PASSWORD_ARGON2ID)`)
  - Required in `staging` and `production` environments
  - Optional in `development` and `test` environments (defaults to password "password")

**Protected Routes:** The following routes require JWT authentication (via HttpOnly cookie or `Authorization: Bearer <token>` header):

- `PUT /data/{category}/{calendar}` - Create calendar data
- `PATCH /data/{category}/{calendar}` - Update calendar data
- `DELETE /data/{category}/{calendar}` - Delete calendar data

**Authentication Endpoints:**

- `POST /auth/login` - Authenticate with username/password, returns access and refresh tokens (sets HttpOnly cookies)
- `POST /auth/refresh` - Refresh access token using refresh token (reads from cookie or body)
- `POST /auth/logout` - End session and clear HttpOnly cookies
- `GET /auth/me` - Check authentication state (returns user info from token, essential for cookie-based auth)

**Cookie-Based Authentication (Phase 2.5):**

The API supports full cookie-only authentication where:

- Tokens are stored in HttpOnly cookies (not accessible to JavaScript, mitigating token theft via XSS)
- `JwtAuthMiddleware` reads token from cookie first, falls back to Authorization header
- `RefreshHandler` reads refresh token from cookie, no request body needed
- Frontend uses `credentials: 'include'` to send cookies automatically

**CORS Configuration:**

- `CORS_ALLOWED_ORIGINS` - Comma-separated list of allowed origins for credentialed CORS requests
  - Default: `*` (all origins allowed - not recommended for production with cookies)
  - Example: `CORS_ALLOWED_ORIGINS=https://example.com,https://admin.example.com`
  - Auth endpoint errors only reflect validated origins (security measure)

See [Authentication Roadmap](docs/enhancements/AUTHENTICATION_ROADMAP.md) for implementation details.

### Local Development Bootstrap

The API server runs on the **host** (not in a container). The `docker-compose.yml`
stack provides the dependent infrastructure (Postgres, Zitadel, OpenFGA,
Mailpit, Adminer) plus a one-shot `litcal-migrate` container that applies
all pending Doctrine migrations against the litcal database. Fresh-clone
setup is a three-step sequence:

```bash
# 1. Start (or restart) the infrastructure and apply migrations.
#
#    Always pass --build so newly-pulled migration files land in the
#    litcal-migrate image before it runs. --build is cheap when nothing
#    in the build context changed (cached layers).
#
#    On first run: scripts/init-db.sql creates roles, databases, the
#    pgcrypto extension, and the empty doctrine_migration_versions
#    tracking table. The litcal-migrate one-shot then applies
#    Version20260518120000 (and any later migrations) to create the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liturgical-Calendar/LiturgicalCalendarAPI](https://github.com/Liturgical-Calendar/LiturgicalCalendarAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
