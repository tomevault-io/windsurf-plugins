---
trigger: always_on
description: You are an expert in PHP, Laravel, React, Typescript, Inertia, Blade, Pest, Tailwind and Radix UI.
---

You are an expert in PHP, Laravel, React, Typescript, Inertia, Blade, Pest, Tailwind and Radix UI.

This application is Acara Plate, a personalized nutrition and meal planning platform that leverages AI to create customized meal plans based on individual user data such as age, weight, height, dietary preferences, and goals. The platform aims to simplify the process of meal planning by providing users with tailored recipes and grocery lists that align with their nutritional needs and lifestyle choices.

<laravel-boost-guidelines>
=== .ai/app.actions rules ===

# App/Actions guidelines

- This application uses the Action pattern and prefers for much logic to live in reusable and composable Action classes.
- Actions live in `app/Actions`, they are named based on what they do, with no suffix.
- Actions will be called from many different places: jobs, commands, HTTP requests, API requests, MCP requests, and more.
- Create dedicated Action classes for business logic with a single `handle()` method.
- Inject dependencies via constructor using private properties.
- Create new actions with `php artisan make:action "{name}" --no-interaction`
- Wrap complex operations in `DB::transaction()` within actions when multiple models are involved.
- Some actions won't require dependencies via `__construct` and they can use just the `handle()` method.

<code-snippet name="Example action class" lang="php">

<?php

declare(strict_types=1);

namespace App\Actions;

final readonly class CreateFavorite
{
    public function __construct(private FavoriteService $favorites)
    {
        //
    }

    public function handle(User $user, string $favorite): bool
    {
        return $this->favorites->add($user, $favorite);
    }
}

</code-snippet>

=== .ai/general rules ===

# General Guidelines

- Don't include any superfluous PHP Annotations, except ones that start with `@` for typing variables.

# Laravel Wayfinder Guidelines

When you generate the routes, use `--with-form` so that `Wayfinder` can create form request classes for you, like so;

```shell
php artisan wayfinder:generate --with-form
```

## Approach for writing tests

 - Use a cleaner, more testable approach. Separate edge cases into their own methods or classes.
 - Use data providers to test multiple scenarios in a single test method.

## Test Coverage Workflow

WHAT MAKES A GREAT TEST:
A great test covers behavior users depend on. It tests a feature that, if broken, would frustrate or block users. It validates real workflows, not implementation details, and it catches regressions before users do. Do NOT write tests just to increase coverage—use coverage as a guide to find untested user-facing behavior. If uncovered code is not worth testing (boilerplate, unreachable error branches, internal plumbing), add @codeCoverageIgnoreStart and @codeCoverageIgnoreEnd annotations instead of writing low-value tests.

PROCESS:
1. Run herd coverage ./vendor/bin/pest --coverage --parallel to see which files have low coverage.
2. Read the uncovered lines and identify the most important user-facing feature that lacks tests. Prioritize: error handling users will hit, controller actions, form requests, service methods, Artisan commands, API endpoints. Deprioritize: internal utilities, edge cases users won't encounter, framework boilerplate.
3. Write ONE meaningful Pest test that validates the feature works correctly for users.
4. Run herd coverage ./vendor/bin/pest --coverage --parallel again; coverage should rise as a side effect of testing real behavior.
5. Commit with message: test(<file>): <describe the user behavior being tested>
6. Append super-concise notes to test-coverage-progress.txt describing what you tested, current coverage %, and any learnings.

ONLY WRITE ONE TEST PER ITERATION.
If statement coverage reaches 100%, output <promise>COMPLETE</promise>.

## Code Testability Guidelines

- **Avoid inline closures in controller responses** - When returning data from controllers (especially with Inertia), don't use inline closures for data transformation as they are difficult to test.
- **Extract formatting logic to models** - Instead of writing transformation closures in controllers, add dedicated methods to models (e.g., `toResponseData()`, `formattedItemsByCategory()`) that can be unit tested independently.
- **Use DTOs for response data** - Instead of inline array type declarations like `@return array{id: int, name: string}`, create dedicated Data Transfer Objects (DTOs) in `app/Data/` using Spatie Laravel Data.
- **Keep controller methods thin** - Controllers should orchestrate, not transform. Move data formatting and business logic to models, actions, or dedicated service classes.

### Bad Example (untestable closure with inline array type):

```php
// In Controller - hard to test, inline type declaration
/**
 * @return array{id: int, name: string}
 */
return [
    'items' => $collection->map(fn ($item) => [
        'id' => $item->id,
        'name' => $item->name,
    ]),
];
```

### Good Example (testable method with DTO):

```php
// In Data/ItemResponseData.php
final class ItemResponseData extends Data
{
    public function __construct(
        public int $id,
        public string $name,
    ) {}
}

// In Model

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acara-app/plate](https://github.com/acara-app/plate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
