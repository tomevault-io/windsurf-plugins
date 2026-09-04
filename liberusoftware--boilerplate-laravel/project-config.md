---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Shape of this repository

This is a **composition host**, not an application. `app/` holds exactly five files:

```
app/Models/User.php                            the one host model
app/Filament/ModulePlugins.php                 composes panel plugins from enabled modules
app/Support/ThemeColors.php                    maps the site theme to a Filament palette
app/Providers/Filament/AdminPanelProvider.php
app/Providers/Filament/AppPanelProvider.php
```

Everything else — search, theming, localization, settings, roles and permissions,
observability — lives in **40 `liberu-module` packages** under `modules/` and
**4 `liberu-theme` packages** under `themes/`. Both directories are Composer install
targets *and* tracked in Git (`.gitignore` negates them explicitly).

Before assuming a class lives in `app/`, search `modules/`. Almost nothing is in `app/`.

### Where a package is edited

**Each package's own repository is the source of truth, and `modules/`, `themes/` are Composer
output.** A change made directly under `modules/<name>/` survives exactly until the next
`composer update`, which fetches from the remote and overwrites it.

So a package is edited in a **clone**, at `~/code/<repo>` — cloned when needed rather than kept as a
permanent 44-repo workspace. `scripts/fleet` drives that:

```bash
scripts/fleet status                          # what is checked out, dirty, unpushed
scripts/fleet clone --only search             # ensure a repository is present
scripts/fleet run 'vendor/bin/pest'           # fan a command across all of them
scripts/fleet commit -m 'Fix the thing'       # stage, commit, push main
scripts/fleet tag 1.2.0 --only search         # explicit list required; humans tag
```

**The runner deliberately stops short of tagging.** A push is recoverable; a tag is what Packagist
publishes and what `ModuleValidator` pins the host to, so an unattended bad wave would be 44
revert-tags. `tag` also refuses a dirty worktree, a non-`main` branch, or unpushed commits.

A wave lands in the host with one commit: `composer update`, then `modules/` and `composer.lock`
together. `modules/` stays **tracked**, and CI fails on an uncommitted diff — that check is what
caught 48-of-48 divergence between the tracked tree and the published packages.

Packagist names drop the `module-` prefix that the GitHub repositories carry:
`liberusoftware/module-search` on GitHub is `liberusoftware/search` on Packagist.

> `scripts/publish-components` rsynced this monorepo *into* the package repositories, which is
> the opposite direction. It was **removed** once the coverage-ratchet wave proved a fleet-wide
> change could be released through `fleet` alone — see `docs/CONFORMANCE.md` §5, step 9.

## Commands

```bash
composer install && npm install

composer test                  # vendor/bin/pest
vendor/bin/pest tests/Feature/SearchTest.php
vendor/bin/pest --filter=SearchTest
vendor/bin/pint                # --test to check without writing
vendor/bin/phpstan analyse     # app/ only, at the level app/ passes

npm run dev                    # vite
npm run build                  # required once, to compile the theme bundles

php artisan migrate
php artisan migrate:fresh --seed

php artisan horizon
php artisan reverb:start
php artisan octane:start --server=roadrunner

php artisan filament:upgrade
php artisan shield:generate

# A single package's own suite, standalone:
cd modules/search && composer update && vendor/bin/pest
```

## Architecture

### Module system

`ModuleManagerServiceProvider` (first entry in `bootstrap/providers.php`) discovers packages
from `config('modules.paths')` and reads each `module.json` — provider class, capabilities,
required packages and capabilities, `default_enabled`, and any Filament plugins.
`ModuleRegistry::resolve($enabled, $disabled)` validates constraints with `Semver` and returns
providers in dependency order.

Installation never implies boot: no package declares `extra.laravel.providers`, so Laravel's
auto-discovery finds nothing to register, and an architecture rule asserts that stays true.
Enablement is a separate, explicit decision.

**A manifest's own `default_enabled` is what boots it** — true for 37 of the 40, false for
`analytics-google`, `analytics-meta` and `localization-mymemory`, which need third-party
credentials and so ship installed but off. `config/modules.php` names no modules at all; it holds
only `MODULES_ENABLED` and `MODULES_DISABLED`, both empty by default. `MODULES_ENABLED` adds
modules their manifests leave off, `MODULES_DISABLED` removes modules their manifests turn on, and
**disabled beats both**. Adding a module to a composition is therefore installing it; there is no
second list to remember.

Disabling a module something else requires is a `DependencyResolutionFailed`, not a quiet omission.
Two architecture rules pin all of this, including that config stays list-free.

Domain packages stay presentation-agnostic. Filament UI lives in companion `*-filament`
packages whose manifests declare `admin` and/or `app` plugin classes; `App\Filament\ModulePlugins`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liberusoftware/boilerplate-laravel](https://github.com/liberusoftware/boilerplate-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
