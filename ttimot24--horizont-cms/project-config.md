---
trigger: always_on
description: This file provides guidance for AI assistants working on the HorizontCMS codebase.
---

# AGENTS.md

This file provides guidance for AI assistants working on the HorizontCMS codebase.

## Overview

HorizontCMS is an open-source CMS built on:

- **PHP 8.2+ / Laravel 11** (with a custom `App\HorizontCMS` Application class, not the stock Laravel)
- **Vue 2.6** (Options API, TS components under `resources/vue/ts`)
- **Bootstrap 5.3**, jQuery, CKEditor 4, Select2, SortableJS
- Modular structure: **themes** (`themes/`) and **plugins** (`plugins/`)

## Commands

| Command                                                                                                     | Purpose                                                |
| ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| `composer test`                                                                                             | PHPUnit Unit + Integration (Gui suite is NOT included) |
| `./vendor/bin/phpunit --testsuite Full`                                                                     | all PHP tests (unit, integration, gui)                 |
| `composer lint`                                                                                             | Pint code style check (`pint --test`)                  |
| `./vendor/bin/pint`                                                                                         | Pint formatting                                        |
| `npm run lint`                                                                                              | ESLint on `resources/vue/ts`                           |
| `npm test`                                                                                                  | Vitest (Vue component tests, with coverage)            |
| `npm run prod` / `npm run watch`                                                                            | Laravel Mix build (frontend)                           |
| `php artisan hcms:install`                                                                                  | CLI installer                                          |
| `php artisan hcms:plugin {--install\|--uninstall\|--activate\|--deactivate\|--download\|--remove} {plugin}` | plugin management                                      |
| `php artisan hcms:theme {--set} {theme}`                                                                    | switch theme                                           |
| `php artisan hcms:user {--create-admin}`                                                                    | create an admin user                                   |

PHPStan (`larastan`, vendor/bin/phpstan) and PhpArchitect (`phparkitect.php`) are available as dev dependencies.

## CI (what GitHub Actions actually runs)

`.github/workflows/github-ci.yml` runs on `master` push + PRs. When in doubt, verify against it:

- **Style**: `vendor/bin/pint --test app bootstrap config database routes` (a subset — `plugins/` and `themes/` are skipped) + `phplint` for syntax.
- **Static analysis**: `vendor/bin/phpstan analyse app bootstrap config database routes --level 0` (level 0, not the larastan default).
- **PHP tests**: `INSTALLED=YES XDEBUG_MODE=coverage ./vendor/bin/phpunit --testsuite Unit` across PHP 8.1–8.5. `INSTALLED=YES` is required because CI has no `.env`.
- **Frontend**: `npm install --force` (peer-dep conflicts under Vue 2.6 / Laravel Mix), then `npm test` and `npm run build`.

## Directory structure

- `app/Controllers/` — admin + website controllers. **Important:** `App\Controllers` namespace, NOT `App\Http\Controllers`.
- `app/Model/` — Eloquent models (Blogpost, Page, Plugin, Settings, User, UserRole, HeaderImage, ScheduledTask, Visits...). Traits: `Draftable`, `HasAuthor`, `HasImage`, `IsActive`, `PaginateSortAndFilter`.
- `app/Services/` — ThemeEngine (`hcms`), BladeThemeEngine, SpaThemeEngine, Theme, Website, ShortCode, SearchEngine.
- `app/Helpers/` — global helpers (classmap autoload): `Security`, `Html`, `UrlManager`, `SocialLink`, `Functions/link.php`. Legacy helpers like `str_slug`, `studly_case` live here.
- `app/Providers/` — module loader / plugin / theme ServiceProviders.
- `app/Http/Middleware/` — web, admin, website, plugin middleware.
- `app/Console/Commands/` — `hcms:*` artisan commands.
- `routes/` — `web.php` (frontend), `backend.php` (admin), `api.php` (api/v1), `console.php`.
- `config/horizontcms.php` — central config: module dirs, theme engines, languages, `backend_prefix` (default `admin`).
- `resources/vue/ts/` — Vue components (`.vue` + `.ts` + `.spec.ts` trio), `model/`, `environments/`.
- `resources/views/` — admin Blade views.
- `resources/tests/` — **tests live here, not in `tests/`.** Namespace: `Tests\`. Suites: unit, integration, gui.
- `plugins/` — installed plugins.

## Routing (important!)

The CMS uses **legacy, URL-based "magic routing"**:

1. Admin (`routes/backend.php`, prefix `admin`): the `/{controller}/{action}/{args}` catch-all resolves the `App\Controllers\<Controller>Controller` class via `App\Http\RouteResolver` and calls the `{action}` method. The RouteResolver is `@deprecated` but still central.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ttimot24/horizont-cms](https://github.com/ttimot24/horizont-cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
