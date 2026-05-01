---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel application for managing database server backups. It uses Livewire for reactive components and Mary UI (robsontenorio/mary, built on daisyUI and Tailwind CSS). The application allows users to register database servers (MySQL, PostgreSQL, MariaDB, SQLite, Redis/Valkey), test connections, and manage backup configurations. See Boost's Foundational Context below for exact package versions.

## Development Commands

**IMPORTANT**: All PHP commands MUST be run through Docker. Never run `php`, `composer`, or `vendor/bin/*` commands directly on the host. Use the Makefile targets or `docker compose exec --user application -T app <command>` instead. Always include `--user application` to ensure correct file permissions.

### Setup and Installation
```bash
make setup              # Full project setup: install deps, env setup, generate key, migrate, build assets
make install            # Install composer and npm dependencies only
docker compose exec --user application -T app composer require <package>  # Install a composer package
docker compose exec --user application -T app composer remove <package>   # Remove a composer package
```

### Running the Application
```bash
make start              # Start all Docker services: php (FrankenPHP), queue worker, mysql, postgres, redis
docker compose up -d    # Alternative: direct docker compose command
docker compose logs -f  # View logs from all services
docker compose logs -f queue  # View queue worker logs only
```

### Testing

**IMPORTANT**: ALWAYS use `make test` commands for running tests. NEVER use `docker compose exec ... php artisan test` directly - it runs tests sequentially and is much slower.

```bash
make test                           # Run all tests in parallel (fast iteration) - ALWAYS USE THIS
make test-sequential                # Run tests sequentially (for debugging only)
make test-filter FILTER=DatabaseServer  # Run specific test class/method
make test-coverage                  # Run tests with coverage report
```

Tests run in parallel by default using Pest's parallel testing feature. This significantly speeds up the test suite (~12-18s for 350+ tests). Use `make test-sequential` if you need to debug test order issues.

### Test Strategy
- Focus on testing business logic and behaviors
- Do not test framework internals or trust that Laravel/Livewire works correctly
- Keep tests minimal and focused - one test per behavior when possible

#### What NOT to Test
- **Form validation rules** - Laravel validation works, don't test `required`, `max:255`, etc.
- **Eloquent relationships** - Don't test that `hasMany`/`belongsTo` work
- **Eloquent cascades** - Don't test `onDelete('cascade')` behavior
- **Session flash messages** - Don't test that `session('status')` contains a message
- **Redirect responses** - Testing redirect URL once per flow is enough
- **Multiple variations of the same thing** - e.g., don't test weekly AND daily recurrence separately

#### What TO Test
- **Authorization** - Who can access what (guests, users, admins)
- **Business logic** - Core application behavior (backup works, restore works, cleanup deletes correct snapshots)
- **Integration points** - External services, commands, scheduled tasks
- **Edge cases in YOUR code** - Not edge cases in the framework

#### Mocking Strategy

**DO Mock:**
- External API services
- Third-party libraries (AWS SDK, S3 client, etc.)

**DON'T Mock:**
- Model/ORM methods
- Simple utility functions

### Code Quality
```bash
make lint-fix           # Auto-fix code style with Laravel Pint (recommended)
make lint-check         # Check code style without fixing

make phpstan            # Run PHPStan static analysis
make analyse            # Alias for phpstan
```

### Database Operations
```bash
make migrate                # Run pending migrations
make migrate-fresh          # Drop all tables and re-migrate
make migrate-fresh-seed     # Fresh migration with seeders
make db-seed                # Run database seeders
```

### Asset Building
```bash
npm run build           # Build production assets with Vite
npm run dev             # Start Vite dev server for hot module replacement
make build              # Alternative: build via Makefile
```

### Docker Services
```bash
make start                      # Start all services (php, queue worker, mysql, postgres, redis)
docker compose up -d            # Alternative: direct docker compose command
docker compose down             # Stop all services
docker compose down -v          # Stop and remove volumes
docker compose logs -f queue    # View queue worker logs
docker compose restart queue    # Restart queue worker (after code changes)
```

The Docker setup provides:
- **php**: FrankenPHP server on port 2226 (http://localhost:2226)
- **queue**: Queue worker processing backup/restore jobs
- **mysql**: MySQL 8.0 on port 3306 (user: admin, password: admin, db: testdb)
- **postgres**: PostgreSQL 16 on port 5432 (user: admin, password: admin, db: testdb)
- **redis**: Redis 7 on port 6379


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [David-Crty/databasement](https://github.com/David-Crty/databasement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
