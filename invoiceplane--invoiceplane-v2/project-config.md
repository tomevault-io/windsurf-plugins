---
trigger: always_on
description: This is the default Copilot prompt for this project.
---

# GitHub Copilot Context

This is the default Copilot prompt for this project.

## How to Use These Instructions

**IMPORTANT:** These instructions contain comprehensive information about the InvoicePlane v2 codebase, architecture, and development workflow. **Trust these instructions** and use them as your primary reference. Only perform additional searches if:
- The information you need is not covered here
- You encounter an error that contradicts these instructions
- You need to locate specific files or code not referenced here

Following these guidelines will significantly reduce exploration time and prevent common mistakes.

## Project Description

This project is **InvoicePlane v2**, a **multi-tenant Laravel application** with a **modular architecture**.

- The application uses **Laravel Filament** for Admin Panel, Company Panel, and InvoicePanel interfaces.
- Code is structured into **Modules**, each module encapsulating its own logic (models, services, repositories, DTOs,
 transformers, tests, etc.).
- Tests for each module are located in:
 `/Modules/(ModuleName)/Tests`

## Tech Stack

- **Backend:** Laravel 12+ (PHP 8.2+)
- **UI Framework:** Filament 4.0
- **Frontend:** Livewire, Tailwind CSS
- **Testing:** PHPUnit 11+
- **Code Quality:** Laravel Pint (PSR-12), PHPStan, Rector
- **Module System:** nwidart/laravel-modules
- **Permissions:** spatie/laravel-permission
- **Multi-tenancy:** Filament Companies with `BelongsToCompany` trait
- **Queue System:** Required for export functionality (Redis, database, or sync for local development)

## Development Commands

### Testing
```bash
# Run all tests (typically 30-60 seconds)
php artisan test

# Run tests with coverage (typically 60-120 seconds)
php artisan test --coverage

# Run specific test suite (faster - 10-30 seconds each)
php artisan test --testsuite=Unit
php artisan test --testsuite=Feature
```

**Important:** Always run tests before finalizing changes. All tests must pass.

### Code Quality
```bash
# Format code with Laravel Pint (auto-fixes PSR-12 violations)
vendor/bin/pint

# Run static analysis (typically 20-40 seconds)
vendor/bin/phpstan analyse

# Run Rector for automated refactoring
vendor/bin/rector process --dry-run
```

**Validation Pipeline:** Before submitting code, you MUST run:
1. `vendor/bin/pint` - Format code
2. `vendor/bin/phpstan analyse` - Check for type errors
3. `php artisan test` - Run all tests

### Setup & Installation
```bash
# See .github/INSTALLATION.md for detailed setup
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed

# Start queue worker for export functionality
php artisan queue:work
```

**Queue Configuration:**
- Export functionality requires a queue worker to be running
- For local development, you can use `QUEUE_CONNECTION=sync` in `.env`
- For production, use Redis or database queue driver with Supervisor

**GitHub Actions CI/CD:**
The following automated checks run on every pull request and MUST pass:
- **PHPUnit** - All tests must pass (`php artisan test`)
- **PHPStan** - Static analysis must pass with no errors (`vendor/bin/phpstan analyse`)
- **Pint** - Code must follow PSR-12 standards (`vendor/bin/pint`)
- **Docker Build** - Docker images must build successfully

See `.github/workflows/` for workflow configurations. Reference `.github/workflows/README.md` for setup details.

## Related Documentation

- **Installation:** `.github/INSTALLATION.md`
- **Contributing:** `.github/CONTRIBUTING.md`
- **Seeding:** `.github/SEEDING.md`
- **Testing:** See test examples in `Modules/*/Tests/`
- **Commit Conventions:** `.github/git-commit-instructions.md`

## Project Layout

### Repository Structure

```
├── .github/               # Workflows, docs, issue/PR templates, Copilot instructions
├── Modules/               # All application logic lives here
│   ├── Clients/           # Client/customer management
│   ├── Core/              # Shared infrastructure, panels, base test cases
│   ├── Expenses/          # Expense tracking
│   ├── Invoices/          # Invoice management + Peppol e-invoicing
│   ├── Payments/          # Payment processing
│   ├── Products/          # Product/item catalog
│   ├── Projects/          # Project management
│   └── Quotes/            # Quote/estimate management
├── app/                   # Laravel bootstrap only — business logic is in Modules/
├── config/                # Laravel configuration files
├── database/seeders/      # Database seeders
└── routes/                # web.php, console.php
```

### Module Directory Conventions

Every module follows the same internal structure:

```
Modules/{Name}/
├── Database/              # Migrations and model factories
├── Enums/                 # PHP 8.1+ enums (used in $casts)
├── Events/ & Listeners/   # Domain events and their listeners
├── Filament/              # Filament UI resources, pages, and components
│   ├── Admin/             # Admin panel resources (Core module only)
│   └── Company/           # Company panel resources
├── Models/                # Eloquent models (no $fillable, no timestamps unless specified)
├── Observers/             # Eloquent observers
├── Providers/             # Module service providers
├── Services/              # Business logic (use Transformers, not raw DTOs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InvoicePlane/InvoicePlane-v2](https://github.com/InvoicePlane/InvoicePlane-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
