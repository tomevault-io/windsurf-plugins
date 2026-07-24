---
trigger: always_on
description: These rules guide GitHub Copilot in this repo so generated code, tests, and docs fit our stack and conventions. This document is optimized to help Copilot provide better support for feature development, bug fixing, and testing in the Thinktecture.EntityFrameworkCore repository.
---

# GitHub Copilot Instructions

These rules guide GitHub Copilot in this repo so generated code, tests, and docs fit our stack and conventions. This document is optimized to help Copilot provide better support for feature development, bug fixing, and testing in the Thinktecture.EntityFrameworkCore repository.

## Quick Start for Development Tasks

### For Feature Development
1. **Understand scope**: Identify affected packages in `src/**` and corresponding tests in `tests/**`
2. **Test-first approach**: Write failing tests before implementation
3. **Minimal public surface**: Keep APIs `internal` unless cross-package usage is needed
4. **Server-evaluable queries**: Ensure EF Core operations stay on the server side
5. **Async + CancellationToken**: Always provide async APIs for I/O operations

### For Bug Fixes
1. **Reproduce first**: Create a failing test demonstrating the bug
2. **Minimal fix**: Implement the smallest change to pass the test
3. **Regression protection**: Ensure adequate test coverage for the fix
4. **Multi-TFM validation**: Test on both `net8.0` and `net9.0`

### For Test Writing
1. **Mirror structure**: Tests follow `tests/<Package>.Tests/**` structure
2. **Naming convention**: `MethodName_Should_DoSomething_When_Condition`
3. **Integration patterns**: Use `IntegrationTestsBase` for SQL Server, appropriate base for SQLite
4. **Deterministic**: Avoid time-based operations, use predictable test data

## Repo Overview

- **Purpose**: Extensions for Entity Framework Core (performance features, SQL Server/SQLite helpers, test utilities)
- **Packages**: Relational, SqlServer, Sqlite, BulkOperations, and Testing variants
- **Structure**:
  - Runtime code: `src/**`
  - Tests: `tests/**`
  - Samples/benchmarks: `samples/**`

## Tech Stack and Targets

- **.NET**: net8.0 and net9.0
- **C#**: LangVersion 13.0 (nullable enabled, implicit usings enabled)
- **EF Core**: 9.x (Microsoft.EntityFrameworkCore.*: 9.0.8)
- **Testing**: xUnit + AwesomeAssertions + NSubstitute + Serilog XUnit sink
- **Logging**: Serilog with XUnit sink for tests/samples
- **Package management**: Central versions in `Directory.Packages.props`
- **Key dependencies**: Microsoft.Data.SqlClient 5.2.3, BenchmarkDotNet 0.15.2, Testcontainers.MsSql 4.6.0

## Core Conventions Copilot Must Follow

### Code Style and Structure
- **Namespaces**: Start with `Thinktecture` and match folder structure
- **Nullable reference types**: Enabled; use explicit null checks with `ArgumentNullException.ThrowIfNull(...)`
- **Async first**: Provide async APIs with `CancellationToken` for I/O operations; avoid sync-over-async
- **Visibility**: Keep public surface minimal; prefer `internal` unless used across packages
- **File organization**: Use file-scoped namespaces, expression-bodied members where appropriate
- **Immutability**: Use `readonly` fields/structs where intended; prefer `record` for value-like objects

### EF Core Specific Patterns
- **Query composition**: Keep operations server-evaluable; avoid client evaluation; prefer `IQueryable` over early `ToList()`
- **SQL safety**: Never concatenate raw SQL; use `FromSqlInterpolated` or existing parameterized helpers
- **Read-only queries**: Use `AsNoTracking()` unless state tracking is needed
- **Transactions**: Use EF Core `IDbContextTransaction`/`ExecutionStrategy` patterns
- **Provider specifics**:
  - **SQL Server**: Use/extend existing abstractions for table hints, temp tables, bulk operations
  - **SQLite**: Respect provider limitations; add conditional code paths for different behavior

## Key Architecture Patterns

### Bulk Operations Architecture
- **Executors**: Implement `IBulkInsertExecutor`, `IBulkUpdateExecutor`, `IBulkInsertOrUpdateExecutor`, `ITruncateTableExecutor`
- **SQL Server implementation**: `SqlServerBulkOperationExecutor` uses `SqlBulkCopy` for inserts and MERGE statements for updates
- **Context factories**: `ISqlServerBulkOperationContextFactory` creates contexts with readers and connection management
- **Options pattern**: Strongly-typed options inherit from base interfaces (e.g., `SqlServerBulkInsertOptions : IBulkInsertOptions`)
- **Property providers**: Use `IEntityPropertiesProvider` for flexible property selection in bulk operations

### Bulk Operations Usage Patterns
- **Insert operations**: `BulkInsertAsync<T>()` extensions on `DbContext` with strongly-typed options
- **Update operations**: `BulkUpdateAsync<T>()` with key properties and update properties specification
- **Upsert operations**: `BulkInsertOrUpdateAsync<T>()` for MERGE-style operations
- **Temp table integration**: `BulkInsertIntoTempTableAsync<T>()` returns `ITempTableQuery<T>` for further querying
- **Value insertion**: `BulkInsertValuesIntoTempTableAsync<T>()` for inserting simple values into temp tables

### Temp Tables Architecture
- **Creation**: `ITempTableCreator` creates temp tables with `ITempTableReference` for lifecycle management
- **Querying**: `ITempTableQuery<T>` wraps `IQueryable<T>` with automatic cleanup via `IAsyncDisposable`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PawelGerr/Thinktecture.EntityFrameworkCore](https://github.com/PawelGerr/Thinktecture.EntityFrameworkCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
