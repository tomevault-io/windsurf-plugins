---
trigger: always_on
description: This file is for an AI agent (or human) **developing the `dmitryisaenko/larafoundry` package itself**. If you are instead *consuming* the package in a host app, read [`docs/integrating-into-an-existing-app.md`](docs/integrating-into-an-existing-app.md) and [`docs/README.md`](docs/README.md) — not this file.
---

# AGENTS.md — LaraFoundry core (dev-context for agents working ON the package)

This file is for an AI agent (or human) **developing the `dmitryisaenko/larafoundry` package itself**. If you are instead *consuming* the package in a host app, read [`docs/integrating-into-an-existing-app.md`](docs/integrating-into-an-existing-app.md) and [`docs/README.md`](docs/README.md) — not this file.

> Companion: `CLAUDE.md` (same directory) points Claude Code here.

## What this package is

A reusable SaaS/CRM **core** for Laravel, extracted phase-by-phase from a production app and modernised (with a security pass on every lifted piece). Proprietary licence. It ships as a Composer package and is consumed by host apps over a `path`/Packagist repository.

- **Composer name:** `dmitryisaenko/larafoundry` · **type:** `library`
- **Namespace:** `Dmitryisaenko\LaraFoundry\` → `src/` (PSR-4). Tests: `Dmitryisaenko\LaraFoundry\Tests\` → `tests/`.
- **Requires:** PHP `^8.2`, Laravel `^12 || ^13`, Inertia `^2 || ^3`, Fortify, Sanctum, Socialite, Ziggy, `intervention/image`, `laravolt/avatar`, `spatie/laravel-activitylog`, `bacon/bacon-qr-code`, `ezyang/htmlpurifier`.
- **Stack the core ships UI in:** Inertia + Vue 3 + Tailwind 4, vue-i18n, Pest.

## Golden rules

1. **Never edit the host.** The core exposes a small fixed set of *seams* and a host plugs into them; the core never asks a host to patch its code. Add behaviour by adding a seam, not by assuming host code. The seams: model traits, package install/migrations, provider registries (`MenuProvider`/`DashboardWidgetProvider`/export/purge registries), config registries, published pages + barrel components, and core services called from host domain (see `docs/README.md`).
2. **Contracts + bindings for anything swappable.** A capability that a host might replace lives behind a contract in `*/Contracts` with a default bound in the service provider (e.g. `MediaStorage`, `AvatarGenerator`, `GeoResolver`, `TenantResolver`, `EntitlementResolver`, `PlanRepositoryContract`, `RegionContext`). Call the contract, never the concrete.
3. **Fail-closed.** Security/tenancy defaults deny: `TenantScope` returns zero rows with no active tenant and throws on an unscoped create; the settings store and permission catalog are fail-closed to their registries; the billing access gate is fail-closed both ways. Keep new code in this spirit.
4. **Config-driven registries over hard-coded lists.** Activity-log events, settings keys, ui_settings keys, email templates, legal pages, OAuth providers, plans — all declared in a published `config/larafoundry*.php` and validated against that registry. Arbitrary keys are rejected.
5. **HTML that reaches mail or a page goes through `HtmlSanitizer`** (the email-template + legal-page editors render super-admin HTML; never let stored content execute).
6. **i18n keys are the English source text.** Frontend strings use `{{ $t('English text') }}`; the bundled `lang/frontend/{locale}.json` translates them. No separate key namespace.
7. **Semver.** Public API (trait method names, config keys, published page contracts, shared-prop shapes, route names) is a contract — break it only with a major bump. Each shipped phase gets a tag.

## Repository layout

```
src/                 # one folder per domain module (PSR-4 Dmitryisaenko\LaraFoundry\)
  ActivityLog/  Auth/ (+ Auth/Qr/)  Authorization/  Billing/  Console/  Contracts/
  Dashboard/  Email/  Http/ (cross-cutting middleware)  Legal/  Media/  Navigation/
  Notifications/  Profile/  Settings/  Tenancy/  Tickets/
  LaraFoundryServiceProvider.php          # the ONE provider (auto-discovered)
config/              # larafoundry.php + larafoundry-{permissions,activitylog,media,notifications,tickets,email,legal}.php
database/migrations/ # auto-loaded into the host (loadMigrationsFrom) — NOT published
routes/              # web.php, auth.php, api.php, qr.php, admin.php, tenancy.php, … (auto-loaded)
lang/                # PHP groups + lang/frontend/{en,uk}.json (the vue-i18n bag)
resources/js/        # Inertia Pages/ (published) + components/, layouts/, composables/, i18n/, index.js (barrel)
resources/css/       # theme.css (Tailwind v4 @theme tokens)
resources/views/     # mail/ + the Inertia root view
tests/               # Pest on Orchestra Testbench
docs/                # host-integrator reference (one page per module) + integration guide
```

A module typically holds: `Models/`, `Http/Controllers/`, `Http/Requests/` (FormRequests), `Http/Middleware/`, `Actions/`, `Policies/`, `Support/`, `Contracts/`, `Concerns/` (traits), `Console/Commands/`, `Providers/`, `Events/`, `Listeners/`. Prefer **Actions** for write operations, **FormRequests** for validation/authorization, **Policies + Gates** for authorization, thin controllers.

## The single service provider

`src/LaraFoundryServiceProvider.php` is the spine. It is auto-discovered (`composer.json` → `extra.laravel.providers`). Read it before any structural change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmitryisaenko/larafoundry](https://github.com/dmitryisaenko/larafoundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
