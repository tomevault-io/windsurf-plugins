---
trigger: always_on
description: Guidance for AI agents (and humans) working inside the `bagisto/b2b-suite` package.
---

# AGENTS.md — Bagisto B2B Suite

Guidance for AI agents (and humans) working inside the `bagisto/b2b-suite` package.
This file describes how the package is wired into Bagisto and the conventions you must
follow when changing it.

## Overview

B2B Suite extends Bagisto's storefront and admin with company accounts, company users
and roles, requisition lists, quick order, quotations (RFQ), purchase orders and
company catalogs (per-company product **and category** visibility, custom pricing and
quantity-tier/volume pricing).

- **Namespace:** `Webkul\B2BSuite` → `src/`
- **Installed path:** `vendor/bagisto/b2b-suite` (via `composer require` — see *Installation
  & registration*). This repo is a **development checkout** at `packages/bagisto/b2b-suite`,
  symlinked into `vendor/` by the root `packages/*/*` path repository.
- **PHP:** 8.1+ (per `composer.json`); developed against **Bagisto 2.4 / Laravel 12**. Blade
  views are styled via the core Shop/Admin themes, which the package rebuilds into its own
  bundles (see *Styling* below).

## Installation & registration

**Installing the package — the [README](./README.md) is canonical:**

1. `composer require bagisto/b2b-suite` (installs into `vendor/bagisto/b2b-suite`).
2. Register `Webkul\B2BSuite\Providers\B2BSuiteServiceProvider` in `bootstrap/providers.php`,
   **after the Shop package** (or last in the array). Composer auto-discovery is intentionally
   **disabled** — discovery would load the provider too early, before Shop.
3. `php artisan b2b-suite:install` (migrate, seed, publish assets/overrides, clear caches).

**Development checkout (this repo):** instead of a registry install, the package lives at
`packages/bagisto/b2b-suite` and is wired via the root `composer.json` path repository
(`"type": "path", "url": "packages/*/*"` + `"bagisto/b2b-suite": "@dev"`), which symlinks it
into `vendor/bagisto/b2b-suite`. Provider registration in `bootstrap/providers.php` is the same.
Do not confuse this dev layout with the install steps above.

`B2BSuiteServiceProvider` itself registers `ModuleServiceProvider` (Concord models) and
`EventServiceProvider`, so there is **no** `config/concord.php` entry.

## The active flag

Almost everything is gated behind the admin config flag:

```php
core()->getConfigData('b2b.general.settings.active')
```

When inactive: B2B routes, menus, the company registration view and the company-specific
parts of overridden views are not shown. Keep new B2B-only behavior behind this flag.

## Extending core without editing it

- **Controllers / models / repositories:** swapped in the container by
  `Providers/B2BSuiteManager.php` (`$app->bind(CoreClass::class, B2BClass::class)` and
  `concord->registerModel(...)`). B2B classes `extend` the core ones and override only
  what they need. Current binds include the core `ProductRepository` (extended for
  company-catalog visibility — see *Company Catalog* below).
- **Inline content:** injected into core blades via `view_render_event(...)` listeners
  registered in `Providers/EventServiceProvider.php` (`$e->addTemplate('b2b::...')`).
- **View / component overrides:** published to the package-namespace override path
  `resources/views/vendor/<namespace>` (Laravel's standard override, registered by core's
  `loadViewsFrom('shop')` / `loadViewsFrom('admin')`). **One mechanism for everything** —
  regular `shop::`/`admin::` views (e.g. `shop::customers.sign-in`) *and* anonymous
  `x-shop::` components (e.g. the account navigation, which compile to `shop::components.<name>`
  and which theme view-path overrides can't reach). Mirror the namespaced path under
  `publishables/resources/vendor/<namespace>/<path>`; everything is published — no runtime
  namespace hacks.

## `publishables/` — the single source of everything that gets published

**Convention: anything that is published to the application lives under `publishables/`.**
Never publish directly from `src/`.

```
publishables/
├── storage/                   → storage/app/public                 sample data
└── resources/
    └── vendor/                → resources/views/vendor             all view/component overrides
        ├── shop/
        │   ├── customers/sign-in.blade.php                         overrides shop::customers.sign-in
        │   ├── checkout/cart/{index,summary,request-quote-modal}.blade.php
        │   └── components/layouts/account/navigation.blade.php     overrides x-shop::layouts.account.navigation
        └── admin/
            └── customers/customers/index/create.blade.php          overrides admin::customers.customers.index.create
```

A file published to `resources/views/vendor/<namespace>/<path>` overrides the matching
namespaced view (`shop::customers.sign-in`, `admin::customers.customers.index.create`, the
`x-shop::layouts.account.navigation` component, …). This is the single override mechanism.

To override another view/component: add it under `publishables/resources/vendor/<namespace>/<path>`
(mirror the namespaced path without the namespace prefix — components live under
`<namespace>/components/<name>`), then re-publish.

## Styling — the package builds its own theme bundles

B2B Blade views are styled with the core **Shop/Admin Tailwind themes**, but they live

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bagisto/b2b-ecommerce](https://github.com/bagisto/b2b-ecommerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
