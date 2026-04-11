---
trigger: always_on
description: Cursor Rules for Laravel 12 Blog API Project for writing codes
---


# Cursor Rules for Laravel 12 Blog API Project

> **Note:** This document contains project-specific rules and patterns. General Laravel development standards are covered in `development-guide-ai.mdc`. Both documents should be followed together.

## Project Overview

This is a modern Laravel 12 Blog API built with PHP 8.4, featuring clean architecture, comprehensive testing, Docker-based development environment, and advanced code quality tools. The API serves as a production-ready backend for a blog platform with authentication, role-based permissions, content management, and automated quality assurance.

## Technology Stack

- **Laravel Framework**: 12.0+ (latest features)
- **PHP**: 8.2+ (with strict typing enabled, targeting 8.4+ features)
- **Database**: MySQL 8.0 (development) + MySQL 8.0 (testing - isolated environment)
- **Cache/Session**: Redis (development) + Redis (testing - isolated environment)
- **Authentication**: Laravel Sanctum (API tokens with abilities)
- **Testing**: Pest PHP 3.8+ (BDD-style testing framework)
- **Static Analysis**: Larastan 3.0+ (PHPStan level 10 for Laravel)
- **Code Formatting**: Laravel Pint (PHP-CS-Fixer preset)
- **API Documentation**: Scramble (OpenAPI/Swagger automatic generation)
- **Containerization**: Docker & Docker Compose (multi-service architecture)
- **Quality Analysis**: SonarQube integration (optional)
- **Git Tools**: Husky hooks, semantic commits, automated validation

## PHP Coding Standards (MANDATORY)

> **Note:** General PHP and Laravel standards are covered in `development-guide-ai.mdc`. This section covers project-specific requirements and additions.

### File Structure Requirements

- **ALWAYS** use `declare(strict_types=1);` at the top of all PHP files after the `<?php` starting tag
- Follow **PSR-12** coding standards strictly
- Use **descriptive, meaningful** names for variables, functions, classes, and files
- Include **comprehensive PHPDoc** for classes, methods, and complex logic
- Prefer **typed properties**, **typed function parameters**, and **typed return types**
- Break code into small, single-responsibility functions or classes
- Avoid magic numbers and hard-coded strings; use **constants**, **config files**, or **Enums**
- Use strict type declarations throughout

### PHP 8.4 Best Practices

- Use **readonly properties** to enforce immutability where appropriate
- Leverage **Enums** for clear, type-safe constants
- Use **First-class callable syntax** for cleaner callbacks
- Utilize **Constructor Property Promotion**
- Use **Union Types**, **Intersection Types**, **true/false return types**, and **Static Return Types**
- Apply the **Nullsafe Operator (?->)** for safe method/property access
- Use **Named Arguments** for clarity when calling functions with multiple parameters
- Prefer **final classes** for utility or domain-specific classes that shouldn't be extended
- Adopt **new `Override` attribute** (PHP 8.4) to explicitly mark overridden methods
- Use **dynamic class constants in Enums** where version-specific behavior is needed

### PHP 8.4 Override Attribute Example

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

final class YourModel extends Model
{
    #[Override]
    protected function casts(): array
    {
        return [
            'created_at' => 'datetime',
            'updated_at' => 'datetime',
        ];
    }
}
```

## Laravel 12 Project Structure & Conventions

### Directory Structure

```
app/
├── Actions/              # Single-responsibility action classes (create as needed)
├── Console/              # Artisan commands (create as needed)
├── Data/                 # Data Transfer Objects (DTOs) (create as needed)
├── Enums/                # Enums for type-safe constants ✅
├── Events/               # Domain events (create as needed)
├── Exceptions/           # Custom exceptions (create as needed)
├── Http/
│   ├── Controllers/      # Thin controllers ✅
│   ├── Middleware/       # HTTP middleware ✅
│   ├── Requests/         # Form Request validation ✅
│   ├── Resources/        # API Resource responses ✅
├── Jobs/                 # Queued jobs (create as needed)
├── Listeners/            # Event listeners (create as needed)
├── Models/               # Eloquent models ✅
├── Policies/             # Authorization policies ✅
├── Providers/            # Service providers ✅
├── Services/             # Business logic ✅
├── Support/              # Helpers & utility classes (create as needed)
└── Rules/                # Custom validation rules (create as needed)
```

### Domain Models

The application follows a blog-centric domain model with the following entities:

#### Core Entities

- **User**: Blog users with role-based permissions
- **Article**: Blog posts with status management
- **Category**: Hierarchical content organization
- **Tag**: Flexible content labeling
- **Comment**: User interactions on articles

#### Supporting Entities

- **Role**: User access levels (Administrator, Editor, Author, Contributor, Subscriber)
- **Permission**: Granular access control
- **Notification**: System-wide messaging
- **NewsletterSubscriber**: Email subscription management

### Enums (PHP 8.1+ Features)

All status and type fields use PHP enums for type safety:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mubbi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
