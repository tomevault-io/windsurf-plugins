---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== .ai/base-instructions rules ===

# Laravel Catalunya AI Base Instructions

These are some base instructions to be used for AI interactions related to the Laravel Catalunya project.

## Strings

All strings in the project should be in English. This includes code comments, commit messages, documentation, and any user-facing text.

For user-facing text, we will use Laravel's localization features to allow for future translations. When adding new user-facing text to the application, use the `__('Your text here')` helper function to allow for localization, but write the actual text in English. All strings surrounded by `__('')` should be updated in the language files located in `lang` directory, with English as the default language.

## Models

After creating or updating models always run this command:

```bash
php artisan ide-helper:models -M
```

This will generate the necessary PHPDoc annotations for the models, which improves IDE support and helps with static analysis tools like PHPStan. Always ensure that your models are properly annotated to maintain code quality and readability.

## Actions

All actions that returns lists of items should be paginated. This means that instead of returning a full collection of items, the action should return a paginated response using Laravel's pagination features. This is important for performance and scalability, especially when dealing with large datasets.

When creating an action that returns a list of items, use the `paginate()` method on the query builder to return a paginated response.

=== .ai/local-environment rules ===

# Local development environment guidelines

## Overview

This project is dockerized in a container called `laravel_cat`.

## Commands

When run Laravel artisan, vendor or composer commands is needed, use the following format:

```bash
docker exec -i laravel_cat <command>
```

For example, to run migrations:

```bash
docker exec -i laravel_cat php artisan migrate
```

To run tests:

```bash
docker exec -i laravel_cat .vendor/bin/pest
```

To install composer dependencies:

```bash
docker exec -i laravel_cat composer install
```

Take into account that `npm` and `node` commands should be run in your local machine, not inside the container. Be sure you have the correct Node.js version declared in the `.nvmrc` file. You can use `nvm` to manage Node.js versions.

=== .ai/pattern rules ===

# Action pattern

This project uses the **Action Pattern** to encapsulate business logic into discrete, reusable classes called "Actions". This approach promotes separation of concerns, making the codebase more maintainable and testable.

## Key Concepts

- **Action Classes**: Each action is represented by a class that contains a single public method, named `execute()`, which performs the specific task.
- **Reusability**: Actions can be reused across different parts of the application, reducing code duplication.
- **Testability**: Actions can be easily tested in isolation, allowing for more straightforward unit tests.

## Location

Action classes are stored in the `app/Actions` directory. You can organize them further into subdirectories based on their domain or functionality.

=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to ensure the best experience when building Laravel applications.

## Foundational Context

This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.4.15
- filament/filament (FILAMENT) - v4
- laravel/framework (LARAVEL) - v12
- laravel/prompts (PROMPTS) - v0
- laravel/sanctum (SANCTUM) - v4
- livewire/flux (FLUXUI_FREE) - v2
- livewire/livewire (LIVEWIRE) - v3
- larastan/larastan (LARASTAN) - v3
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- rector/rector (RECTOR) - v2
- alpinejs (ALPINEJS) - v3
- tailwindcss (TAILWINDCSS) - v4

## Skills Activation

This project has domain-specific skills available. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

- `fluxui-development` — Develops UIs with Flux UI Free components. Activates when creating buttons, forms, modals, inputs, dropdowns, checkboxes, or UI components; replacing HTML form elements with Flux; working with flux: components; or when the user mentions Flux, component library, UI components, form fields, or asks about available Flux components.
- `livewire-development` — Develops reactive Livewire 3 components. Activates when creating, updating, or modifying Livewire components; working with wire:model, wire:click, wire:loading, or any wire: directives; adding real-time updates, loading states, or reactivity; debugging component behavior; writing Livewire tests; or when the user mentions Livewire, component, counter, or reactive UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alibori) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
