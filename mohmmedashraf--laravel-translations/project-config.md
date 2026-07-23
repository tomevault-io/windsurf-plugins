---
trigger: always_on
description: This is the **free** version (`outhebox/laravel-translations`). The pro version lives at `../laravel-translations-pro` and depends on this package — never introduce breaking changes without checking the pro version.
---

# Free & Pro Architecture

This is the **free** version (`outhebox/laravel-translations`). The pro version lives at `../laravel-translations-pro` and depends on this package — never introduce breaking changes without checking the pro version.

## How Pro Extends Free

- **Composer**: Pro requires `outhebox/laravel-translations` via a path symlink and uses a separate namespace (`Outhebox\TranslationsPro`).
- **Backend**: Pro never overrides free classes. It adds its own controllers, services, models, and commands. It hooks into free via event listeners (`TranslationSaved`, `ImportCompleted`).
- **Frontend**: Pro's Vite config defines a `@free` alias pointing to `../laravel-translations/resources/js`. Pro components either re-export from `@free/...` (when no changes needed) or provide their own implementation.
- **Types**: Pro re-exports `@free/types/translations` and extends as needed.
- **Database**: Free models already include pro-ready fields (`ai_generated`, `ai_provider`, `needs_review`, `priority`, `context_note`). Pro adds its own migration tables (revisions, activity_logs, glossary, etc.).
- **Tests**: Each package has its own `tests/Pest.php` and `TestCase`. Pro references free routes from its vendor directory.

<laravel-boost-guidelines>
=== .ai/testing rules ===

# Testing

## Locations

- `tests/Unit/Actions/` — Action unit tests
- `tests/Feature/Http/Controllers/{Domain}/` — HTTP tests
- `tests/Extraction/` — Extraction output tests

## Standards

- AAA: Arrange, Act, Assert
- Test happy path, errors, edge cases
- Test 403 (auth) and 422 (validation)

## Extraction Tests

```php
it('extracts correctly', function () {
    expect(extract(['subscriptions']))
        ->toContain('SubscriptionController')
        ->not->toContain('#feature:');
});
```

## Commands

```bash
php artisan test --compact
```

=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to ensure the best experience when building Laravel applications.

## Foundational Context

This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.4.17
- inertiajs/inertia-laravel (INERTIA) - v2
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/wayfinder (WAYFINDER) - v0
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- @inertiajs/react (INERTIA) - v2
- react (REACT) - v19
- tailwindcss (TAILWINDCSS) - v4
- @laravel/vite-plugin-wayfinder (WAYFINDER) - v0
- eslint (ESLINT) - v9
- prettier (PRETTIER) - v3

## Skills Activation

This project has domain-specific skills available. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

- `wayfinder-development` — Activates whenever referencing backend routes in frontend components. Use when importing from @/actions or @/routes, calling Laravel routes from TypeScript, or working with Wayfinder route functions.
- `pest-testing` — Tests applications using the Pest 4 PHP framework. Activates when writing tests, creating unit or feature tests, adding assertions, testing Livewire components, browser testing, debugging test failures, working with datasets or mocking; or when the user mentions test, spec, TDD, expects, assertion, coverage, or needs to verify functionality works.
- `inertia-react-development` — Develops Inertia.js v2 React client-side applications. Activates when creating React pages, forms, or navigation; using &lt;Link&gt;, &lt;Form&gt;, useForm, or router; working with deferred props, prefetching, or polling; or when user mentions React with Inertia, React pages, React forms, or React navigation.
- `tailwindcss-development` — Styles applications using Tailwind CSS v4 utilities. Activates when adding styles, restyling components, working with gradients, spacing, layout, flex, grid, responsive design, dark mode, colors, typography, or borders; or when the user mentions CSS, styling, classes, Tailwind, restyle, hero section, cards, buttons, or any visual/UI changes.

## Conventions

- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, and naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts

- Do not create verification scripts or tinker when tests cover that functionality and prove they work. Unit and feature tests are more important.

## Application Structure & Architecture

- Stick to existing directory structure; don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MohmmedAshraf/laravel-translations](https://github.com/MohmmedAshraf/laravel-translations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
