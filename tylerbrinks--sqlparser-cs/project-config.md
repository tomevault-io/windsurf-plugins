---
trigger: always_on
description: ﻿# Agent Instructions for SqlParser-CS
---

﻿# Agent Instructions for SqlParser-CS

## Project Overview

SqlParser-CS is a .NET port of the Rust [sqlparser-rs project](https://github.com/sqlparser-rs/sqlparser-rs). It's a SQL lexer and parser that generates an Abstract Syntax Tree (AST) conforming to ANSI/ISO SQL standards and various SQL dialects.

### Key Characteristics
- **Port from Rust**: Maintains functional parity with the original Rust implementation
- **Syntax-only parsing**: Focuses on syntactic analysis, not semantic validation
- **Dialect-extensible**: Supports multiple SQL dialects with customization hooks
- **Target frameworks**: .NET 8, .NET 9, .NET 10

## Architecture and Design Patterns

### Core Design Patterns

1. **Pratt Parser (TDOP)**
   - The expression parser uses Top-Down Operator Precedence parsing
   - Defined in `Parser.Expressions.cs`
   - Handles operator precedence and associativity declaratively

2. **Recursive Descent Parser**
   - Statement parsing uses traditional hand-written recursive descent
   - Defined in `Parser.Statements.cs`
   - Favored over parser generators for simplicity and debuggability

3. **Visitor Pattern**
   - Base implementation in `Ast/Visitor.cs`
   - Traverses the entire AST hierarchy
   - Provides pre/post visit hooks for Statements, Expressions, and Relations

### Project Structure

```
src/
├── SqlParser/              # Core library
│   ├── Ast/               # Abstract Syntax Tree types
│   ├── Dialects/          # SQL dialect implementations
│   ├── Tokens/            # Token definitions
│   ├── Parser.cs          # Main parser logic
│   ├── Parser.Base.cs     # Parser base methods
│   ├── Parser.Expressions.cs  # Expression parsing
│   ├── Parser.Statements.cs   # Statement parsing
│   ├── Tokenizer.cs       # Lexical analysis
│   └── SqlQueryParser.cs  # Public API entry point
├── SqlParser.Tests/       # Test suite (xUnit)
├── SqlParser.Benchmarks/  # Performance benchmarks
└── SqlParserDemo/         # Console demo application
```

## C# Conventions Specific to This Project

### Abstract Base Records with Nested Subtypes

This project uses an **unconventional but intentional pattern** to emulate Rust's discriminated unions:

```csharp
// Abstract base record
public abstract record Statement : IWriteSql, IElement
{
    // Nested concrete implementations
    public record Select(Query Query) : Statement { ... }
    public record Insert(InsertStatement Insert) : Statement { ... }
    public record Delete(DeleteStatement Delete) : Statement { ... }
}

// Usage
var statement = new Statement.Select(query);
```

**Rationalle**
- Mimics Rust's enum variants and unions as much as C# can
- Provides pseudo-namespacing for related types
- Enables pattern matching with type tests
- Maintains close alignment with Rust source

**Key types using this pattern:**
- `Statement` - Top-level SQL statements
- `Expression` - SQL expressions
- `DataType` - SQL data types
- `TableFactor` - Table references
- `SetExpression` - Set operations (UNION, INTERSECT, etc.)

### Code Style

1. **Record Types Everywhere**
   - Use `record` for immutable AST nodes
   - Use `record struct` for small value types
   - Leverage positional syntax for concise definitions

2. **File Organization**
   - One primary type per file
   - Nested types stay with their parent
   - Large types split into partial classes (e.g., `Dialect.cs` + `Dialect.Props.cs`)

3. **Naming Conventions**
   - PascalCase for types, properties, and methods
   - Avoid abbreviations except well-known ones (SQL, AST, etc.)
   - Match Rust naming where possible for maintainability

4. **ReSharper Suppressions**
   - Common suppressions at file level:
     ```csharp
     // ReSharper disable StringLiteralTypo
     // ReSharper disable CommentTypo
     ```
   - Used to handle SQL keywords and dialect-specific terms

## Working with Dialects

### Creating Custom Dialects

All dialects inherit from `Dialect` abstract class:

```csharp
public class MyCustomDialect : Dialect
{
    public override bool IsIdentifierStart(char ch) => ...;
    public override bool IsIdentifierPart(char ch) => ...;
    
    // Optional: Override parsing behavior
    public override Statement? ParseStatement(Parser parser) => ...;
    public override Expression? ParsePrefix(Parser parser) => ...;
    public override Expression? ParseInfix(Parser parser, Expression expr, int precedence) => ...;
}
```

**Key extension points:**
- `IsIdentifierStart/Part` - Define valid identifier characters
- `IsDelimitedIdentifierStart` - Define quote characters
- `ParseStatement` - Custom statement syntax
- `ParsePrefix/Infix` - Custom expression operators
- `GetNextPrecedence` - Custom operator precedence

### Supported Dialects

Located in `src/SqlParser/Dialects/`:
- GenericDialect (baseline)
- AnsiDialect
- BigQueryDialect
- ClickHouseDialect
- DatabricksDialect
- DuckDbDialect
- HiveDialect
- MsSqlDialect
- MySqlDialect
- OracleDialect
- PostgreSqlDialect
- RedshiftDialect
- SnowflakeDialect
- SQLiteDialect

## Testing Guidelines

### Test Framework: xUnit

All tests inherit from `ParserTestBase`:

```csharp
public class MyDialectTests : ParserTestBase
{
    [Fact]
    public void Parse_Custom_Syntax()
    {
        var sql = "SELECT * FROM table";
        var statement = VerifiedStatement(sql);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TylerBrinks/SqlParser-cs](https://github.com/TylerBrinks/SqlParser-cs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
