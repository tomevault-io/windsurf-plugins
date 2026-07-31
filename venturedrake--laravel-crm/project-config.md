---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Laravel **package** (`venturedrake/laravel-crm`) that installs a full CRM into any Laravel application. All source lives under `src/` with namespace `VentureDrake\LaravelCrm\`. Supports Laravel 11–13, PHP 8.2+, Livewire 3–4.

This package is developed in a **multi-project workspace**. Changes here take effect immediately in host apps (via Composer path repository) — no publish step needed for code:

| Project | Path | Purpose |
|---|---|---|
| **This package** | `/Users/andrewdrake/Packages/laravel-crm` | All CRM source code |
| **Laravel 12 host** | `/Users/andrewdrake/Sites/laravel-12-crm-v2` | Primary dev/test host |
| **Premium host** | `/Users/andrewdrake/Sites/laravel-crm-premium` | Laravel 11 + Jetstream + Xero |

## Commands

```bash
# Frontend
npm run dev          # Vite watch mode
npm run build        # Production build (outputs to public/vendor/laravel-crm/)

# PHP code style
composer format      # Run Laravel Pint formatter
composer format-test # Dry-run formatting check

# Tests (Pest, not PHPUnit — uses testbench, SQLite in-memory)
composer test
composer test-coverage  # Generates HTML coverage report

# Key artisan commands (run from a host app)
php artisan laravelcrm:install              # Initial setup
php artisan laravelcrm:permissions          # Seed roles & permissions
php artisan laravelcrm:sample-data          # Generate dev sample data
php artisan laravelcrm:add-user             # Add a user with CRM access
php artisan laravelcrm:encrypt              # Encrypt sensitive DB fields
php artisan laravelcrm:decrypt              # Decrypt DB fields
php artisan laravelcrm:address-types        # Seed address types
php artisan laravelcrm:contact-types        # Seed contact types
php artisan laravelcrm:organization-types   # Seed organization types
php artisan laravelcrm:fields               # Manage custom fields
php artisan laravelcrm:labels               # Manage labels
php artisan laravelcrm:email-campaigns      # Dispatch email campaigns (scheduled)
php artisan laravelcrm:sms-campaigns        # Dispatch SMS campaigns (scheduled)
php artisan laravelcrm:reminders            # Send reminder notifications (scheduled)
php artisan laravelcrm:update               # Check for/apply package updates
php artisan laravelcrm:archive              # Archive old records
php artisan laravelcrm:v2                   # V1→V2 migration helper
php artisan laravelcrm:xero                 # Xero integration utilities
```

## Architecture

### Layered Stack

```
Routes (src/Http/routes.php)
  → Controllers (src/Http/Controllers/)  ← return views/redirects only, no business logic
    → Views (resources/views/)           ← load Livewire components
      → Livewire (src/Livewire/)         ← all UI state and form logic
        → Services (src/Services/)       ← create/update orchestration
          → Models (src/Models/)         ← Eloquent + Observers
```

Repositories (`src/Repositories/`) are thin wrappers — put business logic in Services.

### Two Livewire Namespaces

- **`src/Livewire/`** — current components, registered with `crm-` prefix (e.g. `crm-lead-index`)
- **`src/Http/Livewire/`** — legacy sub-components, registered with short names (e.g. `live-lead-form`)

All components are manually registered in `LaravelCrmServiceProvider` (not auto-discovered). When adding a new Livewire component, register it there.

### Key Source Directories

| Directory | Purpose |
|---|---|
| `src/Models/` | 75+ Eloquent models, all extend `src/Models/Model.php` |
| `src/Services/` | Business logic for create/update (LeadService, DealService, etc.) |
| `src/Observers/` | 58 observers — generate IDs, trigger audit events |
| `src/Policies/` | 41 authorization policies, one per model |
| `src/Console/` | 18 artisan commands |
| `src/Livewire/` | Current Livewire components by domain |
| `src/Http/Livewire/` | Legacy inline/sub-components |
| `src/Traits/` | Shared traits for models and Livewire components |
| `src/View/Components/` | Blade components registered with `crm-` prefix |
| `src/Http/Helpers/` | Global helpers autoloaded via composer (SelectOptions, PersonName, etc.) |
| `src/Http/Middleware/` | 10 custom middleware classes |
| `src/Events/` | Broadcast events (e.g. `ChatMessageSent`) |
| `src/Listeners/` | Event listeners (e.g. `NewAuthDevice` for 2FA) |
| `src/Jobs/` | Queued jobs for campaign dispatch (email, SMS) |
| `src/Mail/` | Mailable classes (SendInvoice, SendQuote, SendPurchaseOrder, EmailCampaignMessage) |
| `src/Notifications/` | Reminder notifications (call, task, meeting, lunch reminders) |
| `src/Scopes/` | Query scopes (`BelongsToTeamsScope`) |
| `resources/views/livewire/` | Livewire component views |
| `resources/lang/` | i18n strings under `laravel-crm::` namespace |
| `database/migrations/` | Migration `.stub` files published to host app during install |

## Model Conventions

| Convention | Detail |
|---|---|
| **Base model** | All models extend `src/Models/Model.php`, which implements `OwenIt\Auditing\Auditable` and defines `saveQuietly()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [venturedrake/laravel-crm](https://github.com/venturedrake/laravel-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
