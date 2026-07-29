---
trigger: always_on
description: Guidelines for contributing to FluentMigrator - a database migration framework for .NET
---


# FluentMigrator Development Guidelines

FluentMigrator is a migration framework for .NET much like Ruby on Rails Migrations. It provides a structured way to evolve database schemas across multiple database providers.

## Project Overview

FluentMigrator supports multiple database providers including:
- SQL Server
- PostgreSQL
- MySQL
- SQLite
- Oracle
- Firebird
- Snowflake
- SAP HANA
- Redshift
- DB2

## C# Language Guidelines

- Use the latest C# features (currently C# 13) when appropriate
- Apply code-formatting style defined in `.editorconfig`
- Insert a newline before the opening curly brace of any code block (per `.editorconfig`)
- Use pattern matching and switch expressions wherever possible
- Use `nameof` instead of string literals when referring to member names
- Write clear and concise XML doc comments for all public APIs
  - Include `<example>` and `<code>` blocks when applicable
  - Document parameters, returns, exceptions, and remarks

## Naming Conventions

- Follow PascalCase for class names, method names, and public members
- Use camelCase for private fields with underscore prefix (`_camelCase`)
- Prefix interface names with "I" (e.g., `IMigrationProcessor`)
- Use PascalCase for constant fields
- Use PascalCase for static fields

## Nullable Reference Types

- Declare variables non-nullable by default
- Check for `null` at entry points
- Always use `is null` or `is not null` instead of `== null` or `!= null`
- Trust C# null annotations and avoid redundant null checks

## Migration Development Patterns

### Migration Structure

- Each migration should inherit from `Migration` or `AutoReversingMigration`
- Use the `[Migration(version)]` attribute with a timestamp-based version number (e.g., `[Migration(20090906205342)]`)
- Implement both `Up()` and `Down()` methods for reversibility
- Use `AutoReversingMigration` when the framework can automatically generate `Down()` from `Up()` operations

### Example Migration Pattern

```csharp
[Migration(20090906205342)]
public class AddUsersTable : Migration
{
    public override void Up()
    {
        Create.Table("Users")
            .WithIdColumn()
            .WithColumn("Name").AsString().NotNullable()
            .WithColumn("Email").AsString().NotNullable();
    }

    public override void Down()
    {
        Delete.Table("Users");
    }
}
```

### Database-Agnostic Code

- Write migrations to be database-agnostic when possible
- Use the `IfDatabase()` method to provide database-specific implementations when needed
- Test against multiple database providers to ensure compatibility
- Use `ProcessorIdConstants` for database identification

```csharp
IfDatabase(ProcessorIdConstants.SqlServer)
    .Create.Index("IX_Users").OnTable("Users")
        .OnColumn("Name").Ascending()
        .WithOptions().NonClustered()
        .Include("Login");

IfDatabase(processorId => !processorId.Contains(ProcessorIdConstants.SqlServer))
    .Create.Index("IX_Users").OnTable("Users")
        .OnColumn("Name").Ascending();
```

### Common Migration Operations

- Use fluent interface methods like `Create.Table()`, `Alter.Table()`, `Delete.Table()`
- Use extension methods like `.WithIdColumn()` and `.WithTimeStamps()` from `FluentMigrator.SqlServer`
- Leverage `Execute.Sql()` for complex operations that don't have fluent equivalents
- Use `Insert.IntoTable()` for data seeding when appropriate

## Testing Guidelines

### Test Organization

- Unit tests go in `FluentMigrator.Tests/Unit`
- Integration tests go in `FluentMigrator.Tests/Integration`
- Use NUnit framework with `[TestFixture]` and `[Test]` attributes
- Categorize tests using `[Category("Integration")]` and database-specific categories like `[Category("SqlServer")]`

### Test Patterns

- Use Shouldly for assertions (e.g., `result.ShouldBe(expected)`)
- Do NOT emit "Act", "Arrange" or "Assert" comments in tests
- Follow existing test naming conventions in nearby files
- Test both `Up()` and `Down()` migration methods
- Test migrations against actual database providers for integration tests
- Use `TestCaseSource` for parameterized tests across multiple database providers

### Example Test Pattern

```csharp
[TestFixture]
[Category("Integration")]
[Category("SqlServer")]
public class SqlServerMigrationTests
{
    [Test]
    public void CanCreateTable()
    {
        // Setup code
        var runner = CreateRunner();
        
        // Execute migration
        runner.Up(new CreateUsersTable());
        
        // Verify
        TableExists("Users").ShouldBe(true);
    }
}
```

## Processor and Generator Development

### Processors

- Processors execute migrations against specific database providers
- Inherit from appropriate base classes (e.g., `GenericProcessorBase`)
- Implement provider-specific SQL generation and execution
- Handle database-specific quirks and limitations
- Always properly dispose of database connections and commands

### Generators

- Generators create SQL statements from migration expressions
- Inherit from `GeneratorBase` or database-specific generator base classes
- Override methods to generate provider-specific SQL syntax
- Use quoters for proper identifier quoting
- Handle database-specific data types, index options, and constraints


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluentmigrator/fluentmigrator](https://github.com/fluentmigrator/fluentmigrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
