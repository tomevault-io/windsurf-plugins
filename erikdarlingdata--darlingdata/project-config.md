---
trigger: always_on
description: This document outlines the T-SQL coding style preferences for Erik Darling (Darling Data, LLC) and must be strictly followed when writing or modifying SQL code.
---

# Erik Darling's T-SQL Coding Style Guide

This document outlines the T-SQL coding style preferences for Erik Darling (Darling Data, LLC) and must be strictly followed when writing or modifying SQL code.

## Project Constraints

- **No Additional Dependencies**: Do not create helper functions, procedures, views, or any other database objects as dependencies for stored procedures. All logic must be self-contained within the procedure itself to avoid deployment and dependency management complexity.

## General Formatting

- **Keywords**: All SQL keywords in UPPERCASE (SELECT, FROM, WHERE, JOIN, etc.)
- **Functions**: All SQL functions in UPPERCASE (CONVERT, ISNULL, OBJECT_ID, etc.)
- **Data types**:
  - Never abbreviate data types (use INTEGER instead of INT)
  - All data types must be lowercase (varchar, nvarchar, datetime2, bigint, etc.)
  - Length specifications must also be lowercase: nvarchar(max), not nvarchar(MAX)
  - Precision and scale specifications must be lowercase: decimal(38,2), not DECIMAL(38,2)
  - Always use sysname for SQL Server object names (database names, table names, schema names, column names, index names, etc.) rather than nvarchar(128)
- **Keywords**: Never abbreviate keywords (use EXECUTE instead of EXEC, TRANSACTION instead of TRAN, PROCEDURE instead of PROC)
- **Indentation**: 4 spaces for each level of indentation (NEVER use tabs)
- **Line breaks**: Each statement on a new line
- **Spacing**: Consistent spacing around operators (=, <, >, etc.)
- **Block separation**: Empty line between logical code blocks (maximum of two empty lines between statements)
- **Quotes**: Use single quotes for string literals and N-prefix for Unicode strings (N'string')
- **TOP syntax**: Always include parentheses, as in TOP (100) not TOP 100
- **Object creation**: Generally use CREATE OR ALTER for objects instead of DROP/CREATE
- **Table aliases**: Tables must always have aliases, even in simple queries
- **Column references**: Always qualify columns with their table alias
- **Commas**: Trailing commas always. 

## Comments

- Always use block comments with /* ... */ for most comments, never use double dash (--)
- Include parameter descriptions as inline comments after parameter definitions
- Use ASCII art for header blocks to visually distinguish sections
- Include copyright and attribution information in header comments
- Prefix code sections with descriptive comments about what the section does
- Use comments to describe:
  - New code blocks
  - Complex expressions
  - Table purposes
  - Complex logic
  - The logical flow of code

## Naming Conventions

- **Parameters**: Prefixed with @ and use snake_case (@database_name, @debug)
- **Variables**: Same as parameters (@database_id, @sql)
- **Temporary Tables**: Prefixed with # and use descriptive snake_case (#filtered_objects)
- **Aliases**: Short, meaningful lowercase names (ap, o, t)
- **Objects**: Use clear, descriptive names

## Query Structure

- **SELECT statements**:
  - SELECT keyword on first line
  - Column list starts on next line, indented four spaces
  - Trailing commas for multi-line column lists
  - Columns aligned vertically for readability
  - FROM clause on new line at same indent level as SELECT
  - Column aliases should always use the pattern: column_name = column_expression
    - Example: some_date = DATEADD(DAY, 1, GETDATE())
  - Always terminate queries with a semicolon

- **Table references**:
  - Always use schema prefixes for all objects except temporary objects
  - Examples: FROM dbo.objects, FROM tempdb.dbo.objects
  - Temporary tables don't need schema: FROM #temp_table

- **Table aliases**:
  - Always use the AS keyword with table aliases: table_name AS alias
  - Example: FROM dbo.sys_objects AS o

- **Windowing functions**:
  - Format with OVER on same line as function
  - PARTITION BY and ORDER BY on separate lines indented
  - Parentheses on their own lines
  ```sql
  SELECT
      n = ROW_NUMBER() OVER
          (
              PARTITION BY
                  column_name
              ORDER BY
                  other_column
          )
  ```

- **JOIN syntax**:
  - Use modern ANSI JOIN syntax (JOIN table ON condition)
  - JOIN keyword on new line at same indent level as FROM
  - ON conditions indented from JOIN
  - JOIN conditions with AND should be aligned like this:
  ```sql
  FROM dbo.table_a AS a0
  JOIN dbo.table_a AS a1
    ON  a0.col = a1.col
    AND a0.col = a1.col
  ```
  - For correlated queries and joins, the table most recently referenced should come first in the ON clause:
  ```sql
  FROM first_table AS ft0
  JOIN dbo.first_table AS ft1
    ON ft1.col = ft0.col
  ```

- **Clauses**:
  - GROUP BY, ORDER BY, and HAVING clauses should always begin on a new line, indented four spaces from the main statement
  - WHERE clauses with AND conditions should be formatted with AND aligned:
  ```sql
  WHERE a.col = 1
  AND   b.col = 2
  ```
  - EXISTS and NOT EXISTS should use this format with 1/0 in the SELECT:
  ```sql
  WHERE EXISTS
  (
      SELECT
          1/0
      FROM other_table AS ot
      WHERE ot.col = t.col
  )
  ```

- **Subqueries**:
  - Subqueries should never be one-liners

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erikdarlingdata/DarlingData](https://github.com/erikdarlingdata/DarlingData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
