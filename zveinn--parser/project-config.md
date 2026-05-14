---
trigger: always_on
description: Handles complex nested structures:
---

# Parser Library - Technical Documentation

## Project Overview

**Parser** is a sophisticated Go library that provides SQL-like query capabilities for filtering struct slices without requiring a database. It implements a complete query language parser, lexer, and evaluator using Go's reflection and generics for type-safe operations.

## Architecture

### Core Components

#### 1. **Lexer System** (`enhanced_lexer.go`)
The enhanced lexer tokenizes query strings into meaningful tokens:
- Handles negative numbers, scientific notation, and comma-separated numbers
- Supports quoted strings with escape sequences
- Processes operators and keywords case-insensitively
- Manages complex numeric formats (e.g., `1.5e4`, `-100.5`, `1,000,000`)

**Key Token Types:**
- Literals: `IDENTIFIER`, `STRING`, `NUMBER`
- Operators: `EQ`, `NE`, `LT`, `GT`, `GE`, `LE`
- Logical: `AND`, `OR`, `NOT`
- Special: `CONTAINS`, `IS NULL`, `ANY`, `COMMA`, `LPAREN`, `RPAREN`

#### 2. **Parser** (`parser.go:851-1200`)
Converts token stream into an Abstract Syntax Tree (AST):
- Implements recursive descent parsing
- Handles operator precedence (OR has lower precedence than AND)
- Supports parenthetical grouping
- Generates detailed error messages for syntax errors

**AST Node Types:**
```go
- ComparisonExpression: Field comparisons (e.g., `Age > 25`)
- ConjunctionExpression: AND operations
- OrExpression: OR operations
- NotExpression: NOT operations
- IsNullExpression: NULL checks
- AnyExpression: ANY operations for multi-value checks
```

#### 3. **Evaluator** (`parser.go:305-845`)
Evaluates AST against struct data using reflection:
- Traverses nested struct fields and maps
- Handles pointer dereferencing automatically
- Supports slice operations and containment checks
- Case-insensitive field matching

#### 4. **Humanized Value Parser** (`parser.go:1420-1675`)
Converts human-readable values to numeric equivalents:
- **Time durations**: `30s`, `5m`, `2h30m` → seconds
- **Byte sizes**: `10GB`, `8GiB`, `500MB` → bytes
- **SI prefixes**: `1.5K`, `2.3M` → numeric values
- **Comma-separated**: `1,000,000` → 1000000

**Priority Resolution Order:**
1. Time units (highest priority)
2. Byte size units
3. SI prefixes (uppercase only)
4. Comma-separated numbers

### API Functions

#### Main Functions

**`Parse[T any](query string, data []T) ([]T, error)`**
- Generic function that filters a slice of structs
- Returns matching items
- Type-safe through generics

**`ParseEncoder[T any](query string, data []T, encoder func(v any) error) (int, error)`**
- Variant that streams results through an encoder
- Avoids memory allocation for results slice
- Returns count of matched items
- Useful for large datasets or streaming responses

**`GetExclusiveField(query, param string) (bool, string)`** (`query_extractor.go`)
- Extracts parameter values from queries
- Only returns values in exclusive AND contexts
- Useful for query optimization and caching

### Query Language Syntax

#### Operators

| Type | Operators | Example |
|------|----------|---------|
| Comparison | `=`, `!=`, `<`, `>`, `<=`, `>=` | `Age > 25` |
| String/Slice | `CONTAINS` | `Skills CONTAINS 'Go'` |
| Logical | `AND`, `OR`, `NOT` | `Age > 25 AND Name = 'Alice'` |
| Null Check | `IS NULL`, `IS NOT NULL` | `Department IS NOT NULL` |
| Multi-value | `ANY` | `ANY(Skills) = ANY('Go', 'Rust')` |

#### Field Access
- **Direct fields**: `Name`, `Age`
- **Nested structs**: `Department.Name`, `Department.Location`
- **Map values**: `Tags.level`, `Config.timeout`
- **Case-insensitive fields**: `name`, `NAME`, `Name` all work

#### String Value Comparisons
All string comparisons are **case-insensitive**:
- **Equality**: `Name = 'alice'` matches "Alice", "ALICE", "alice"
- **Inequality**: `Brand != 'samsung'` excludes all case variations
- **Ordering**: Uses case-insensitive comparison (`strings.ToLower`)
- **Contains**: `Description CONTAINS 'phone'` matches "Phone", "PHONE", etc.
- **Arrays**: `Tags CONTAINS 'premium'` matches any case variation in the array

#### Value Formats
- **Strings**: `'Alice'`, `'Bob\'s House'` (escaped quotes)
- **Numbers**: `42`, `-10`, `3.14`, `1.5e4`
- **Booleans**: `true`, `false`
- **Humanized**: `30s`, `10GB`, `1.5K`, `1,000,000`

### Internal Implementation Details

#### Reflection Strategy (`parser.go:198-284`)
The `getFieldValues` function implements sophisticated field traversal:
1. Splits field path by dots
2. Handles pointers and interfaces transparently
3. Traverses slices, returning multiple values
4. Supports map access with string keys
5. Flattens nested slices at leaf nodes

#### Performance Optimizations

1. **Short-circuit Evaluation**: OR/AND expressions stop early when result is determined
2. **Minimal Allocations**: Reuses reflection values where possible
3. **Efficient Lexing**: Single-pass tokenization with lookahead
4. **Slice Preallocation**: `results = make([]T, 0, len(data))`

#### Error Handling
- **Parse Errors**: Syntax errors with position information
- **Evaluation Errors**: Field not found, type mismatches
- **Detailed Messages**: Includes context and suggestions
- **Error Recovery**: Parser attempts to continue after errors

### File Structure

```
parser/
├── parser.go               # Core parsing and evaluation logic (1676 lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zveinn/parser](https://github.com/zveinn/parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
