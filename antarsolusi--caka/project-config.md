---
trigger: always_on
description: This document provides the essential context needed to work effectively with this codebase. It is accurate as of the last time it was updated; when you change something described here, update this file too.
---

# Project Agent Guide

This document provides the essential context needed to work effectively with this codebase. It is accurate as of the last time it was updated; when you change something described here, update this file too.

---

## Project Overview

This is a **Laravel 13** web application skeleton. It is a brand-new, uncustomized Laravel project using modern Laravel conventions and the latest major versions of its toolchain. The application currently serves only the default Laravel welcome page at `/`.

- **Language**: PHP 8.3+
- **Framework**: Laravel 13.8
- **Frontend**: Vite 8 + Tailwind CSS v4 + Blade
- **Testing**: Pest PHP 4.7
- **Primary natural language**: English

---

## Technology Stack & Versions

### Backend
- `php`: ^8.3
- `laravel/framework`: ^13.8
- `laravel/tinker`: ^3.0

### Frontend
- `vite`: ^8.0.0
- `tailwindcss`: ^4.0.0
- `@tailwindcss/vite`: ^4.0.0
- `laravel-vite-plugin`: ^3.1
- `concurrently`: ^9.0.1

### Development & Testing
- `pestphp/pest`: ^4.7
- `pestphp/pest-plugin-laravel`: ^4.1
- `laravel/pint`: ^1.27 (code style)
- `laravel/pail`: ^1.2.5 (real-time log tailing)
- `laravel/pao`: ^1.0.6
- `nunomaduro/collision`: ^8.6
- `fakerphp/faker`: ^1.23
- `mockery/mockery`: ^1.6

---

## Directory Structure

Standard Laravel 13 layout:

```
app/
  Http/Controllers/     # HTTP controllers
  Models/               # Eloquent models
  Providers/            # Service providers
bootstrap/
  app.php               # Application bootstrap (Application::configure)
  providers.php         # Registered service providers
config/                 # Laravel configuration files
  app.php, auth.php, cache.php, database.php,
  filesystems.php, logging.php, mail.php, queue.php,
  services.php, session.php
database/
  factories/            # Eloquent model factories
  migrations/           # Schema migrations
  seeders/              # Database seeders
resources/
  css/app.css           # Tailwind CSS entry point
  js/app.js             # JavaScript entry point
  views/                # Blade templates
routes/
  web.php               # Web routes
  console.php           # Console / Artisan commands
tests/
  Feature/              # Feature tests (Pest)
  Unit/                 # Unit tests (Pest)
  Pest.php              # Pest configuration & shared helpers
  TestCase.php          # Base test case
public/                 # Web server document root
storage/                # Logs, caches, compiled views, etc.
```

---

## Build, Development & Test Commands

### Initial Setup
```bash
composer setup
```
This runs: `composer install`, copies `.env.example` to `.env`, generates `APP_KEY`, runs migrations, installs npm dependencies, and builds frontend assets.

### Development
```bash
composer dev
```
Runs four processes concurrently with color-coded output:
- `server` — `php artisan serve`
- `queue` — `php artisan queue:listen --tries=1 --timeout=0`
- `logs` — `php artisan pail --timeout=0`
- `vite` — `npm run dev`

If any process exits, all others are killed (`--kill-others`).

### Frontend Build
```bash
npm run dev     # Vite dev server with HMR
npm run build   # Production build (outputs to public/build/)
```

### Testing
```bash
composer test
```
This clears the config cache and then runs `php artisan test`, which delegates to Pest.

You can also run Pest directly:
```bash
./vendor/bin/pest
```

### Code Style
```bash
./vendor/bin/pint
```
Laravel Pint enforces the Laravel preset. Run this before committing.

### Common Artisan Commands
```bash
php artisan migrate:fresh --seed    # Reset DB and seed
php artisan db:seed                 # Run seeders
php artisan tinker                  # Interactive REPL
```

---

## Code Style Guidelines

- **PHP**: Follow the Laravel preset enforced by Pint. Key rules:
  - 4-space indentation
  - PSR-12 / Laravel coding style
  - Prefer explicit imports over facades when practical
- **EditorConfig** (already configured):
  - `charset = utf-8`
  - `end_of_line = lf`
  - `indent_size = 4` (2 for YAML files)
  - `insert_final_newline = true`
  - `trim_trailing_whitespace = true`
- **JavaScript / CSS**:
  - Vite + ES modules (`"type": "module"` in `package.json`)
  - Tailwind CSS v4 with `@import 'tailwindcss'` syntax
  - Font: Instrument Sans (400, 500, 600) loaded via Bunny Fonts through `laravel-vite-plugin/fonts`
- **Blade / HTML**: Use standard Blade directives. The welcome view uses Tailwind utility classes extensively.

---

## Testing Strategy

- **Framework**: Pest PHP (not plain PHPUnit)
- **Configuration**: `tests/Pest.php`
  - Feature tests extend `Tests\TestCase` via `pest()->extend(TestCase::class)->in('Feature')`
  - `RefreshDatabase` is available but **commented out** by default; uncomment if you need database isolation for Feature tests
- **Suites**:
  - `tests/Unit/` — Unit tests
  - `tests/Feature/` — HTTP / integration tests
- **Test Environment** (`phpunit.xml`):
  - `APP_ENV=testing`
  - `DB_CONNECTION=sqlite`
  - `DB_DATABASE=:memory:`
  - `CACHE_STORE=array`
  - `QUEUE_CONNECTION=sync`
  - `SESSION_DRIVER=array`
- **Helpers**: Add shared helpers to `tests/Pest.php` or create dedicated helper files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antarsolusi/caka](https://github.com/antarsolusi/caka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
