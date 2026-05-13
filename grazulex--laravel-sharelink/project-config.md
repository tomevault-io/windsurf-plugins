---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Laravel ShareLink is a comprehensive Laravel package for generating secure, time-limited sharing capabilities. It allows applications to share files, routes, and model previews with advanced security features including password protection, IP filtering, rate limiting, and audit trails.

**Package Name**: `grazulex/laravel-sharelink`
**Requirements**: PHP 8.3+, Laravel 11.x | 12.x

## Development Commands

### Testing
```bash
# Run all tests with coverage
composer test
# Or directly with Pest
./vendor/bin/pest --colors=always --coverage
```

### Code Quality
```bash
# Run Laravel Pint (code style fixer)
composer pint
# Or directly
./vendor/bin/pint

# Run PHPStan static analysis (level 5)
composer phpstan
# Or directly
./vendor/bin/phpstan analyse --memory-limit=2G --configuration=phpstan.neon

# Run Rector for automated refactoring
composer rector
# Or directly
./vendor/bin/rector

# Run all quality checks in sequence
composer full
```

### Artisan Commands (Package)
The package provides these commands that can be tested during development:
```bash
# Create a share link
php artisan sharelink:create /path/to/file --expires=60 --max-clicks=5

# List all share links
php artisan sharelink:list --active --expired

# Revoke a link by token
php artisan sharelink:revoke abc123xyz

# Prune expired/revoked links older than N days
php artisan sharelink:prune --days=7
```

## Architecture Overview

### Core Service Layer
The package follows a service-oriented architecture with clear separation of concerns:

**ShareLinkManager** (`src/Services/ShareLinkManager.php`):
- Primary service for creating share links via a fluent `PendingShareLink` builder
- Handles URL generation (regular and signed URLs)
- Provides link extension functionality
- The `PendingShareLink` class builds up link configuration before calling `generate()` to persist

**ShareLinkRevoker** (`src/Services/ShareLinkRevoker.php`):
- Dedicated service for revoking links
- Fires `ShareLinkRevoked` events

### Model Layer
**ShareLink Model** (`src/Models/ShareLink.php`):
- Uses UUID primary keys via `HasUuids` trait
- Stores `resource` as JSON (can be string path or array for routes/models)
- Stores `metadata` as JSON for extensibility (IP filters, burn flags, etc.)
- Automatically generates 32-character random tokens on creation
- Dynamic `fillable` attributes: adds `created_by` when user tracking is enabled
- Key methods: `isExpired()`, `isRevoked()`, `incrementClicks()`, `markAccessed()`

### HTTP Layer
**ShareLinkController** (`src/Http/Controllers/ShareLinkController.php`):
- Main controller handling link access via `show()` method
- Middleware (`EnsureShareLinkIsValid`) validates token, expiration, revocation, IP filters, and rate limits BEFORE controller execution
- Handles password validation with throttling (429 responses on too many attempts)
- Increments click count and updates audit fields (first/last access, IP)
- Implements burn-after-reading: revokes or deletes link after first successful access
- Supports multiple resource types:
  - **Local files**: Direct streaming, supports HTTP Range requests for partial content (206)
  - **X-Sendfile/X-Accel-Redirect**: Optimized file delivery via web server
  - **Storage disks**: Handles `disk:path` format for Laravel Storage
  - **Routes**: Redirects to named routes with parameters
  - **Models**: Returns JSON preview (app can customize)
- Content negotiation: Returns JSON resource for API clients

**ManageShareLinkController** (`src/Http/Controllers/ManageShareLinkController.php`):
- Optional management endpoints (enabled via config)
- POST `/share/{token}/revoke` - Revoke a link
- POST `/share/{token}/extend` - Extend expiration
- Optional Gate authorization via `sharelink.management.gate` config

### Middleware & Security
**EnsureShareLinkIsValid** (`src/Http/Middleware/EnsureShareLinkIsValid.php`):
- Validates token exists and not revoked/expired
- Enforces max_clicks limits
- Checks IP allow/deny lists (global config + per-link metadata)
- Rate limiting per token (configurable max attempts/decay window)
- Returns standardized JSON error responses with status codes

### Events & Observability
The package fires events for all major actions:
- `ShareLinkCreated` - When a link is generated
- `ShareLinkAccessed` - On successful access
- `ShareLinkRevoked` - When manually revoked
- `ShareLinkExpired` - When accessed after expiration

**ShareLinkObserver** (`src/Support/ShareLinkObserver.php`):
- Subscribes to all events when `observability.enabled` is true
- Logs events (non-PII) when `observability.log` is enabled
- Can push to metrics sinks when `observability.metrics` is enabled

### Configuration Architecture
Configuration (`config/sharelink.php`) uses a nested structure:

- **route**: Prefix, middleware for access routes
- **management**: Enable/disable, middleware, gate for management endpoints
- **signed**: Enable signed URLs, require them, default TTL
- **burn**: Burn-after-reading settings (enabled, auto-detect, strategy)
- **limits**: IP filtering (allow/deny), rate limiting, password throttling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Grazulex/laravel-sharelink](https://github.com/Grazulex/laravel-sharelink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
