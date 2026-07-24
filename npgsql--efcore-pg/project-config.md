---
trigger: always_on
description: This is the PostgreSQL provider for [Entity Framework Core](https://github.com/dotnet/efcore), built on top of [Npgsql](https://github.com/npgsql/npgsql). It translates EF Core LINQ queries into PostgreSQL SQL and handles PostgreSQL-specific features like arrays, JSON, ranges, full-text search, NodaTime, and PostGIS/NetTopologySuite.
---

# Copilot Instructions for Npgsql EF Core Provider (EFCore.PG)

This is the PostgreSQL provider for [Entity Framework Core](https://github.com/dotnet/efcore), built on top of [Npgsql](https://github.com/npgsql/npgsql). It translates EF Core LINQ queries into PostgreSQL SQL and handles PostgreSQL-specific features like arrays, JSON, ranges, full-text search, NodaTime, and PostGIS/NetTopologySuite.

## Build and Test

```bash
# Build
dotnet build

# Run all tests (requires PostgreSQL - see "Test Database" below)
dotnet test

# Run a specific test project
dotnet test test/EFCore.PG.FunctionalTests
dotnet test test/EFCore.PG.Tests

# Run a single test class
dotnet test test/EFCore.PG.FunctionalTests --filter FullyQualifiedName~NorthwindSelectQueryNpgsqlTest

# Run a single test method
dotnet test test/EFCore.PG.FunctionalTests --filter "FullyQualifiedName~NorthwindSelectQueryNpgsqlTest.Select_bool_closure"
```

### Test Database

Tests auto-start a PostgreSQL testcontainer if no connection string is configured. To use an existing PostgreSQL instance, set the environment variable:

```bash
export Test__Npgsql__DefaultConnection="Server=localhost;Username=npgsql_tests;Password=npgsql_tests"
```

Some tests require PostGIS. Set `NPGSQL_TEST_POSTGIS=true` to enforce PostGIS availability (tests fail instead of skip if PostGIS is missing).

## Architecture

### Source Projects (src/)

- **EFCore.PG**: Main PostgreSQL provider. Contains query translation, type mappings, migrations, scaffolding, and service registration.
- **EFCore.PG.NodaTime**: Plugin for NodaTime date/time types.
- **EFCore.PG.NTS**: Plugin for PostGIS spatial types via NetTopologySuite.
- **Shared/**: Utility code shared across all source projects via `<Compile Include>`.

### Test Projects (test/)

- **EFCore.PG.FunctionalTests**: Integration tests requiring a PostgreSQL database. Inherits extensively from EF Core's `Relational.Specification.Tests` base classes.
- **EFCore.PG.Tests**: Unit tests that don't require a database.

### Key Source Directories (under src/EFCore.PG/)

- **Query/ExpressionTranslators/Internal/**: LINQ-to-SQL method/member translators (e.g. `NpgsqlStringMethodTranslator`, `NpgsqlArrayMethodTranslator`). Each translates .NET method calls into PostgreSQL SQL expressions.
- **Query/Expressions/Internal/**: Custom PostgreSQL SQL expression nodes (`PgArrayIndexExpression`, `PgJsonTraversalExpression`, `PgRegexMatchExpression`, `PgILikeExpression`, `PgAnyExpression`, `PgUnnestExpression`, etc.).
- **Query/Internal/**: Query pipeline components — SQL generation (`NpgsqlQuerySqlGenerator`), translation visitors, postprocessors.
- **Storage/Internal/**: Type mapping between .NET types and PostgreSQL types (e.g. `NpgsqlStringTypeMapping`). Each mapping stores an `NpgsqlDbType` and configures `NpgsqlParameter` at execution time.
- **Migrations/**: Migration SQL generation, including PostgreSQL-specific DDL (extensions, enums, sequences).
- **Extensions/**: Extension methods for configuring the provider (`UseNpgsql()`) and PostgreSQL-specific model building (e.g. `HasPostgresExtension()`, `HasPostgresEnum()`).

### Plugin Architecture

NodaTime and NTS plugins hook into the main provider via EF Core's plugin interfaces (`IMethodCallTranslatorPlugin`, `IMemberTranslatorPlugin`, `IRelationalTypeMappingSourcePlugin`, etc.) registered through DI. This allows extending translations and type mappings without modifying the core provider.

## Test Patterns

### Inherited Specification Tests

Most functional tests inherit from EF Core base test classes and override methods to add PostgreSQL-specific SQL assertions:

```csharp
public class NorthwindSelectQueryNpgsqlTest : NorthwindSelectQueryRelationalTestBase<NorthwindQueryNpgsqlFixture<NoopModelCustomizer>>
{
    public override async Task Select_bool_closure(bool async)
    {
        await base.Select_bool_closure(async);

        AssertSql(
            """
SELECT c."CustomerID", c."Address", ...
FROM "Customers" AS c
""");
    }
}
```

* All inherited query tests systematically override all methods in the EF-provided base class, in order to call `AssertSql()` to assert on the exact generated SQL. Always do this when e.g. inheriting a new EF functional test class.
* Such test overrides must be placed in the same position in the file as its base test method in EF, to maintain consistency between EF and EFCore.PG. To do that, check the source for the base test class in the EF repo (https://github.com/dotnet/efcore); note that you may need to consult a specific commit in the EF repo that corresponds to what EFCore.PG is referencing.
* EF query test contains a built-in ability to rewrite the SQL inside `AssertSql()` calls to whatever the provider currently emits; to trigger this, set the `EF_TEST_REWRITE_BASELINES` env var to `1` and run any test(s) - this will modify the source code. You can then inspect the change to confirm that it makes sense, etc.

### Conditional Execution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [npgsql/efcore.pg](https://github.com/npgsql/efcore.pg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
