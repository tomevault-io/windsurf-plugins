---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== .ai/php-laravel rules ===

## General code instructions

- Don't generate code comments above the methods or code blocks if they are obvious. Don't add docblock comments when defining variables, unless instructed to, like `/** @var \App\Models\User $currentUser */`. Generate comments only for something that needs extra explanation for the reasons why that code was written.
- For new features, you MUST generate Pest automated tests.
- To run tests, ALWAYS delegate to the `test-runner` subagent via Task tool. Do not run `vendor/bin/sail artisan test` directly — it pollutes your context with long output.
- For library documentation, if some library is not available in Laravel Boost 'search-docs', always use context7. Automatically use the Context7 MCP tools to resolve library id and get library docs without me having to explicitly ask.
- If you made changes to CSS/Javascript files or added new Tailwind classes in Blade, run `npm run build` after all front-end changes are finished.

## Memory instructions (mem0)

This project uses the mem0 skill for persistent memory across sessions.
ACTIVELY use mem0 — do not wait to be asked. The skill handles the API
mechanics;

---

## PHP instructions

- In PHP, use `match` operator over `switch` whenever possible
- Generate Enums always in the folder `app/Enums`, not in the main `app/` folder, unless instructed differently.
- Always use Enum value as the default in the migration if column values are from the enum. Always casts this column to the enum type in the Model.
- Don't create temporary variables like `$currentUser = auth()->user()` if that variable is used only one time.
- Always use Enum where possible instead of hardcoded string values, if Enum class exists. For example, in Blade files, and in the tests when creating data if field is casted to Enum then use that Enum instead of hardcoding the value.

---

## Laravel instructions

- Always run Laravel-related CLI commands using Sail: prefix commands with `./vendor/bin/sail`.
    - Use `./vendor/bin/sail artisan migrate` instead of `php artisan migrate`
    - Use `./vendor/bin/sail artisan test` instead of `php artisan test`
    - Use `./vendor/bin/sail artisan make:model User` instead of `php artisan make:model User`
    - Use `./vendor/bin/sail composer install` instead of `composer install`
    - Use `./vendor/bin/sail npm run build` instead of `npm run build`
- Never assume global PHP, Composer, or Node execution. Always run commands inside the Sail container.
- **Eloquent Observers** should be registered in Eloquent Models with PHP Attributes, and not in AppServiceProvider. Example: `#[ObservedBy([UserObserver::class])]` with `use Illuminate\Database\Eloquent\Attributes\ObservedBy;` on top
- Aim for "slim" Controllers/Components and put larger logic pieces in Service classes
- Use Laravel helpers instead of `use` section classes. Examples: use `auth()->id()` instead of `Auth::id()` and adding `Auth` in the `use` section. Other examples: use `redirect()->route()` instead of `Redirect::route()`, or `str()->slug()` instead of `Str::slug()`.
- Don't use `whereKey()` or `whereKeyNot()`, use specific fields like `id`. Example: instead of `->whereKeyNot($currentUser->getKey())`, use `->where('id', '!=', $currentUser->id)`.
- Don't add `::query()` when running Eloquent `create()` statements. Example: instead of `User::query()->create()`, use `User::create()`.
- When adding columns in a migration, update the model's `$fillable` array to include those new attributes.
- Never chain multiple migration-creating commands (e.g., `make:model -m`, `make:migration`) with `&&` or `;` — they may get identical timestamps. Run each command separately and wait for completion before running the next.
- Enums: If a PHP Enum exists for a domain concept, always use its cases (or their `->value`) instead of raw strings everywhere — routes, middleware, migrations, seeds, configs, and UI defaults.
- Don't create Controllers with just one method which just returns `view()`. Instead, use `Route::view()` with Blade file directly.
- Always use Laravel's @session() directive instead of @if(session()) for displaying flash messages in Blade templates.
- In Blade files always use `@selected()` and `@checked()` directives instead of `selected` and `checked` HTML attributes. Good example: @selected(old('status') === App\Enums\ProjectStatus::Pending->value). Bad example: {{ old('status') === App\Enums\ProjectStatus::Pending->value ? 'selected' : '' }}.

### Service classes

- Use Service classes to encapsulate reusable business logic, keeping Controllers and Livewire Components slim.
- Service classes MUST be created in the `app/Services/` folder.
- If a Service is used in only ONE method of a Controller or Component, inject it directly into that method via type-hinting. If it is used in MULTIPLE methods, initialize it in the Constructor (or `mount()`/`boot()` for Livewire Components).
- The same injection rule applies to both traditional Controllers and Livewire Components — use `mount()` or `boot()` to inject Services in Components when needed across multiple methods, or inject directly into the action method.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beerandcodeteam/vitrine-local](https://github.com/beerandcodeteam/vitrine-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
