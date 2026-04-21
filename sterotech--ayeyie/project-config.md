---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ayeyie Poultry Feed Software is a Laravel 12 application built for Ayeyie Poultry Feed Depot. It's an integrated payment and pickup system with fraud detection, QR code receipts, and inventory management. The system uses the TALL stack (Tailwind, Alpine.js, Laravel, Livewire) with Livewire Volt and Flux UI components.

## Common Development Commands

### Development Environment
```bash
# Start development environment (includes server, queue, logs, and Vite)
composer run dev
# OR manually start individual services:
php artisan serve
php artisan queue:listen --tries=1
php artisan pail --timeout=0
npm run dev
```

### Code Quality & Testing
```bash
# Run tests
php artisan test
./vendor/bin/pest

# Code formatting with Laravel Pint
./vendor/bin/pint

# Static analysis with PHPStan/Larastan
./vendor/bin/phpstan analyse

# Generate IDE helpers
php artisan ide-helper:generate
php artisan ide-helper:models
```

### Database Management
```bash
# Run migrations
php artisan migrate

# Fresh migration with seeding
php artisan migrate:fresh --seed

# Create migration
php artisan make:migration create_table_name

# Create model with factory and migration
php artisan make:model ModelName -mf
```

### Build and Deployment
```bash
# Build assets for production
npm run build

# Clear various caches
php artisan optimize:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
```

## Architecture and Code Patterns

### Domain Organization
The application follows a domain-driven structure:
- **Admin Module**: Product, user, audit log, stock alert, and suspicious activity management
- **Customer Module**: Order creation, editing, and pickup management  
- **Core Models**: Product, User, Transaction, Receipt, Pickup, etc. with comprehensive relationships

### Livewire Component Structure
- **Full-page Components**: Located in `app/Livewire/` with corresponding Blade views in `resources/views/livewire/`
- **Volt Components**: Single-file components using Livewire Volt syntax
- **Action Classes**: Located in `app/Livewire/Actions/` for complex business logic (e.g., `CreateUser`, `UpdateUser`)
- **Form Classes**: Located in `app/Livewire/Forms/` for form handling and validation

### Model Patterns
- All models use `declare(strict_types=1)` and are marked `final`
- Custom primary keys (e.g., `product_id` instead of `id`) 
- Comprehensive PHPDoc with `@property` annotations for IDE support
- Proper relationship definitions with generic type hints
- Factory support for all models

### Database Design
- 3rd Normal Form (3NF) schema design
- Custom primary key naming convention (`product_id`, `transaction_id`, etc.)
- Comprehensive foreign key relationships
- Migration files include detailed comments explaining field purposes
- Consistent use of `created_at` and `updated_at` timestamps

### Testing Strategy
- Uses Pest testing framework with PHPUnit as the base
- Feature tests for Livewire components in `tests/Feature/Livewire/`
- Unit tests for models in `tests/Unit/Models/`
- Authentication tests in `tests/Feature/Auth/`
- Test configuration includes `RefreshDatabase` trait and Storage faking

### Code Quality Standards
- Laravel Pint with custom configuration in `pint.json`
- PHPStan at maximum level with Larastan for Laravel-specific analysis
- Strict typing enforced throughout the codebase
- Final classes and comprehensive type hints
- Ordered class elements and imports

## Key Framework Integrations

### Livewire & Volt
- Primary frontend framework using Livewire 3.x with Volt integration
- Flux UI components from Livewire for consistent design system
- URL-aware components with `#[Url]` attributes for search and filtering

### Authentication & Authorization
- Laravel Breeze-style authentication
- Policy-based authorization (though policies are currently restrictive placeholders)
- Role-based access control in User model

### Queue & Notifications
- Laravel's built-in queue system for background processing
- Notification system configured for customer SMS/email updates

## Important Development Notes

### Database Conventions
- Always use custom primary keys matching table names (e.g., `product_id` for products table)
- Check existing migration patterns before creating new tables
- Use proper foreign key relationships with descriptive constraint names

### Livewire Best Practices
- Use action classes for complex business logic instead of embedding in components
- Implement proper URL state management for filters and search
- Follow the established pattern of query builders with filtering, sorting, and pagination

### Code Style Requirements
- All new PHP files must include `declare(strict_types=1)`
- Use final classes unless inheritance is specifically needed
- Include comprehensive PHPDoc annotations
- Follow existing naming conventions for routes, views, and components

### Testing Requirements
- Write feature tests for all Livewire components
- Include model tests for relationships and business logic
- Use factories for test data generation
- Test both happy path and edge cases

## Standard Admin Page Layout Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SterotECH) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
