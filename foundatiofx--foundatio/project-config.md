---
trigger: always_on
description: You are an expert .NET engineer working on Foundatio, a production-grade library used by thousands of developers. Your changes must maintain backward compatibility, performance, and reliability. Approach each task methodically: research existing patterns, make surgical changes, and validate thoroughly.
---

# Agent Guidelines for Foundatio

You are an expert .NET engineer working on Foundatio, a production-grade library used by thousands of developers. Your changes must maintain backward compatibility, performance, and reliability. Approach each task methodically: research existing patterns, make surgical changes, and validate thoroughly.

**Craftsmanship Mindset**: Every line of code should be intentional, readable, and maintainable. Write code you'd be proud to have reviewed by senior engineers. Prefer simplicity over cleverness. When in doubt, favor explicitness and clarity.

## Repository Overview

Foundatio provides pluggable building blocks for distributed .NET applications:

- **Caching** (`ICacheClient`) - In-memory, Redis, hybrid caching with expiration
- **Queues** (`IQueue<T>`) - Message queuing with work items, delays, retries
- **Locks** (`ILockProvider`) - Distributed locking for coordination
- **Messaging** (`IMessageBus`) - Pub/sub messaging between services
- **Jobs** (`IJob`) - Background job processing with scheduling
- **Storage** (`IFileStorage`) - Unified file storage abstraction
- **Resilience** - Retry policies, circuit breakers, rate limiting

Design principles: **interface-first**, **testable**, **swappable implementations**, **in-memory + external providers**.

## Quick Start

```bash
# Build
dotnet build Foundatio.slnx

# Test
dotnet test Foundatio.slnx

# Format code
dotnet format Foundatio.slnx
```

**Note**: When building within a workspace, use `Foundatio.All.slnx` instead to include all Foundatio projects in the build and test cycle.

## Project Structure

```text
src
├── Foundatio                         # Core abstractions and in-memory implementations
│   ├── Caching                       # ICacheClient - caching with expiration
│   ├── Queues                        # IQueue<T> - message queuing
│   ├── Lock                          # ILockProvider - distributed locking
│   ├── Messaging                     # IMessageBus - pub/sub messaging
│   ├── Jobs                          # IJob - background job processing
│   ├── Storage                       # IFileStorage - unified file API
│   ├── Resilience                    # Retry policies, circuit breakers
│   ├── Serializer                    # ISerializer abstractions
│   └── Extensions                    # Extension methods for DI and utilities
├── Foundatio.Extensions.Hosting      # ASP.NET Core hosting integration
└── Foundatio.TestHarness             # Shared test base classes
tests
└── Foundatio.Tests                   # Unit and integration tests
samples                               # Sample applications
benchmarks                            # Performance benchmarks
docs                                  # Documentation site
```

## Coding Standards

### Style & Formatting

- Follow `.editorconfig` rules and [Microsoft C# conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- Run `dotnet format` to auto-format code
- Match existing file style; minimize diffs
- No code comments unless necessary—code should be self-explanatory

### Architecture Patterns

- **Interface-first design**: All core features expose interfaces (`ICacheClient`, `IQueue<T>`, `IFileStorage`)
- **Dependency Injection**: Use constructor injection; extend via `IServiceCollection` extensions
- **In-memory implementations**: Always provide for testing and local development
- **Naming**: `Foundatio.[Feature]` for projects, `I[Feature]` for interfaces
- **External providers**: Redis, Azure, AWS implementations live in separate repositories

### Code Quality

- Write complete, runnable code—no placeholders, TODOs, or `// existing code...` comments
- Use modern C# features: pattern matching, nullable references, `is` expressions, target-typed `new()`
- Follow SOLID, DRY principles; remove unused code and parameters

### Nullable Reference Types (NRT) Conventions

The codebase has NRT enabled. Follow these patterns:

- **Entity/model properties** (`= null!`): Acceptable for properties set by deserialization or ORM frameworks. Prefer `required` keyword where construction is controlled.
- **Expression tree `!`**: Required when expressions like `SortDescending(x => x.DateCreatedUtc!)` box `DateTime?` to `object?` but the delegate expects `object`. The value is never evaluated -- only used for field path extraction. This is correct and unavoidable.
- **`.Where(x => x is not null).Select(x => x!)`**: The compiler can't prove non-null through a lambda boundary after filtering. This is the idiomatic post-filter pattern.
- **`null!` in test `Assert.Throws` calls**: Intentional contract violation to test null guards.
- **`[return: MaybeNull]` with `return default!`**: Idiomatic pattern for unconstrained generic methods that may return null (e.g., `Deserialize<T>`).
- **Vendored code** (`#nullable disable`): Leave internalized third-party code (FastCloner, Nito) as-is. Ensure public API boundaries have proper null checks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FoundatioFx/Foundatio](https://github.com/FoundatioFx/Foundatio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
