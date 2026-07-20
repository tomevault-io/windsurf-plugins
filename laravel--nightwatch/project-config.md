---
trigger: always_on
description: Laravel Nightwatch is a hosted application monitoring platform package for Laravel. It gathers metrics within Laravel applications and transmits them to the Nightwatch service. This is an official Laravel package that must maintain high code quality standards.
---

# Laravel Nightwatch Agent Guidelines

## Project Overview

Laravel Nightwatch is a hosted application monitoring platform package for Laravel. It gathers metrics within Laravel applications and transmits them to the Nightwatch service. This is an official Laravel package that must maintain high code quality standards.

## Code Style & Formatting

- **Use Laravel Pint** with the project's configuration (`pint.json`)
- **Strict comparisons**: Use `===` and `!==` instead of `==` and `!=`
- **Strict parameters**: Enable strict parameter checking
- **Global namespace imports**: Import classes, constants, and functions from global namespace
- **Native function invocation**: Use namespaced native function calls
- **Method chaining indentation**: Properly indent method chains
- **Explicit string variables**: Use explicit string variable syntax when appropriate
- **Combine consecutive isset/unset**: Group consecutive isset/unset calls
- **Ternary to null coalescing**: Prefer null coalescing operator (`??`) over ternary

## Type Safety

- **PHPStan Level Max**: All code must pass PHPStan at maximum level
- **Type hints**: Always provide type hints for:
  - Method parameters
  - Return types
  - Properties (with PHPDoc when needed)
- **Generic types**: Use PHPDoc generics for complex types (e.g., `@template TState`)
- **Array shapes**: Document array shapes with PHPDoc (e.g., `array{enabled: bool, sampling: array{...}}`)

## Architecture Patterns

### Hooks Pattern
- Hooks are event listeners that capture Laravel framework events
- Located in `src/Hooks/`
- Should be `final` classes marked `@internal`
- Always wrap hook logic in try-catch and report exceptions as handled
- Example pattern:
```php
final class QueryExecutedListener
{
    public function __construct(
        private Core $nightwatch,
    ) {}

    public function __invoke(QueryExecuted $event): void
    {
        try {
            $this->nightwatch->query($event);
        } catch (Throwable $e) {
            $this->nightwatch->report($e, handled: true);
        }
    }
}
```

### Sensors Pattern
- Sensors transform events into records and payload factories
- Located in `src/Sensors/`
- Should be `final` classes marked `@internal`
- Return tuple: `[Record, callable(): array<mixed>]` (for record-based sensors) or `?array` (for direct array sensors like `ScheduledTaskSensor`)
- Use `Location` helper for file/line detection
- Use `Clock` for timestamp management
- **Check property visibility**: Before using reflection, verify if Laravel properties are public. Prefer direct property access over reflection when possible.
- **Trust Laravel's internal constraints**: If Laravel enforces constraints on a value (e.g., requires >= 1 second for repeating tasks), trust those constraints rather than re-checking in our code

### State Management
- Use `RequestState` for HTTP requests
- Use `CommandState` for Artisan commands, scheduled tasks, and job attempts
- State is managed through `Core` class
- Execution stages tracked via `ExecutionStage` enum

### Concerns/Traits
- Shared functionality goes in `src/Concerns/`
- Examples: `CapturesState`, `RedactsRecords`, `RejectsRecords`
- Use traits to keep `Core` class focused

## Class Design

- **Final classes**: Use `final` for internal classes that shouldn't be extended
- **Visibility**: Use `private` by default, `protected` only when necessary
- **Property promotion**: Prefer constructor property promotion
- **Readonly**: Consider `readonly` properties when appropriate (PHP 8.2+)
- **Avoid unnecessary variables**: Only create variables when they're used multiple times or significantly improve readability. Don't create variables for single-use values that can be inlined directly.

## Documentation

- **@internal**: Mark all internal classes/methods with `@internal`
- **@api**: Mark public API methods with `@api`
- **PHPDoc**: Provide comprehensive PHPDoc for:
  - Complex types
  - Generic types
  - Array shapes
  - Callable signatures
- **Method documentation**: Document public methods, especially those marked `@api`
- **Avoid unnecessary comments**: Don't add inline comments unless necessary, especially when existing code doesn't already have them for similar scenarios. Let the code be self-documenting through clear naming and structure.

## Error Handling

- **Graceful degradation**: Never let monitoring code break the application
- **Exception handling**: Always catch `Throwable` in hooks
- **Report as handled**: Internal exceptions should be reported as `handled: true`
- **Unrecoverable exceptions**: Use `Nightwatch::unrecoverableExceptionOccurred()` for fatal errors

## Testing

- **Test coverage**: Maintain high test coverage
- **Test structure**: 
  - Feature tests in `tests/Feature/`
  - Unit tests in `tests/Unit/`
- **Test helpers**: Use `TestCase` helper methods:
  - `fakeIngest()` for mocking ingest
  - `fakeTcpStreams()` for testing network
  - `setExecutionStart()` for time manipulation
- **Test isolation**: Each test should be independent
- **Test naming**: Use descriptive test method names


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laravel/nightwatch](https://github.com/laravel/nightwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
