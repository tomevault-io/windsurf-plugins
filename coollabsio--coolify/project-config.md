---
trigger: always_on
description: This file provides guidance to agentic coding tools when working with code in this repository.
---

# AGENTS.md

This file provides guidance to agentic coding tools when working with code in this repository.

## Project Overview

Coolify is an open-source, self-hostable PaaS (alternative to Heroku/Netlify/Vercel). It manages servers, applications, databases, and services via SSH. Built with Laravel 12 (using Laravel 10 file structure), Livewire 3, and Tailwind CSS v4.

## Design Reference

For UI/UX design specifications, principles, and visual standards, consult `DESIGN.md` in the [coollabsio/architecture](https://github.com/coollabsio/architecture) repo.

## Development Environment

Docker Compose-based dev setup with services: coolify (app), postgres, redis, soketi (WebSockets), vite, testing-host, mailpit, minio.

```bash
# Start dev environment (uses docker-compose.dev.yml)
spin up                          # or: docker compose -f docker-compose.dev.yml up -d
spin down                        # stop services
```

The app runs at `localhost:8000` by default. Vite dev server on port 5173.

## Common Commands

```bash
# Tests (Pest 4)
php artisan test --compact                          # all tests
php artisan test --compact --filter=testName         # single test
php artisan test --compact tests/Feature/SomeTest.php  # specific file

# Code formatting (Pint, Laravel preset)
vendor/bin/pint --dirty --format agent              # format changed files

# Frontend
npm run dev                     # vite dev server
npm run build                   # production build
```

## Browser Tests (Pest Browser Plugin)

Uses `pestphp/pest-plugin-browser` with Laravel Dusk 8. New browser tests go in `tests/v4/Browser/`.

```bash
# Run all browser tests
php artisan test --compact tests/v4/Browser/

# Run a specific browser test file
php artisan test --compact tests/v4/Browser/LoginTest.php

# Run a specific test by name
php artisan test --compact --filter='can login with valid credentials'
```

### Writing Browser Tests

- Place new tests in `tests/v4/Browser/` — legacy Dusk tests in `tests/Browser/` should not be used as reference.
- Use `RefreshDatabase` and seed required data (at minimum `InstanceSettings::create(['id' => 0])`) in `beforeEach`.
- Key API: `visit()`, `fill(field, value)`, `click(text)`, `assertSee()`, `assertDontSee()`, `assertPathIs()`, `screenshot()`.
- Always call `screenshot()` at the end of each test for debugging.
- For authenticated tests, create a helper function that logs in via the UI:

```php
function loginAsRoot(): mixed
{
    return visit('/login')
        ->fill('email', 'test@example.com')
        ->fill('password', 'password')
        ->click('Login');
}
```

- See `tests/v4/Browser/LoginTest.php`, `tests/v4/Browser/DashboardTest.php`, and `tests/v4/Browser/RegistrationTest.php` for conventions.
- Chrome driver runs on `localhost:4444`, app on `localhost:8000` (configured in `tests/DuskTestCase.php`).
- Legacy Dusk macros in `app/Providers/DuskServiceProvider.php` use the old `type()`/`press()` API — do not mix with Pest Browser Plugin's `fill()`/`click()` API.

## Architecture

### Backend Structure (app/)
- **Actions/** — Domain actions organized by area (Application, Database, Docker, Proxy, Server, Service, Shared, Stripe, User, CoolifyTask, Fortify). Uses `lorisleiva/laravel-actions` with `AsAction` trait — actions can be called as objects, dispatched as jobs, or used as controllers.
- **Livewire/** — All UI components (Livewire 3). Pages organized by domain: Server, Project, Settings, Security, Notifications, Terminal, Subscription, SharedVariables. This is the primary UI layer — no traditional Blade controllers. Components listen to private team channels for real-time status updates via Soketi.
- **Jobs/** — Queue jobs for deployments (`ApplicationDeploymentJob`), backups, Docker cleanup, server management, proxy configuration. Uses Redis queue with Horizon for monitoring.
- **Models/** — Eloquent models extending `BaseModel` which provides auto-CUID2 UUID generation. Key models: `Server`, `Application`, `Service`, `Project`, `Environment`, `Team`, plus standalone database models (`StandalonePostgresql`, `StandaloneMysql`, etc.). Common traits: `HasConfiguration`, `HasMetrics`, `HasSafeStringAttribute`, `ClearsGlobalSearchCache`.
- **Services/** — Business logic services (ConfigurationGenerator, DockerImageParser, ContainerStatusAggregator, HetznerService, etc.). Use Services for complex orchestration; use Actions for single-purpose domain operations.
- **Helpers/** — Global helpers loaded via `bootstrap/includeHelpers.php` from `bootstrap/helpers/` — organized into `shared.php`, `constants.php`, `versions.php`, `subscriptions.php`, `domains.php`, `docker.php`, `services.php`, `github.php`, `proxy.php`, `notifications.php`.
- **Data/** — Spatie Laravel Data DTOs (e.g., `ServerMetadata`).
- **Enums/** — PHP enums (TitleCase keys). Key enums: `ProcessStatus`, `Role` (MEMBER/ADMIN/OWNER with rank comparison), `BuildPackTypes`, `ProxyTypes`, `ContainerStatusTypes`.
- **Rules/** — Custom validation rules (`ValidGitRepositoryUrl`, `ValidServerIp`, `ValidHostname`, `DockerImageFormat`, etc.).

### API Layer
- REST API at `/api/v1/` with OpenAPI 3.0 attributes (`use OpenApi\Attributes as OA`) for auto-generated docs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coollabsio/coolify](https://github.com/coollabsio/coolify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
