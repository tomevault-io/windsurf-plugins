---
trigger: always_on
description: This is a Laravel 12.x project with Filament admin panel integration. The project uses:
---

# AI Coding Instructions for matthew Project

## Project Overview
This is a Laravel 12.x project with Filament admin panel integration. The project uses:
- PHP 8.2+
- Laravel Sanctum for API authentication
- Spatie Media Library for file handling
- Spatie Permissions for role/permission management

## Key Development Workflows

### Local Development
```bash
# Start all development servers and watchers
composer dev
```
This runs:
- Laravel development server
- Queue worker for background jobs
- Pail log viewer
- Vite asset compilation

### Testing
```bash
composer test
```
This clears config cache and runs PHPUnit tests.

## Database Configuration
- Uses SQLite by default (`database/database.sqlite`)
- Migration table tracks version history with timestamps
- See `config/database.php` for supported drivers (MySQL, PostgreSQL, SQLite)

## Project Structure Conventions
- Models in `app/Models/`
- Controllers in `app/Http/Controllers/` 
- Database migrations in `database/migrations/`
- Tests split between `tests/Feature/` and `tests/Unit/`

## Authentication & Authorization
- Uses Laravel Sanctum for API token authentication
- Role/permission management via Spatie Permissions package
- Auth config in `config/auth.php`

## Asset Management
- Uses Vite for asset bundling
- CSS/JS source files in `resources/`
- Compiled assets served from `public/`

## Queue System
- Queue configuration in `config/queue.php`
- Job classes stored in `app/Jobs/`
- Uses database queue driver by default

## Debugging Tools
- Laravel Pail for enhanced log viewing
- Standard Laravel debugging via `APP_DEBUG=true`
- PHPUnit for testing

## Common Tasks
- Create model with migration: `php artisan make:model Name -m`
- Create controller: `php artisan make:controller NameController`
- Run migrations: `php artisan migrate`
- Create seeder: `php artisan make:seeder NameSeeder`
- Clear cache: `php artisan cache:clear`

## Admin Panel
- Built with Filament v4
- Admin interface files in `app/Filament/`
- Configuration in `config/filament.php`

## Naming Conventions
- Database columns use snake_case
- Models use PascalCase singular form
- Controllers use PascalCase and end in "Controller"
- Migration files prefixed with timestamp
- Test classes end in "Test"

===

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.3.12
- filament/filament (FILAMENT) - v4
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- livewire/livewire (LIVEWIRE) - v3
- laravel/pint (PINT) - v1


## Conventions
- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts
- Do not create verification scripts or tinker when tests cover that functionality and prove it works. Unit and feature tests are more important.

## Application Structure & Architecture
- Stick to existing directory structure - don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling
- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `npm run build`, `npm run dev`, or `composer run dev`. Ask them.

## Replies
- Be concise in your explanations - focus on what's important rather than explaining obvious details.

## Documentation Files
- You must only create documentation files if explicitly requested by the user.


=== boost rules ===

## Laravel Boost
- Laravel Boost is an MCP server that comes with powerful tools designed specifically for this application. Use them.

## Artisan
- Use the `list-artisan-commands` tool when you need to call an Artisan command to double check the available parameters.

## URLs
- Whenever you share a project URL with the user you should use the `get-absolute-url` tool to ensure you're using the correct scheme, domain / IP, and port.

## Tinker / Debugging
- You should use the `tinker` tool when you need to execute PHP to debug code or query Eloquent models directly.
- Use the `database-query` tool when you only need to read from the database.

## Reading Browser Logs With the `browser-logs` Tool
- You can read browser logs, errors, and exceptions using the `browser-logs` tool from Boost.
- Only recent browser logs will be useful - ignore old logs.

## Searching Documentation (Critically Important)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nettsite) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
