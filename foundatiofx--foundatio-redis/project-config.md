---
trigger: always_on
description: You are an expert .NET engineer working on Foundatio, a production-grade library used by thousands of developers. Your changes must maintain backward compatibility, performance, and reliability. Approach each task methodically: research existing patterns, make surgical changes, and validate thoroughly.
---

# Agent Guidelines for Foundatio.Redis

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
dotnet build Foundatio.Redis.slnx

# Test
dotnet test Foundatio.Redis.slnx

# Format code
dotnet format Foundatio.Redis.slnx
```

**Note**: When building within a workspace, use `Foundatio.All.slnx` instead to include all Foundatio projects in the build and test cycle.

## Project Structure

```text
src
└── Foundatio.Redis                    # Redis provider implementations
    ├── Cache                         # ICacheClient implementations for Redis
    ├── Queues                        # IQueue<T> implementations for Redis
    ├── Messaging                     # IMessageBus implementations for Redis
    ├── Storage                       # IFileStorage implementations for Redis
    ├── Extensions                    # Extension methods for DI
    ├── Scripts                       # Redis scripts
    └── Utility                       # Utility classes
tests
└── Foundatio.Redis.Tests             # Unit and integration tests
samples                               # Sample applications
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
- Clear, descriptive naming; prefer explicit over clever
- Use `AnyContext()` (e.g., `ConfigureAwait(false)`) in library code (not in tests)
- Prefer `ValueTask<T>` for hot paths that may complete synchronously
- Always dispose resources: use `using` statements or `IAsyncDisposable`
- Handle cancellation tokens properly: check `token.IsCancellationRequested`, pass through call chains

### Common Patterns

- **Async suffix**: All async methods end with `Async` (e.g., `GetAsync`, `SetAsync`)
- **CancellationToken**: Last parameter, defaulted to `default` in public APIs
- **Extension methods**: Place in `Extensions/` directory, use descriptive class names (e.g., `CacheClientExtensions`)
- **Logging**: Use structured logging with `ILogger`, log at appropriate levels
- **Exceptions**: Use `ArgumentException.ThrowIfNullOrEmpty(parameter)` for validation. For feature-specific errors, use consistent exception types: `StorageException` for file storage operations, `CacheException` for caching operations, `MessageBusException` for messaging operations. This ensures consumers get predictable exception types regardless of the underlying implementation (Redis, Azure, AWS, etc.). Throw `ArgumentNullException`, `ArgumentException`, `InvalidOperationException` with clear messages for general validation and operation errors.

### Single Responsibility

- Each class has one reason to change
- Methods do one thing well; extract when doing multiple things
- Keep files focused: one primary type per file
- Separate concerns: don't mix I/O, business logic, and presentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FoundatioFx/Foundatio.Redis](https://github.com/FoundatioFx/Foundatio.Redis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
