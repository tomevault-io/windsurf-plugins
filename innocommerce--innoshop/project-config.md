---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit Rules

- Do NOT include "Co-Authored-By: Claude" or any similar co-author attribution in commit messages.

## Project Overview

InnoShop is an open-source e-commerce system built on Laravel 12 with PHP 8.2+. It features a modular "Innopacks" architecture, plugin system, multi-language/currency support, and AI integration.

## Development Commands

### PHP/Laravel
```bash
# Run the application
php artisan serve

# Run tests
php artisan test
php artisan test --filter=TestName

# Code quality
composer pint              # Run Laravel Pint (PHP CS Fixer)
composer phpstan           # Run PHPStan on plugins and innopacks
composer phpstan:plugins   # Run PHPStan on plugins only
composer phpstan:innopacks # Run PHPStan on innopacks only

# Database
php artisan migrate
php artisan db:seed

# Cache clearing (important after route/config changes)
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
```

### Frontend Assets (Laravel Mix)
```bash
# Install dependencies
npm install

# Development builds
npm run dev          # Build once
npm run watch        # Watch for changes
npm run hot          # Hot module replacement

# Production build
npm run production

# Build specific theme
THEME=petnow npm run dev
```

## Architecture Overview

### Innopacks Modular System

The codebase is organized into modular packages under `/innopacks/`:

- **common/** - Shared models, repositories, services, and traits used across all modules
- **front/** - Frontend controllers, views, and routes for the customer-facing store
- **panel/** - Admin panel controllers, views, and routes for backend management
- **restapi/** - RESTful API endpoints (front-api.php, panel-api.php)
- **plugin/** - Plugin management system core
- **install/** - Installation wizard
- **enterprise/** - Enterprise-specific features
- **devtools/** - Development tools and utilities

### Key Architectural Patterns

**Repository Pattern**: All data access goes through repository classes in `innopacks/common/src/Repositories/`. Example: `ProductRepo::getInstance()->getFrontList($filters)`.

**Multi-language Support**: Models use the `Translatable` trait. Translation tables (e.g., `product_translations`) store localized content with locale columns.

**Database Prefix**: All tables use the `inno_` prefix (configured in `.env` as `DB_PREFIX=inno_`).

**Custom Route Groups**: The application defines custom middleware groups in `bootstrap/app.php`:
- `front` - Frontend web requests
- `panel` - Admin panel web requests
- `front_api` - Frontend API requests (with Sanctum)
- `panel_api` - Admin panel API requests (with Sanctum)

### Plugin System

Plugins are located in `/plugins/` and are auto-discovered. Each plugin:
- Has its own directory with `composer.json` (merged via wikimedia/composer-merge-plugin)
- Uses the `Plugin\` namespace
- Can define routes, views, controllers, and service providers
- Can be enabled/disabled through the admin panel

### Theme System

Themes are in `/themes/` (e.g., `petnow`, `thangka`). The build system (`webpack.mix.js`):
- Compiles theme assets from `themes/{name}/css/` and `themes/{name}/js/`
- Outputs to `public/static/themes/{name}/`
- Copies assets to theme's `public/` directory for distribution
- Set `THEME` environment variable to build a specific theme

### Code Quality Standards

- **Laravel Pint**: Uses Laravel preset with custom array alignment rules (see `pint.json`)
- **PHPStan**: Level 1 analysis for `plugins/` and `innopacks/` directories
- **Testing**: PHPUnit with separate Unit and Feature test suites
- **File Headers**: All PHP files (including test files) MUST include the standard copyright header:
```php
<?php
/**
 * Copyright (c) Since 2024 InnoShop - All Rights Reserved
 *
 * @link       https://www.innoshop.com
 * @author     InnoShop <team@innoshop.com>
 * @license    https://opensource.org/licenses/OSL-3.0 Open Software License (OSL 3.0)
 */
```
- **Test File Organization**: Test files should only contain tests relevant to their module. For example, `innopacks/install/tests/` should only contain installation-related tests.

### Helper Functions

Helper functions are defined in multiple locations and are auto-loaded via composer.json:
- `innopacks/install/helpers.php` - Installation-related helpers
- `innopacks/common/helpers.php` - Common helpers: `front_route()`, `panel_route()`, `is_admin()`, `current_locale()`
- `innopacks/panel/helpers.php` - Admin panel helpers
- `innopacks/plugin/helpers.php` - Plugin system helpers

### Model Conventions

- All models extend `InnoShop\Common\Models\BaseModel`
- Use `HasPackageFactory` trait for factory support
- Use `Translatable` trait for multi-language models
- Relationships use explicit foreign keys
- JSON/array columns use Laravel casts

### Repository Conventions

- All repositories extend `InnoShop\Common\Repositories\BaseRepo`
- Use singleton pattern: `ProductRepo::getInstance()`
- Methods return models, collections, or paginators
- Filter parsing via `RequestFilterParser` service

---

# Plugin Development Guide

## Correct Plugin Directory Structure

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [innocommerce/innoshop](https://github.com/innocommerce/innoshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
