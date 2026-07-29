---
trigger: always_on
description: **YAFFA (Yet Another Free Financial Application)** is a self-hosted personal finance web application built with **Laravel 12** (PHP 8.4) and **Vue 3**. It helps users track income/expenses, manage multiple currencies, handle investments, and perform long-term financial planning. The application includes ~7,700 lines of PHP code across Controllers, Models, Services, and more.
---

# YAFFA - Coding Agent Instructions

## Project Overview

**YAFFA (Yet Another Free Financial Application)** is a self-hosted personal finance web application built with **Laravel 12** (PHP 8.4) and **Vue 3**. It helps users track income/expenses, manage multiple currencies, handle investments, and perform long-term financial planning. The application includes ~7,700 lines of PHP code across Controllers, Models, Services, and more.

**Key Technologies:**

- Backend: PHP 8.4, Laravel 12 Framework
- Frontend: Vue 3, Bootstrap 5, CoreUI, jQuery, DataTables
- Build: Laravel Mix (Webpack), NPM
- Testing: PHPUnit (Unit/Feature tests), Laravel Dusk (Browser tests)
- Database: MySQL 8
- Code Quality: Laravel Pint (PSR-12), PHPStan (Level 5), ESLint

## Repository Structure

```
├── app/                    # Laravel application code
│   ├── Console/           # Artisan commands
│   ├── Http/Controllers/  # Request handlers
│   ├── Models/           # Eloquent models
│   ├── Services/         # Business logic (TransactionService, InvestmentService, etc.)
│   ├── Policies/         # Authorization logic
│   ├── Jobs/             # Queue jobs
│   └── Rules/            # Validation rules
├── resources/
│   ├── js/               # Vue 3 components and JavaScript
│   ├── views/            # Blade templates
│   └── sass/             # SCSS stylesheets
├── tests/
│   ├── Unit/             # Unit tests
│   ├── Feature/          # Feature tests
│   └── Browser/          # Laravel Dusk browser tests
├── config/               # Laravel configuration files
├── database/
│   ├── migrations/       # 43 database migration files
│   ├── factories/        # Model factories for testing
│   └── seeders/          # Database seeders (including demo.sql)
├── routes/
│   ├── web.php           # Web routes
│   ├── api.php           # API routes
│   └── breadcrumbs.php   # Breadcrumb definitions
├── docker/               # Docker deployment files
├── public/               # Public assets (CSS, JS - generated)
└── storage/              # Application storage (logs, cache)
```

**Important Configuration Files:**

- `composer.json` - PHP dependencies and scripts
- `package.json` - NPM dependencies and build scripts
- `phpunit.xml` - PHPUnit test configuration
- `phpstan.neon` - Static analysis configuration (Level 5)
- `pint.json` - Laravel Pint (PHP CS Fixer) rules (PSR-12 based)
- `.eslintrc.js` - ESLint configuration for Vue 3
- `webpack.mix.js` - Laravel Mix build configuration
- `.env.example` - Environment variable template
- `docker-compose.yml` - Local development with Laravel Sail

## Build & Development Commands

### Initial Setup

**CRITICAL: Composer install may encounter GitHub API rate limiting issues.** If you see "Could not authenticate against github.com" errors, this is due to GitHub API rate limits when downloading packages. This is a known environment limitation and may require:

- Waiting for rate limits to reset
- Using `--prefer-source` flag (slower but may bypass some issues)
- Accepting that some packages may need manual intervention

```bash
# 1. Install PHP dependencies (may take 5-10 minutes, can timeout at 300s due to GitHub rate limits)
composer install --prefer-dist --no-interaction

# 2. Install Node dependencies (~43 seconds)
npm ci

# 3. Setup environment file
cp .env.example .env
php artisan key:generate

# 4. Run database migrations (requires MySQL)
php artisan migrate
```

### Building Assets

**ALWAYS run asset builds before testing UI changes.**

```bash
# Development build (fast, for development)
npm run dev                    # ~30-40 seconds

# Production build (optimized, minified - use for deployment)
# This is used only for the `main` branch as part of releasing new versions.
npm run build             # ~32 seconds
```

**Build Output:** Generated files go to `public/js/` and `public/css/` - these are Git-ignored and should not be committed.

### Linting

**Run linters before committing code to catch style and quality issues.**

```bash
# PHP linting (PSR-12 code style)
./vendor/bin/pint              # Auto-fixes style issues

# PHP static analysis (PHPStan Level 5)
./vendor/bin/phpstan analyse   # Finds type errors and bugs

# JavaScript/Vue linting
npx eslint resources/js --ext .js,.vue
```

**Note:** Pint excludes `vendor/`, `public/`, `storage/`, `bootstrap/` directories. PHPStan analyzes `app/` directory only.

### Testing

**IMPORTANT:** The automated test suite requires a MySQL database. Tests are configured in `phpunit.xml` and use environment variables from `.env.ci` or `.env.dusk.ci`.

```bash
# Run PHPUnit tests (Unit and/or Feature tests, NO Dusk browser tests)
vendor/bin/phpunit --testsuite Unit
vendor/bin/phpunit --testsuite Feature
vendor/bin/phpunit --testsuite Unit,Feature

# Run Dusk browser tests (requires Chrome/Selenium)
php artisan dusk:chrome-driver --detect      # First-time setup
php artisan dusk                             # Run browser tests
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kantorge/yaffa](https://github.com/kantorge/yaffa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
