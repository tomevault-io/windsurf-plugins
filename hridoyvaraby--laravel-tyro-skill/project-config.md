---
trigger: always_on
description: Complete knowledge and operation guide for Tyro - the Laravel authentication, authorization, and role-based access control package. Use this skill when working with Laravel projects that use Tyro, when setting up authentication/authorization, managing roles/privileges, using Tyro commands, middleware, Blade directives, or REST API endpoints. Trigger for any mention of Tyro, Laravel auth, RBAC, role management, privilege management, user suspension, or Laravel Sanctum integration with permissions
---


# Laravel Tyro Skill

This skill provides complete knowledge of **Tyro** - a comprehensive authentication, authorization, and role-based access control (RBAC) solution for Laravel 12 and 13.

## When to Use This Skill

Use this skill when:
- Setting up Tyro in a Laravel project
- Managing users, roles, or privileges
- Using Tyro's CLI commands (40+ commands)
- Implementing route protection with middleware
- Using Blade directives for permission checks
- Working with Tyro's REST API
- Troubleshooting Tyro-related issues
- Auditing user actions and role changes

---

## Quick Overview

**Tyro** provides:
- Complete authentication with Laravel Sanctum
- Role-based access control (RBAC)
- Fine-grained privilege management
- User suspension workflows
- 40+ Artisan commands
- 7 Blade directives
- REST API endpoints
- Comprehensive audit trail
- Two-tier caching system

**Default seeded roles:**
- `super-admin` - Full system access
- `administrator` - Administrative access
- `editor` - Content management
- `user` - Standard registered user
- `customer` - Customer account
- `all` - Universal access (wildcard)

**Default credentials (after seeding):**
- Email: `admin@tyro.project`
- Password: `tyro`

---

## Installation & Setup

### One-Command Installation

```bash
composer require hasinhayder/tyro
php artisan tyro:sys-install
```

This single command:
1. Installs and configures Sanctum
2. Runs migrations
3. Prompts to seed roles/privileges/admin user
4. Prepares your User model

### Manual Installation Steps

If you need more control:

```bash
# Install package
composer require hasinhayder/tyro

# Publish assets (optional)
php artisan vendor:publish --tag=tyro-config
php artisan vendor:publish --tag=tyro-migrations

# Run migrations
php artisan migrate

# Seed roles and privileges
php artisan tyro:seed-all --force

# Prepare User model
php artisan tyro:user-prepare
```

### User Model Requirements

Your User model (default: `App\Models\User`) must use the `HasTyroRoles` trait:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Sanctum\HasApiTokens;
use HasinHayder\Tyro\Concerns\HasTyroRoles;

class User extends Authenticatable
{
    use HasApiTokens, HasTyroRoles;
}
```

The trait provides ALL role/privilege methods. No other code changes needed.

---

## CLI Commands Reference

### System Commands

```bash
# Full installation
php artisan tyro:sys-install

# Prepare User model with trait
php artisan tyro:user-prepare

# Show version
php artisan tyro:sys-version

# Show package info
php artisan tyro:sys-about

# Open documentation
php artisan tyro:sys-doc

# Open GitHub repo
php artisan tyro:sys-star

# Get Postman collection
php artisan tyro:sys-postman
```

### User Management

```bash
# Create user with default role
php artisan tyro:user-create

# List all users
php artisan tyro:user-list

# List users with roles
php artisan tyro:user-list-with-roles

# Update user
php artisan tyro:user-update --user=1 --name="New Name"

# Delete user (prevents deleting last admin)
php artisan tyro:user-delete --user=1

# Suspend user (revokes all tokens)
php artisan tyro:user-suspend --user=admin@example.com --reason="Policy violation"

# Unsuspend user
php artisan tyro:user-unsuspend --user=admin@example.com

# List suspended users
php artisan tyro:user-suspended

# Show user's roles and privileges
php artisan tyro:user-roles --user=1
```

### Role Management

```bash
# List all roles
php artisan tyro:role-list

# List roles with privileges
php artisan tyro:role-list-with-privileges

# Create new role
php artisan tyro:role-create --name="Content Manager" --slug="content-manager"

# Update role
php artisan tyro:role-update --role="content-manager" --name="Content Editor"

# Delete role (protected roles cannot be deleted)
php artisan tyro:role-delete --role="content-manager"

# Assign role to user
php artisan tyro:role-assign --user=5 --role="editor"

# Remove role from user
php artisan tyro:role-remove --user=5 --role="editor"

# List users with specific role
php artisan tyro:role-users --role="editor"
```

### Privilege Management

```bash
# List all privileges
php artisan tyro:privilege-list

# Create privilege
php artisan tyro:privilege-create --name="Delete Articles" --slug="articles.delete"

# Update privilege
php artisan tyro:privilege-update --privilege="articles.delete" --name="Remove Articles"

# Delete privilege
php artisan tyro:privilege-delete --privilege="articles.delete"

# Attach privilege to role
php artisan tyro:privilege-attach --role="editor" --privilege="articles.publish"

# Detach privilege from role
php artisan tyro:privilege-detach --role="editor" --privilege="articles.publish"

# Show user's privileges
php artisan tyro:user-privileges --user=1
```

### Authentication & Token Management

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HridoyVaraby/laravel-tyro-skill](https://github.com/HridoyVaraby/laravel-tyro-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
