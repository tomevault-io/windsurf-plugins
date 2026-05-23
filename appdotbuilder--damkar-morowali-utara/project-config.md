---
trigger: always_on
description: Project rules for PHP, Laravel, Inertia, Vite, Npm, React, Typescript, and Tailwind
---


# Context

This is a Laravel app using PHP 8+, with strict types `declare(strict_types=1);`, Composer, Laravel 12, InertiaJS 2, Vite, npm, React 19 with TypeScript, Tailwind 4, and radix-ui.

No other libraries or packages should be added unless the user approves.
The developer has the option of using Sail, Herd, or artisan serve to run their app.
Make use of `config('app.url)` or the URL helper to generate URLs to provide to the user.

# How to behave

- Think through and create a step by step plan before acting so you have a todo list to work through
    - The first step is always to evaluate the existing codebase to gather existing conventions to use in future steps
    - The final step is always to validate the modified code using `composer lint` and `composer test`
- Work through the plan step by step - creating failing tests, then the functionality to ensure the tests pass
- Follow all rules and Laravel 11+ conventions closely
- New code MUST strictly follow the same style as existing code and Laravel conventions - naming, structure, patterns, formatting, architecture
  **Directory structure adherence**: Always follow Laravel's standard directory structure (`app/Models/`, `app/Http/Controllers/`, `app/Http/Middleware/`, etc.) without creating non-standard directories unless explicitly approved.
  **Naming conventions**: Use Laravel's established naming patterns - `PascalCase` for controllers and models, `snake_case` for database tables and columns, `kebab-case` for URLs, and descriptive migration names with `artisan make:migration`.

# PHP Standards

- Use PHP 8+ features if they match the current code conventions: match(), union types, nullsafe operators, named arguments, attributes/annotations, constructor property promotion,
- Always enforce strict typing with declare(strict_types=1), scalar type hinting, return types, parameter types, and property types
- Use backed enums for fixed values
- Use short, focused, well-name, and testable methods
- Follow PSR-12 and PSR-4 coding standards.
- Prefer value objects over raw arrays when appropriate.
- Avoid over-engineering — keep things simple and pragmatic.
- Never leave TODOs or FIXMEs without clear context or a linked issue.
- Never leave comments within code blocks, only on methods.

# PHPDoc

- Write PHPDocs when needed based on this guide

Below is an example of a valid Laravel documentation block. Note that the @param attribute is followed by two spaces, the argument type, two more spaces, and finally the variable name:

```php
/**
 * Register a binding with the container.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */

public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```

When the `@param` or `@return` attributes are redundant due to the use of native types, they can be removed:

```php
/*
* Execute the job.
*/
public function handle(AudioProcessor $processor): void
{
    //
}
```

However, when the native type is generic, please specify the generic type through the use of the @param or @return attributes:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

# Creating new Laravel files

- Always use artisan commands to generate Laravel files: `artisan make:xyz`

# Laravel tests

- Always write Pest tests for new or updated functionality
- Tests must be written using Pest functions and assertions
- You MUST NOT write PHPUnit tests
- When asserting status codes on a response, use the specific method like `assertOk`, `assertForbidden`, `assertNotFound` etc, instead of using `assertStatus(403)` or similar
- When creating models for tests, you **MUST** use the factories for the models. Make use of custom states available
- Run the tests with `composer test` after finalizing functionality
- Run `composer lint` after changes

<example-pest-test path="tests/Unit/ExampleTest.php">
```php
<?php
test('that true is true', function () {
    expect(true)->toBeTrue();
});
```
</example-pest-test>

<example-pest-test path="tests/Feature/ExampleTest.php">

```php
<?php

it('returns a successful response', function () {
    $response = $this->get('/');
    $response->assertOk();
});
```

</example-pest-test>

---

# Laravel Coding Guidelines

## Backend Architecture & Laravel Patterns

### Laravel Framework Standards

- Implement proper type hints for all method parameters and return types (`Response`, `RedirectResponse`, etc.)
- Use Laravel's built-in validation, authentication, and authorization systems
- Commands created in `app\Console\Commands\` are automatically registered and available to use
- Never use `env()` directly in code, use `config('app.name')` for example
    - Use new environment variables for any new sensitive or configurable options, then add to a config file for use with `config('')`
- Implement proper error handling with custom exception classes when needed
- Listeners auto-listen for the events if they are type-hinted correctly
- Scheduled commands live in `routes/console.php`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appdotbuilder/damkar-morowali-utara](https://github.com/appdotbuilder/damkar-morowali-utara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
