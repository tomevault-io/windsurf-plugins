---
trigger: always_on
description: Bulla is a self-hosted comment system built with Laravel, designed to replace isso.
---

# Bulla - Development Guidelines

## Project Overview

Bulla is a self-hosted comment system built with Laravel, designed to replace isso.

## Tech Stack

- **Backend**: Laravel 12, PHP 8.3+, Laravel Actions
- **Database**: SQLite (default) or PostgreSQL
- **Admin Panel**: React + TypeScript + Inertia.js v2 + Mantine UI v8
- **Embed Widget**: TypeScript + Preact + Mantine (via preact/compat), <25KB
- **Deployment**: Docker with FrankenPHP

## Architecture Patterns

### Laravel Actions
Use `lorisleiva/laravel-actions` for all business logic:
```php
class CreateComment
{
    use AsAction;

    public function handle(Thread $thread, array $data): Comment
    {
        // Business logic here
    }

    public function asController(Request $request, string $uri): JsonResponse
    {
        // HTTP handling here
    }
}
```

### Database Driver Pattern
For features that differ between SQLite and PostgreSQL (like full-text search), use the driver pattern:
```php
$driver = match (DB::connection()->getDriverName()) {
    'pgsql' => new PostgresSearchDriver,
    default => new SqliteSearchDriver,
};
```

### Single Tenant
This is a single-tenant application. No multi-site support, one admin user only.

## Code Style

### PHP
- Use `declare(strict_types=1);` in all PHP files
- Use constructor property promotion
- Use typed properties and return types
- Follow PSR-12 coding style (enforced by Pint)
- Use PHPDoc for complex types

### TypeScript/React (Admin Panel)
- Use functional components with hooks
- Use Mantine components for UI
- Use Inertia's `useForm` for form handling
- Use Inertia's `Link` component for navigation
- Linting enforced by Biome

## Testing

Run tests for both database drivers:
```bash
# SQLite (default)
php artisan test

# PostgreSQL
php artisan test --configuration=phpunit.pgsql.xml
```

## Key Directories

- `app/Actions/` - Business logic (Laravel Actions)
- `app/Models/` - Eloquent models
- `app/Search/` - Database-specific search drivers
- `embed/` - Embed widget source code (separate build)
- `resources/js/` - Admin panel React components

## Frontend Builds

### Admin Panel (Inertia + Mantine)
Located in `resources/js/`
Build: `npm run build`
- Full React app with Mantine UI
- Size not critical (admin only)

### Embed Widget (Preact + Mantine)
Located in `embed/`
Build: `cd embed && npm run build`
Output: `public/embed.js`
- Target size: **<25KB gzipped**
- Uses Preact with `preact/compat` for Mantine compatibility
- Separate Vite config optimized for size

## Database

Default: SQLite at `database/database.sqlite`
Switch to PostgreSQL via `DB_CONNECTION=pgsql` in `.env`

## Security Considerations

- All comment content is sanitized
- IPs are anonymized (/24 IPv4, /48 IPv6)
- CORS validation for embed requests
- CSRF protection for admin actions
- Rate limiting on comment creation

===

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.5.1
- inertiajs/inertia-laravel (INERTIA) - v2
- laravel/framework (LARAVEL) - v12
- laravel/mcp (MCP) - v0
- laravel/prompts (PROMPTS) - v0
- larastan/larastan (LARASTAN) - v3
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- @inertiajs/react (INERTIA) - v2
- react (REACT) - v19
- tailwindcss (TAILWINDCSS) - v4

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [angristan/bulla](https://github.com/angristan/bulla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
