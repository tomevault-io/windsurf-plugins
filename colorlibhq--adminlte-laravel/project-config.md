---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AdminLTE 4 for Laravel** (`colorlibhq/adminlte-laravel`) is the official Laravel package integration of AdminLTE 4 (Bootstrap 5.3, vanilla JS, no jQuery). It provides:

- Config-driven sidebar menu with permissions, active states, badges
- 40 Blade components (widgets, forms, charts, calendar, kanban, wizard, modals)
- Scaffolding system (`adminlte:scaffold`) for 18 DB-backed app sections
- Dependency-free RBAC (roles, permissions, middleware, permission-aware Gate)
- Auth scaffolding (`adminlte:make-auth`) for plain/Breeze/Fortify
- Plugin system for lazy-loading JS libraries (Flatpickr, Tom Select, Tabulator, Quill, ApexCharts, jsVectorMap, FullCalendar, SortableJS)
- i18n with 9 locales (en, de, es, fr, it, ja, pt_BR, ru, zh), RTL support
- Bundled demo pages and in-app docs (served at `/docs`)
- Vite-first asset pipeline

This is a **Composer package**, not a Laravel application. Requires PHP ^8.3 and Laravel (illuminate) ^13.0. Tests run against Orchestra Testbench — the package is never "run" directly; it's consumed via `composer require colorlibhq/adminlte-laravel` + `php artisan adminlte:install`.

## Development Commands

```bash
composer test       # PHPUnit
composer lint       # Pint check only (what CI runs)
composer fix        # Pint, applying fixes
composer analyse    # PHPStan level 8 (bakes in --memory-limit=1G — the bare
                    # `vendor/bin/phpstan` exhausts PHP's default 128M limit)
composer check      # lint + analyse + test (the full CI sequence)

# Run a single test class/method
vendor/bin/phpunit tests/SmokeTest.php
vendor/bin/phpunit tests/SmokeTest.php --filter testComponentsRenderWithoutErrors
```

CI (`.github/workflows/tests.yml`) runs on PHP 8.3 + 8.4 + 8.5 against Laravel 13 / Testbench 11, in this order: **composer lint → composer analyse → composer test**. All three must pass.

## Architecture & Key Concepts

### Service Provider (`AdminLteServiceProvider`)

Single entry point. On boot it:

- Registers 40 Blade components under the `adminlte-` prefix (e.g. `<x-adminlte-card>`), from the `$components` map at the top of the class
- Registers the `AdminLte` menu-builder singleton (aliased `adminlte`) and the `PluginManager` singleton
- Registers `@pluginStyles` / `@pluginScripts` Blade directives — these emit PHP that runs at **request time**, so plugins enabled by components during render are reflected
- Registers translations both under the `adminlte::` namespace **and** as a default-namespace path, so views can use plain `__('adminlte.key')` without publishing
- Registers demo routes (`/demo/*`, toggle with `config('adminlte.demo')`) and in-app docs routes (`/docs/{page}`, renders the markdown in `docs/` with the AdminLTE layout)
- Opportunistically wires RBAC for the consuming app: model policies, `role`/`permission` middleware aliases, and a permission-aware `Gate::before` — all guarded by `class_exists` on `App\...` classes that only exist after `adminlte:scaffold rbac`
- Listens to auth events (Login/Logout/Failed) and writes them via `ActivityLogger`, which **no-ops when the `activity_log` table is absent**

### Artisan Commands (`src/Console/`)

| Command | Purpose |
| --- | --- |
| `adminlte:install` | Publish config + Vite stubs, wire Vite, prompt for npm deps (`--only=config\|views\|assets\|lang`) |
| `adminlte:scaffold {section}` | Publish a DB-backed section into the consuming app (`--all`, `--force`, `--seed`) |
| `adminlte:make-auth` | Auth controllers/routes (`--type=plain\|breeze\|fortify`) with hardening |
| `adminlte:status` | Show install state |

### Scaffolding System (`ScaffoldCommand` + `resources/stubs/`)

The largest subsystem. `ScaffoldCommand` holds a declarative `$manifest` mapping each of 18 sections (dashboard, mailbox, chat, kanban, calendar, projects, file-manager, profile, settings, invoice, pricing, faq, notifications, api, impersonation, activity-log, realtime, rbac) to what it publishes: migrations, models, factories, controllers, Form Requests, policies, seeders, feature tests, views, and route stubs. All stubs live in `resources/stubs/` as `.php.stub` files organized by type (`models/`, `controllers/`, `migrations/`, `policies/`, `rbac/`, `realtime/`, etc.). To change what a section generates, edit both the manifest and the stub files.

### Menu System (`AdminLte` + `MenuItemHelper`)

- **`AdminLte` class** (singleton): builds and filters the sidebar/navbar menu from `config('adminlte.menu')`
- Menu items flow through a **filter pipeline** (`config('adminlte.filters')`), in order:
  1. `SearchFilter` — normalizes menu data
  2. `GateFilter` — filters by authorization (`can` key; works with the RBAC `Gate::before`)
  3. `HrefFilter` — resolves routes to URLs
  4. `ActiveFilter` — marks current page as active
- Singleton ensures runtime `addAfter()` calls persist for the request (reset next request)
- Scoped retrieval: `menu('sidebar')`, `menu('navbar-left')`, `menu('navbar-right')`
- The filter pipeline is the single source of truth for menu rendering logic

### Plugin System (`PluginManager`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ColorlibHQ/adminlte-laravel](https://github.com/ColorlibHQ/adminlte-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
