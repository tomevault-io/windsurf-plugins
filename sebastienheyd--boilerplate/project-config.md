---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Laravel Boilerplate package** based on AdminLTE 3, designed for creating back-office applications with user management, roles, permissions, and dashboard widgets. The package is compatible with Laravel 11-13 and distributed via Composer with namespace `Sebastienheyd\Boilerplate`.

## Development Commands

### Testing and Quality
- `make test` - Run PHPUnit tests with stop-on-failure
- `make testcoverage` - Run tests with code coverage (requires Xdebug)
- `make testcoveragehtml` - Generate HTML coverage report
- `make cs` - Check coding standards with PHP_CodeSniffer
- `make csfix` - Fix coding standards automatically
- `make clean` - Clean test artifacts and vendor files

### Composer Scripts
- `composer test` - Run PHPUnit tests
- `composer lint` - Run PHPStan static analysis
- `composer serve` - Start development server with Orchestra Testbench
- `composer build` - Build workbench for testing

### Asset Compilation
- `npm run docs:dev` - Start VuePress documentation development server
- `npm run docs:build` - Build documentation for production
- Frontend assets are compiled via Laravel Mix (see `src/webpack.mix.js`)

## Architecture Overview

### Service Provider Pattern
The main `BoilerplateServiceProvider` handles:
- Configuration merging for 7 config files (`app`, `auth`, `dashboard`, `laratrust`, `locale`, `menu`, `theme`)
- Package registration (Laratrust, Laravel Menu, DataTables)
- Custom middleware registration
- View composers for automatic data injection
- Repository singletons for Menu, Navbar, Datatables, and Dashboard Widgets

### Key Architectural Patterns
- **Repository Pattern**: Centralized registration for Menu items, DataTables, Dashboard widgets, Navbar items
- **Factory Pattern**: Dynamic generation of menus, widgets, and datatables
- **Observer Pattern**: Events for user actions (`UserCreated`, `UserSaved`, `UserDeleted`)
- **Command Pattern**: Console commands for code generation

### Code Generation System
Console commands for scaffolding:
- `boilerplate:datatable` - Generate DataTable classes
- `boilerplate:dashboard` - Create dashboard controllers
- `boilerplate:menu` - Add menu items
- `boilerplate:permission` - Manage permissions
- `boilerplate:scaffold` - Complete CRUD scaffolding
- `boilerplate:widget` - Dashboard widgets

Templates are stored in `src/Console/stubs/` with `.stub` extension.

### Asset Pipeline
Laravel Mix configuration handles:
- **SCSS compilation**: AdminLTE, Bootstrap 4, component-specific styles
- **JavaScript bundling**: AdminLTE, DataTables, TinyMCE, FullCalendar, and custom scripts
- **Plugin integration**: FontAwesome, Moment.js, Select2, DatePickers, ColorPicker
- **Multi-language support**: Asset localization for multiple languages

### Testing Architecture
- **Orchestra Testbench**: Laravel package testing environment
- **Test Structure**: Organized by functionality (Components, Console, Controllers, Dashboard)
- **Database**: SQLite in-memory for testing
- **No Mocks Policy**: Prefers integration tests over mocked dependencies
- **Test Isolation**: Each test suite runs independently

### Security & Permissions
- **Laratrust Integration**: Complete role-based access control
- **Custom Models**: `User`, `Role`, `Permission`, `PermissionCategory`
- **Middleware Stack**: Comprehensive security layer with custom middleware
- **Permission Categories**: Hierarchical permission organization

### Middleware Architecture
The package provides 5 custom middleware for authentication and user experience:

#### Core Middleware
- **`BoilerplateAuthenticate`**: Extends Laravel's Authenticate middleware, redirects unauthenticated users to `boilerplate.login` route
- **`BoilerplateGuest`**: Prevents authenticated users from accessing guest-only routes (login, register), redirects to dashboard
- **`BoilerplateEmailVerified`**: Configurable email verification (via `boilerplate.auth.verify_email` config), extends Laravel's EnsureEmailIsVerified
- **`BoilerplateLocale`**: Automatic locale management with Carbon integration, supports user preferences and cookie storage
- **`BoilerplateImpersonate`**: Admin user impersonation system with session-based state management

#### Middleware Registration
The ServiceProvider registers middleware aliases for both legacy and dotted notation:
```php
// Legacy aliases (backward compatibility)
$this->router->aliasMiddleware('boilerplateauth', BoilerplateAuthenticate::class);
$this->router->aliasMiddleware('boilerplateguest', BoilerplateGuest::class);
$this->router->aliasMiddleware('boilerplatelocale', BoilerplateLocale::class);

// Modern dotted aliases (preferred)
$this->router->aliasMiddleware('boilerplate.auth', BoilerplateAuthenticate::class);
$this->router->aliasMiddleware('boilerplate.guest', BoilerplateGuest::class);
$this->router->aliasMiddleware('boilerplate.locale', BoilerplateLocale::class);
$this->router->aliasMiddleware('boilerplate.emailverified', BoilerplateEmailVerified::class);

// Conditional registration
if (config('boilerplate.app.allowImpersonate', false)) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebastienheyd/boilerplate](https://github.com/sebastienheyd/boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
