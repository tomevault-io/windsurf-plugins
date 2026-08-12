---
trigger: always_on
description: This is a Laravel package for queue monitoring with metrics, analytics, and insights. You are an expert with all the packages and versions listed below. Ensure you abide by these specific packages & versions.
---

# Queue Metrics for Laravel - Development Guidelines

## Foundational Context
This is a Laravel package for queue monitoring with metrics, analytics, and insights. You are an expert with all the packages and versions listed below. Ensure you abide by these specific packages & versions.

- php - ^8.3|^8.4
- laravel/framework - ^11.0
- cboxdk/system-metrics - ^1.2
- spatie/laravel-package-tools - ^1.16
- spatie/laravel-prometheus - ^1.3
- laravel/pint - ^1.14
- pestphp/pest - ^4.0
- orchestra/testbench - ^9.14
- larastan/larastan - ^3.0

## Conventions
- You must follow all existing code conventions used in this package. When creating or editing a file, check sibling files for the correct structure, approach, naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Package Structure & Architecture
- This is a Laravel package, not an application. Use package development best practices.
- Stick to existing directory structure - don't create new base folders without approval.
- Do not change the package's dependencies without approval.
- Main source code goes in `src/`, tests in `tests/`, config in `config/`
- Use Spatie's Laravel Package Tools conventions for service provider setup

## Verification & Testing
- Tests are critical for package development. Always write tests for new features.
- Do not create verification scripts when tests cover that functionality.

## Documentation Files
- You must only create documentation files if explicitly requested by the user.

## PHP

- Always use curly braces for control structures, even if it has one line.

### Constructors
- Use PHP 8 constructor property promotion in `__construct()`.
    - <code-snippet>public function __construct(public GitHub $github) { }</code-snippet>
- Do not allow empty `__construct()` methods with zero parameters.

### Type Declarations
- Always use explicit return type declarations for methods and functions.
- Use appropriate PHP type hints for method parameters.

<code-snippet name="Explicit Return Types and Method Params" lang="php">
protected function isAccessible(User $user, ?string $path = null): bool
{
    ...
}
</code-snippet>

## Comments
- Prefer PHPDoc blocks over comments. Never use comments within the code itself unless there is something _very_ complex going on.

## PHPDoc Blocks
- Add useful array shape type definitions for arrays when appropriate.

## Enums
- Typically, keys in an Enum should be TitleCase. For example: `FavoritePerson`, `BestLake`, `Monthly`.

## Laravel Package Development

- This is a Laravel package. Use Orchestra Testbench for testing Laravel functionality.
- Service providers should extend Spatie's `PackageServiceProvider` for convention and ease of use.

### Database & Models
- Always use proper Eloquent relationship methods with return type hints.
- Prefer Eloquent models and relationships over raw database queries.
- Generate code that prevents N+1 query problems by using eager loading.
- When modifying columns in migrations, include all previously defined attributes to prevent data loss.
- Casts should be defined in a `casts()` method on models rather than the `$casts` property.

### Configuration
- Package configuration should be publishable via service provider.
- Use environment variables only in configuration files - never use the `env()` function directly outside of config files.
- Use `config('queue-metrics.key')` not `env('QUEUE_METRICS_KEY')`.

### Testing with Orchestra Testbench
- Use Orchestra Testbench to test package functionality in a Laravel environment.
- When creating models for tests, use factories. Check if the factory has custom states before manually setting up models.
- Use `fake()` for generating test data following existing conventions.

## Laravel Pint Code Formatter

- You must run `vendor/bin/pint --dirty` before finalizing changes to ensure your code matches the project's expected style.
- Do not run `vendor/bin/pint --test`, simply run `vendor/bin/pint` to fix any formatting issues.

## Pest Testing Framework

- This package uses Pest (v4) for testing. All tests must be written using Pest syntax.
- Every time a test has been updated, run that specific test to verify it passes.
- When tests relating to your feature are passing, ask the user if they would like to run the entire test suite.
- Tests should cover all happy paths, failure paths, and edge cases.
- You must not remove any tests or test files from the tests directory without approval.

### Running Tests
- Run the minimal number of tests using filters before finalizing.
- To run all tests: `vendor/bin/pest` or `composer test`
- To run tests in a specific file: `vendor/bin/pest tests/Feature/ExampleTest.php`
- To filter tests by name: `vendor/bin/pest --filter=testName`
- To run tests with coverage: `vendor/bin/pest --coverage` or `composer test-coverage`

### Pest Syntax
- Use `test()` function for tests: `test('it can do something', function() { ... })`
- Use `it()` for behavior-driven tests: `it('does something', function() { ... })`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cboxdk/laravel-queue-autoscale](https://github.com/cboxdk/laravel-queue-autoscale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
