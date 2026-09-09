---
trigger: always_on
description: > Free Laravel 12 admin dashboard template · RTL Layout Support · Tailwind CSS v4 · Blade Components · Alpine.js · Vite · ApexCharts · FullCalendar · Swiper · Flatpickr
---

# AGENTS.md — TailAdmin Free Laravel

> Free Laravel 12 admin dashboard template · RTL Layout Support · Tailwind CSS v4 · Blade Components · Alpine.js · Vite · ApexCharts · FullCalendar · Swiper · Flatpickr

## Repo Map

```
app/
├── Helpers/                  # MenuHelper and UI utilities
bootstrap/
├── app.php                   # Laravel application configuration & middleware setup
public/
├── images/                   # Static images, user avatars, icons (flag-us.svg, flag-sa.svg)
resources/
├── css/
│   └── app.css               # Tailwind CSS v4 theme (@theme), global utilities & 3rd party overrides
├── js/
│   ├── app.js                # Main JS entry point (Alpine.js, component dynamic imports)
│   ├── bootstrap.js          # Axios & HTTP setup
│   └── components/           # Client-side modules (calendar-init, charts, map)
└── views/
    ├── components/           # Reusable Blade components (<x-ui.*>, <x-form.*>, <x-common.*>, etc.)
    │   ├── common/           # Shared page elements (page-breadcrumb, component-card, table-dropdown)
    │   ├── ecommerce/        # Ecommerce dashboard widgets (metrics, monthly-target, recent-orders)
    │   ├── form/             # Form controls (input, select, date-picker, dropzone)
    │   ├── header/           # Header widgets (user-dropdown, notification-dropdown)
    │   ├── profile/          # User profile cards (personal-info, profile-card, address-card)
    │   ├── tables/           # Table variations (basic-tables-one to five)
    │   └── ui/               # UI primitives (alert, avatar, badge, button, modal)
    ├── layouts/              # Master layouts (app.blade.php, fullscreen-layout.blade.php, sidebar.blade.php, app-header.blade.php)
    └── pages/                # Route view templates (dashboard/, auth/, ui-elements/, form/, tables/, chart/, calender.blade.php)
routes/
├── web.php                   # Web application routes
├── api.php                   # API routes
└── console.php               # Console commands
```

## Stack

- **Laravel 12** with **PHP >= 8.2**.
- **Blade Template Engine** using modular, component-driven architecture (`<x-component-name />`).
- **Tailwind CSS v4** configured with `@tailwindcss/vite` and `@theme` tokens in `resources/css/app.css`.
- **Alpine.js v3** for reactive UI interactions, toggles, dropdowns, and global store management (`Alpine.store('theme')`).
- **Vite 7** with `laravel-vite-plugin` for lightning-fast asset compilation.
- **Third-Party Libraries**: ApexCharts, FullCalendar, Swiper, Flatpickr, jsVectorMap, Leaflet, MapLibre GL, Prism.js.
- Scripts:
  - `composer run dev` — runs `php artisan serve`, `npm run dev`, `php artisan pail` concurrently.
  - `npm run dev` / `npm run build` — Vite development and production asset bundling.
  - `composer test` — runs test suite via Pest PHP.

## Conventions

- **New Page**:
  - Add route in `routes/web.php`.
  - Create Blade view in `resources/views/pages/<category>/<page-name>.blade.php`.
  - Extend the appropriate layout: `@extends('layouts.app')` for dashboard pages or `@extends('layouts.fullscreen-layout')` for auth/error/utility pages.
- **New Reusable Component**:
  - Domain-specific component: `resources/views/components/<feature>/<component-name>.blade.php`.
  - Generic UI primitive: `resources/views/components/ui/<component-name>.blade.php`.
  - Form component: `resources/views/components/form/<component-name>.blade.php`.
  - Shared wrapper: `resources/views/components/common/<component-name>.blade.php`.
  - Always use `@props([...])` to declare default values and expected component properties.
- **Icons**:
  - Drop reusable SVGs in `resources/views/components/svg/<icon-name>.blade.php` and invoke via `<x-svg.icon-name />`.
  - For inline SVG icons in components, always use `fill-current` / `stroke-current` and size with `w-*` / `h-*` tokens.
- **Layout Structure**:
  - Dashboard pages: Wrapped in `@extends('layouts.app')` with `<x-common.page-breadcrumb>` at the top and content sections inside `<x-common.component-card>`.
  - Full-width / Auth pages: Wrapped in `@extends('layouts.fullscreen-layout')`.
- **State Management**:
  - Component-level state: Use `x-data="{ ... }"` in Alpine.js.
  - Global theme state: Managed via `Alpine.store('theme')` (handles `light`, `dark`, and system preference synced to `localStorage`).

## RTL Layout Architecture (Free Version Rules)

- **No Multi-Language Dictionaries**:
  - The Free version does not include translation dictionaries or localized routing.
  - UI copy remains in English.
  - RTL mode is a layout-direction feature toggled between English (LTR) and Arabic (RTL).
- **RTL State & Toggle**:
  - Toggled via the user dropdown Language menu (`user-dropdown.blade.php`).
  - Persisted in `localStorage.setItem("dir", "rtl" | "ltr")`.
  - Dynamically updates `document.documentElement.setAttribute("dir", ...)`.
  - Automatically closes the user dropdown upon language/direction selection.
  - Immediate execution script in `<head>` of `app.blade.php` and `fullscreen-layout.blade.php` prevents flash of unstyled LTR content.
- **CSS Logical Properties & RTL Spacing**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TailAdmin/tailadmin-laravel](https://github.com/TailAdmin/tailadmin-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
