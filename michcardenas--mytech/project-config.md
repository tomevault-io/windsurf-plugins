---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel 12 e-commerce application with features including:
- Shopping cart functionality (anayarojo/shoppingcart package)
- User roles and permissions (spatie/laravel-permission)
- Square payment integration (Square Connect & Square APIs)
- Full-stack development with Laravel Breeze authentication
- Asset compilation with Vite and TailwindCSS

## Development Commands

### Starting Development Environment
```bash
composer run dev
```
This runs a comprehensive development environment with:
- PHP development server (`php artisan serve`)
- Queue worker (`php artisan queue:listen`)
- Log monitoring (`php artisan pail`)
- Vite dev server for frontend assets (`npm run dev`)

### Individual Services
```bash
# Backend only
php artisan serve

# Frontend assets (development)
npm run dev

# Frontend assets (production build)
npm run build

# Queue worker
php artisan queue:listen --tries=1

# Real-time logs
php artisan pail --timeout=0
```

### Testing
```bash
# Run all tests
composer run test
# Or directly:
php artisan test

# Clear config before testing
php artisan config:clear
```

### Code Quality
```bash
# Laravel Pint (code formatting)
./vendor/bin/pint

# PHPUnit tests
./vendor/bin/phpunit
```

## Architecture

### Backend Structure
- **Models**: Located in `app/Models/` - Eloquent models for database entities
- **Controllers**: Located in `app/Http/Controllers/` - HTTP request handling
- **Routes**:
  - `routes/web.php` - Web routes (heavily customized with 200+ lines of routes)
  - `routes/auth.php` - Authentication routes (Laravel Breeze)
- **Database**: Uses migrations in `database/migrations/`
- **Permissions**: Implements role-based permissions via Spatie package

### Frontend Structure
- **Views**: Blade templates in `resources/views/`
- **Assets**:
  - CSS in `resources/css/`
  - JavaScript in `resources/js/`
- **Build Tool**: Vite with Laravel plugin
- **Styling**: TailwindCSS with forms plugin and Alpine.js

### Key Dependencies
- **E-commerce**: anayarojo/shoppingcart for cart functionality
- **Payments**: Square Connect API integration
- **Authentication**: Laravel Breeze
- **Permissions**: Spatie Laravel Permission
- **Frontend**: Alpine.js, TailwindCSS, Vite

### Configuration
- Uses `.env` file for environment configuration
- TailwindCSS configuration in `tailwind.config.js`
- Vite configuration in `vite.config.js`
- Editor configuration in `.editorconfig`

## Database

The application uses SQLite by default (based on composer.json post-create scripts). Database file location: `database/database.sqlite`

### Migrations
```bash
# Run migrations
php artisan migrate

# Fresh migration with seeders
php artisan migrate:fresh --seed
```

===

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.2.12
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- laravel/breeze (BREEZE) - v2
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- phpunit/phpunit (PHPUNIT) - v11
- alpinejs (ALPINEJS) - v3
- tailwindcss (TAILWINDCSS) - v3


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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michcardenas/mytech](https://github.com/michcardenas/mytech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
