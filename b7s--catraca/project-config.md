---
trigger: always_on
description: This document captures architectural patterns, best practices, and design decisions for building maintainable, testable, and well-organized PHP applications. These principles apply to any PHP codebase — whether web APIs, CLI tools, background workers, or libraries — using any modern framework or none at all.
---

# AGENTS.md — PHP Architecture & Performance Best Practices

This document captures architectural patterns, best practices, and design decisions for building maintainable, testable, and well-organized PHP applications. These principles apply to any PHP codebase — whether web APIs, CLI tools, background workers, or libraries — using any modern framework or none at all.

## Table of Contents

1. [Class Design Principles](#class-design-principles)
2. [Service Extraction](#service-extraction)
3. [Result Objects](#result-objects)
4. [Separation of Concerns](#separation-of-concerns)
5. [Dependency Injection](#dependency-injection)
6. [Code Quality Principles](#code-quality-principles)
7. [Performance Optimization](#performance-optimization)
8. [Error Handling](#error-handling)
9. [CLI-Specific Patterns](#cli-specific-patterns)
10. [Framework-Specific Notes](#framework-specific-notes)
11. [Directory Structure Examples](#directory-structure-examples)
12. [Other Considerations](#other-considerations)

---

## Design Principles

### Single Responsibility
Every class should have one reason to change. If a class handles input parsing, business logic, and output formatting, split it.

**Before (bad — mixed concerns):**
```php
class UserController
{
    public function store(Request $request): Response
    {
        // Validation
        $data = $request->validate([
            'email' => 'required|email',
            'name' => 'required|string',
        ]);

        // Business logic
        $user = new User;
        $user->email = $data['email'];
        $user->name = $data['name'];
        $user->password = bcrypt($data['password']);
        $user->save();

        // Side effect
        Mail::to($user)->send(new WelcomeEmail($user));

        // Formatting
        return response()->json([
            'id' => $user->id,
            'email' => $user->email,
            'created_at' => $user->created_at->toIso8601String(),
        ]);
    }
    
    public function show(Request $request): Response
    {
        // ...
    }
}
```

**After (good — thin controller, rich service):**
```php
class UserController
{
    public function __construct(
        private readonly UserService $users,
    ) {}

    public function store(CreateUserRequest $request): UserResource
    {
        $user = $this->users->create($request->validated());

        return new UserResource($user);
    }
    
    public function show(Request $request): Response
    {
        // ...
    }
}
```

### Thin Layers Pattern
Every layer in your application should be thin:
- **Controllers / Commands** — Accept input, delegate, return output (≤ 1000 lines)
- **Services** — Orchestrate business operations (≤ 250 lines)
- **Repositories / Query Builders** — Abstract data access
- **Value Objects / DTOs** — Encapsulate data with validation

When any layer grows beyond its limit, extract a new class.

### Constructor Property Promotion
Use PHP 8+ constructor property promotion to reduce boilerplate:

```php
// Avoid — unnecessary repetition
class ImportService
{
    private CsvParser $parser;
    private LoggerInterface $logger;

    public function __construct(CsvParser $parser, LoggerInterface $logger)
    {
        $this->parser = $parser;
        $this->logger = $logger;
    }
}

// Good — concise
readonly class ImportService
{
    public function __construct(
        private CsvParser $parser,
        private LoggerInterface $logger,
    ) {}
}
```

### Readonly Classes vs Readonly Properties
When **all** constructor-promoted properties are `readonly`, prefer making the **entire class readonly** instead of individual properties. This reduces boilerplate and signals immutability at the class level.

**Before (bad — redundant `readonly` on every parameter):**
```php
class UserData
{
    public function __construct(
        private readonly string $name,
        private readonly string $email,
        private readonly DateTimeImmutable $createdAt,
    ) {}
}
```

**After (good — class-level readonly):**
```php
readonly class UserData
{
    public function __construct(
        private string $name,
        private string $email,
        private DateTimeImmutable $createdAt,
    ) {}
}
```

**When to use `readonly class`:**
- All properties are initialized via constructor promotion
- No property needs to be mutable after construction
- The class has no dynamic properties

**When NOT to use `readonly class`:**
- Any property needs to be mutable (e.g., counters, state tracking)
- The class has non-promoted properties that must be writable
- The class extends a non-readonly parent (PHP limitation)

### Empty Constructors
Do not allow empty `__construct()` methods with zero parameters unless the constructor is private (e.g., for singletons or static factories).

```php
// Bad — pointless
class Calculator
{
    public function __construct() {}

    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
}

// Good — no constructor needed
class Calculator
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b7s/catraca](https://github.com/b7s/catraca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
