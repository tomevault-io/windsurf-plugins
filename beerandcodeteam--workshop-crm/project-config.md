---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== .ai/language-and-localization rules ===

## Language & Localization Rules


- All technical guidelines, code standards, and documentation MUST be written in English.
- All source code (class names, method names, variables, comments, enums, etc.) MUST be written in English.
- However, ALL user-facing content MUST be written in Brazilian Portuguese (pt-BR).

This includes, but is not limited to:

- Labels
- Validation error messages
- Flash messages
- Alerts
- Notifications
- Button texts
- Form placeholders
- Tooltips
- Modal texts
- Email contents
- API validation responses
- Exception messages displayed to users
- Front-end UI strings

- Do NOT mix English and Portuguese in user-facing content.
- Do NOT leave default Laravel validation messages in English.
- If translations are needed, they MUST use Brazilian Portuguese (pt-BR) conventions and terminology.
- Avoid literal translations. Prefer natural Brazilian Portuguese phrasing used in real applications.

Technical rule of thumb:

> Code in English. Interface in Brazilian Portuguese.

=== .ai/php-laravel rules ===

## General code instructions


- Don't generate code comments above the methods or code blocks if they are obvious. Don't add docblock comments when defining variables, unless instructed to, like `/** @var \App\Models\User $currentUser */`. Generate comments only for something that needs extra explanation for the reasons why that code was written.
- For new features, you MUST generate Pest automated tests.
- For library documentation, if some library is not available in Laravel Boost 'search-docs', always use context7. Automatically use the Context7 MCP tools to resolve library id and get library docs without me having to explicitly ask.
- If you made changes to CSS/Javascript files or added new Tailwind classes in Blade, run `npm run build` after all front-end changes are finished.

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



<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beerandcodeteam/workshop-crm](https://github.com/beerandcodeteam/workshop-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
