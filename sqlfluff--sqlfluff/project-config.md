---
trigger: always_on
description: This file provides guidance for developing and extending SQL dialect support in SQLFluff.
---

# Dialect Development - AI Assistant Instructions

This file provides guidance for developing and extending SQL dialect support in SQLFluff.

## Overview

SQLFluff supports 25+ SQL dialects through an inheritance-based system. Each dialect extends the ANSI base dialect and overrides specific grammar segments to match the target SQL variant's syntax.

## Dialect Architecture

### Inheritance Hierarchy

```
ANSI (base) ← All dialects inherit from here
├── T-SQL (Microsoft SQL Server)
├── PostgreSQL
│   └── Redshift (extends PostgreSQL)
├── MySQL
│   └── MariaDB (extends MySQL)
├── BigQuery
├── Snowflake
└── ... (20+ more dialects)
```

### File Organization

```
src/sqlfluff/dialects/
├── dialect_ansi.py              # Base ANSI SQL dialect
├── dialect_tsql.py              # T-SQL (SQL Server)
├── dialect_postgres.py          # PostgreSQL
├── dialect_bigquery.py          # Google BigQuery
├── dialect_snowflake.py         # Snowflake
├── ...
├── dialect_ansi_keywords.py     # ANSI reserved/unreserved keywords
├── dialect_tsql_keywords.py     # T-SQL keywords
└── dialect_instructions/    # Per-dialect agent instructions (optional)
    ├── tsql.md
    ├── postgres.md
    └── ...
```

## Creating/Extending a Dialect

### Basic Dialect Structure

```python
"""The T-SQL (Microsoft SQL Server) dialect."""

from sqlfluff.core.dialects import load_raw_dialect
from sqlfluff.core.parser import BaseSegment
from sqlfluff.core.parser.grammar import (
    Sequence, OneOf, Ref, Bracketed, Delimited, AnyNumberOf, Optional
)

# Load parent dialect
ansi_dialect = load_raw_dialect("ansi")

# Create new dialect as copy
tsql_dialect = ansi_dialect.copy_as("tsql")

# Set keywords from separate file
tsql_dialect.sets("reserved_keywords").update([
    "CLUSTERED", "NONCLUSTERED", "ROWGUIDCOL", "TOP"
])

# Define new segments specific to T-SQL
class TopClauseSegment(BaseSegment):
    """TOP clause for T-SQL SELECT statements."""

    type = "top_clause"
    match_grammar = Sequence(
        "TOP",
        OneOf(
            Ref("NumericLiteralSegment"),
            Bracketed(Ref("ExpressionSegment")),
        ),
        Sequence("PERCENT", optional=True),
        Sequence("WITH", "TIES", optional=True),
    )

# Override existing ANSI segments
tsql_dialect.replace(
    SelectStatementSegment=Sequence(
        "SELECT",
        Ref("TopClauseSegment", optional=True),  # T-SQL addition
        Ref("SelectClauseSegment"),
        Ref("FromClauseSegment", optional=True),
        Ref("WhereClauseSegment", optional=True),
    ),
)
```

### Grammar Composition Primitives

Located in `src/sqlfluff/core/parser/grammar/`:

| Primitive | Purpose | Example |
|-----------|---------|---------|
| `Sequence()` | Ordered sequence of elements | `Sequence("SELECT", Ref("SelectClauseSegment"))` |
| `OneOf()` | Choice between alternatives | `OneOf("ASC", "DESC")` |
| `Delimited()` | Comma-separated list | `Delimited(Ref("ColumnReferenceSegment"))` |
| `AnyNumberOf()` | Zero or more repetitions | `AnyNumberOf(Ref("WhereClauseSegment"))` |
| `Bracketed()` | Content in parentheses | `Bracketed(Ref("ExpressionSegment"))` |
| `Ref()` | Reference to another segment | `Ref("TableReferenceSegment")` |
| `Optional()` | Optional element (or use `optional=True`) | `Optional(Ref("WhereClause"))` |

### Grammar Organization Patterns

#### Internal Grammar (Private Attributes with `_` prefix)

Use for grammar components specific to one statement:

```python
class CreateDatabaseStatementSegment(BaseSegment):
    """A CREATE DATABASE statement."""

    # Internal grammar - only used in this segment
    _filestream_option = OneOf(
        Sequence("NON_TRANSACTED_ACCESS", Ref("EqualsSegment"), "OFF"),
        Sequence("DIRECTORY_NAME", Ref("EqualsSegment"), Ref("QuotedLiteralSegment")),
    )

    _create_database_option = OneOf(
        Sequence("FILESTREAM", Bracketed(Delimited(_filestream_option))),
        Sequence("DEFAULT_LANGUAGE", Ref("EqualsSegment"), Ref("LanguageNameSegment")),
        Sequence("DEFAULT_FULLTEXT_LANGUAGE", Ref("EqualsSegment"), Ref("LanguageNameSegment")),
    )

    type = "create_database_statement"
    match_grammar = Sequence(
        "CREATE", "DATABASE",
        Ref("DatabaseReferenceSegment"),
        Sequence("WITH", Delimited(_create_database_option), optional=True),
    )
```

#### Shared Segments (Named Classes)

Create separate segment classes for reusable components:

```python
class FileSpecSegment(BaseSegment):
    """File specification - reusable in CREATE/ALTER statements."""

    type = "file_spec"
    match_grammar = Bracketed(
        Sequence(
            Sequence("NAME", Ref("EqualsSegment"), Ref("QuotedLiteralSegment"), optional=True),
            Sequence("FILENAME", Ref("EqualsSegment"), Ref("QuotedLiteralSegment")),
            Sequence("SIZE", Ref("EqualsSegment"), Ref("FileSizeSegment"), optional=True),
        )
    )

# Now FileSpecSegment can be used in multiple statements
class CreateDatabaseStatementSegment(BaseSegment):
    match_grammar = Sequence(
        "CREATE", "DATABASE",
        Ref("DatabaseReferenceSegment"),
        Sequence("ON", Delimited(Ref("FileSpecSegment")), optional=True),
    )

class AlterDatabaseStatementSegment(BaseSegment):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sqlfluff/sqlfluff](https://github.com/sqlfluff/sqlfluff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
