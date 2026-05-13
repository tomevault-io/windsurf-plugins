---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zeta is a composable, type-safe, async-first validation framework for .NET inspired by Zod. It uses schema-first validation with a Result pattern (no exceptions for validation failures).

## Build and Test Commands

```bash
# Build the entire solution
dotnet build

# Run all tests
dotnet test

# Run a single test by name
dotnet test --filter "FullyQualifiedName~StringSchemaTests.Email_ValidEmail_Succeeds"

# Run tests in a specific project
dotnet test tests/Zeta.Tests

# Run benchmarks
dotnet run --project benchmarks/Zeta.Benchmarks -c Release

# Run the sample API
dotnet run --project samples/Zeta.Sample.Api
```

## Architecture

### Solution Structure
- `src/Zeta/` - Core validation library (no dependencies)
- `src/Zeta.AspNetCore/` - ASP.NET Core integration (Minimal APIs and Controllers)
- `tests/Zeta.Tests/` - Core library tests (xUnit)
- `tests/Zeta.AspNetCore.Tests/` - ASP.NET integration tests
- `samples/Zeta.Sample.Api/` - Example API demonstrating usage
- `benchmarks/Zeta.Benchmarks/` - Performance benchmarks vs FluentValidation/DataAnnotations

### Core Types (src/Zeta/Core/)
- `ISchema<T>` - Contextless validation interface. Returns `Result<T>`.
- `ISchema<T, TContext>` - Context-aware validation interface (separate, no inheritance from `ISchema<T>`). Returns `Result`.
- `Result<T>` - Discriminated result type with `IsSuccess`, `Value`, `Errors`, and monadic operations (`Map`, `Then`, `Match`)
- `ValidationError(Path, Code, Message)` - Error record with JSONPath (e.g., `$.user.address.street`)
- `ValidationContext<TData>` - Contains typed context data, path tracking, CancellationToken & TimeProvider
- `ValidationContext` - Contains path tracking, CancellationToken & TimeProvider

### Rule System (src/Zeta/Rules/)
- `IValidationRule<T>` - Context-free async validation rule using `ValidationContext`
- `IValidationRule<T, TContext>` - Context-aware async validation rule using `ValidationContext<TContext>`
- `RuleEngine<T>` - Executes context-free rules for contextless schemas
- `ContextRuleEngine<T, TContext>` - Executes context-aware rules
- `DelegateValidationRule<T>` / `DelegateValidationRule<T, TContext>` - Delegate wrappers for inline rules

### Static Validators (src/Zeta/Validation/)
Shared validation logic used by both contextless and context-aware schemas:
- `StringValidators` - MinLength, MaxLength, Email, Url, Regex, etc.
- `NumericValidators` - Min, Max, Positive, Negative, Precision, etc.
- `CollectionValidators` - MinLength, MaxLength, NotEmpty for arrays/lists

### Schema Types (src/Zeta/Schemas/)
All schemas are created via the static `Z` class entry point as contextless schemas:
- `Z.String()` returns `StringSchema` which implements `ISchema<string>`
- Use `.Using<TContext>()` to promote to context-aware when needed

Schema types: `StringSchema`, `IntSchema`, `DoubleSchema`, `DecimalSchema`, `BoolSchema`, `GuidSchema`, `DateTimeSchema`, `DateOnlySchema`, `TimeOnlySchema`, `ObjectSchema`, `CollectionSchema`

### Key Patterns

**Fluent Method Chaining**: Schemas return `this` from validation methods for chaining:
```csharp
Z.String().MinLength(3).MaxLength(100).Email()
```

**ObjectSchema Field Validation**: Uses expression trees to extract property names, auto-camelCases them for error paths. Use fluent inline builders for most fields:
```csharp
// Default: Fluent inline builders
Z.Object<User>()
    .Field(u => u.Email, s => s.Email().MinLength(5))  // Error path: "$.email"
    .Field(u => u.Age, s => s.Min(18).Max(100))
    .Field(u => u.Price, s => s.Positive().Precision(2))

// Nullable value types — null skips validation automatically
Z.Object<User>()
    .Field(u => u.OptionalAge, s => s.Min(0).Max(120))     // int? — null OK
    .Field(u => u.OptionalBalance, s => s.Positive())        // decimal? — null OK
    .Field(u => u.Bio, s => s.MaxLength(500).Nullable())     // string? — call .Nullable()

// Supported types: string, int, double, decimal, bool, Guid, DateTime, DateOnly, TimeOnly

// For composability: pre-built schemas when reusing across multiple objects
var addressSchema = Z.Object<Address>()
    .Field(a => a.Street, s => s.MinLength(3))
    .Field(a => a.ZipCode, s => s.Regex(@"^\d{5}$"));

Z.Object<User>()
    .Field(u => u.Email, s => s.Email())
    .Field(u => u.Address, addressSchema)  // Reuse nested schema, path: "$.address.street"

// Collection fields with .Each() for element validation (RFC 003)
Z.Object<User>()
    .Field(u => u.Roles, roles => roles
        .Each(r => r.MinLength(3).MaxLength(50))  // Validate each string element
        .MinLength(1).MaxLength(10))  // Validate collection size

// Standalone collections
Z.Collection<string>()  // Parameterless, generic type parameter
    .Each(s => s.Email())  // Apply validation to each element
    .MinLength(1)          // Collection-level validation

// For complex nested objects, pass a pre-built schema
var orderItemSchema = Z.Object<OrderItem>()
    .Field(i => i.ProductId, s => s /* Guid validation */)
    .Field(i => i.Quantity, s => s.Min(1));


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sonberg/zeta](https://github.com/Sonberg/zeta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
