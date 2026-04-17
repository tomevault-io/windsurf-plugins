---
trigger: always_on
description: This is **Sandbox** - a Laravel 12 + Vue 3 + Inertia.js application with a beautiful full-screen clock feature, hosted using Laravel Herd.
---

# GitHub Copilot Instructions

## Project Overview
This is **Sandbox** - a Laravel 12 + Vue 3 + Inertia.js application with a beautiful full-screen clock feature, hosted using Laravel Herd.

## Development Preferences

### Build Process (IMPORTANT)
- **Always suggest**: `npm run build` 
- **Never suggest**: `npm run dev` or watch mode
- **Reason**: We prefer single-step builds over background processes for cleaner development

### Architecture Patterns
- **Backend**: Laravel controllers return `Inertia::render('ComponentName')`
- **Frontend**: Vue 3 Composition API with `<script setup lang="ts">
- **Routing**: Use auto-generated TypeScript routes from `@/routes`
- **Styling**: Tailwind CSS 4.1 utility classes

### File Organization
```
resources/js/pages/     # Inertia page components (main app pages)
resources/js/components/ # Reusable Vue components  
resources/js/routes/    # Auto-generated TypeScript routes (don't edit manually)
routes/web.php          # Laravel routes (edit here to add new routes)
```

### Code Style Guidelines
- **Vue Components**: Use Composition API with TypeScript
- **Imports**: Use `@/` alias for `resources/js/` directory
- **CSS**: Prefer Tailwind utilities over custom CSS
- **Routes**: Import from `@/routes` and use generated functions

### URL Structure (Local Development)
- Base: `https://sandbox.test` (Laravel Herd with HTTPS for local development)
- Clock: `https://sandbox.test/clock` (main feature)
- No localhost or port numbers (Herd manages this locally)
- **Note**: Production URLs will be different - this is local development setup only

### Adding New Features
1. Create Vue page component in `resources/js/pages/`
2. Add route in `routes/web.php` 
3. Run `npm run build` to regenerate TypeScript routes
4. Import and use the generated route function

### Technology Stack
- Laravel 12 with Inertia.js
- Vue 3 + TypeScript 
- Tailwind CSS 4.1 (new syntax)
- Vite 7 with Laravel Vite Plugin + Wayfinder
- SQLite database
- Laravel Herd hosting

### Current Features
- Full-screen real-time clock (`/clock`) with gradient background and animations
- Laravel Fortify authentication system
- Auto-generated TypeScript routes via Wayfinder plugin
- Responsive design with Tailwind CSS

## Code Examples

### Adding a New Page
```php
// routes/web.php
Route::get('my-page', function () {
    return Inertia::render('MyPage');
})->name('my-page');
```

```vue
<!-- resources/js/pages/MyPage.vue -->
<script setup lang="ts">
import { Head } from '@inertiajs/vue3';
</script>

<template>
    <Head title="My Page - Sandbox" />
    <div class="min-h-screen bg-gradient-to-br from-slate-900 to-purple-900">
        <h1 class="text-4xl text-white">My New Page</h1>
    </div>
</template>
```

### Using Routes in Components
```vue
<script setup lang="ts">
import { myPage } from '@/routes';
import { Link } from '@inertiajs/vue3';
</script>

<template>
    <Link :href="myPage()" class="text-purple-300 hover:text-purple-100">
        Go to My Page
    </Link>
</template>
```

## Environment Context (Local Development)
- App Name: "Sandbox"
- Local Domain: sandbox.test (managed by Herd for development)
- HTTPS automatically provided by Herd locally
- Local Session domain: .sandbox.test
- **Note**: Production environment will have different domain and configuration

===
<laravel-boost-guidelines>
===
# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

```
- php - 8.4.13
- inertiajs/inertia-laravel (INERTIA) - v2
- laravel/fortify (FORTIFY) - v1
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- laravel/wayfinder (WAYFINDER) - v0
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- phpunit/phpunit (PHPUNIT) - v11
- @inertiajs/vue3 (INERTIA) - v2
- tailwindcss (TAILWINDCSS) - v4
- vue (VUE) - v3
- @laravel/vite-plugin-wayfinder (WAYFINDER) - v0
- eslint (ESLINT) - v9
- prettier (PRETTIER) - v3
```


## Conventions
- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts
- Do not create verification scripts or tinker when tests cover that functionality and prove it works. Unit and feature tests are more important.

## Application Structure & Architecture
- Stick to existing directory structure - don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brownrl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
