---
trigger: always_on
description: You are an expert .NET engineer working on Foundatio.Repositories, a production-grade repository pattern library built on top of Foundatio. This library is used by developers to build robust data access layers with Elasticsearch. Your changes must maintain backward compatibility, performance, and reliability. Approach each task methodically: research existing patterns, make surgical changes, and validate thoroughly.
---

# Agent Guidelines for Foundatio.Repositories

You are an expert .NET engineer working on Foundatio.Repositories, a production-grade repository pattern library built on top of Foundatio. This library is used by developers to build robust data access layers with Elasticsearch. Your changes must maintain backward compatibility, performance, and reliability. Approach each task methodically: research existing patterns, make surgical changes, and validate thoroughly.

**Craftsmanship Mindset**: Every line of code should be intentional, readable, and maintainable. Write code you'd be proud to have reviewed by senior engineers. Prefer simplicity over cleverness. When in doubt, favor explicitness and clarity.

## Repository Overview

Foundatio.Repositories provides a generic repository pattern implementation built on Foundatio building blocks:

- **Repository Pattern** (`IRepository<T>`, `IReadOnlyRepository<T>`) - CRUD operations with async events
- **Searchable Repositories** (`ISearchableRepository<T>`) - Dynamic querying with Foundatio.Parsers
- **Elasticsearch Implementation** - Full-featured Elasticsearch repository with index management
- **Caching Integration** - Real-time cache invalidation with distributed cache support
- **Message Bus Integration** - Entity change notifications for real-time applications
- **Patch Operations** - JSON patch, partial document, and script-based updates
- **Soft Deletes** - Built-in soft delete support with query filtering
- **Document Versioning** - Optimistic concurrency with version tracking
- **Index Management** - Schema versioning, daily/monthly strategies, migrations
- **Jobs** - Index maintenance, snapshots, reindexing

Design principles: **interface-first**, **built on Foundatio primitives**, **Elasticsearch-optimized**, **testable with in-memory implementations**.

## Quick Start

```bash
# Build
dotnet build Foundatio.Repositories.slnx

# Test shared repository tests
dotnet test tests/Foundatio.Repositories.Tests/Foundatio.Repositories.Tests.csproj

# Test Elasticsearch implementation (requires running Elasticsearch)
docker compose up -d
dotnet test tests/Foundatio.Repositories.Elasticsearch.Tests/Foundatio.Repositories.Elasticsearch.Tests.csproj

# Format code
dotnet format Foundatio.Repositories.slnx
```

**Note**: When building within a workspace, use `Foundatio.All.slnx` instead to include all Foundatio projects in the build and test cycle.

## Project Structure

```text
src
├── Foundatio.Repositories             # Core repository abstractions
│   ├── Exceptions                     # Repository-specific exceptions
│   ├── Extensions                     # Extension methods for repositories
│   ├── JsonPatch                      # JSON patch operation support
│   ├── Migration                      # Document migration infrastructure
│   ├── Models                         # Entity interfaces and base models
│   ├── Options                        # Command and query options
│   ├── Queries                        # Query builders and interfaces
│   └── Utility                        # ObjectId, helpers
└── Foundatio.Repositories.Elasticsearch  # Elasticsearch implementation
    ├── Configuration                  # Index configuration and mappings
    ├── CustomFields                   # Dynamic field support
    ├── Extensions                     # Elasticsearch-specific extensions
    ├── Jobs                           # Maintenance and migration jobs
    ├── Options                        # Elasticsearch-specific options
    ├── Queries                        # Query builders for Elasticsearch
    ├── Repositories                   # Base repository implementations
    └── Utility                        # Elasticsearch utilities
tests
├── Foundatio.Repositories.Tests       # Core repository unit tests
└── Foundatio.Repositories.Elasticsearch.Tests  # Elasticsearch integration tests
    └── Repositories                   # Test repository implementations
        ├── Configuration              # Test index configurations
        ├── Models                     # Test entity models
        └── Queries                    # Test query implementations
samples
└── Foundatio.SampleApp                # Sample Blazor application
    ├── Client                         # Blazor WebAssembly client
    ├── Server                         # ASP.NET Core server with repositories
    └── Shared                         # Shared models
```

## Coding Standards

### Style & Formatting

- Follow `.editorconfig` rules and [Microsoft C# conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- Run `dotnet format` to auto-format code
- Match existing file style; minimize diffs
- No code comments unless necessary—code should be self-explanatory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FoundatioFx/Foundatio.Repositories](https://github.com/FoundatioFx/Foundatio.Repositories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
