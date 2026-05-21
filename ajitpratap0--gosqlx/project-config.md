---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GoSQLX is a **production-ready**, **race-free**, high-performance SQL parsing SDK for Go that provides lexing, parsing, and AST generation with zero-copy optimizations. The library is designed for enterprise use with comprehensive object pooling for memory efficiency.

**Requirements**: Go 1.26+ (upgraded from 1.23 to fix stdlib vulnerabilities; `mark3labs/mcp-go` requires 1.23)

**Production Status**: ✅ Validated for production deployment (v1.6.0+, current: v1.14.0)
- Thread-safe with zero race conditions (20,000+ concurrent operations tested)
- 1.38M+ ops/sec sustained, 1.5M peak with memory-efficient object pooling
- ~80-85% SQL-99 compliance (window functions, CTEs, set operations, MERGE, etc.)
- Multi-dialect support: PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, SQLite, Snowflake, ClickHouse (8 dialects)

## Architecture

### Core Components

- **Tokenizer** (`pkg/sql/tokenizer/`): Zero-copy SQL lexer with full UTF-8 support
- **Parser** (`pkg/sql/parser/`): Recursive descent parser with one-token lookahead
- **AST** (`pkg/sql/ast/`): Abstract Syntax Tree nodes with visitor pattern support
- **Keywords** (`pkg/sql/keywords/`): Multi-dialect SQL keyword definitions
- **Models** (`pkg/models/`): Core data structures (tokens, spans, locations)
- **Errors** (`pkg/errors/`): Structured error handling with position tracking
- **Metrics** (`pkg/metrics/`): Production performance monitoring
- **Security** (`pkg/sql/security/`): SQL injection detection with severity classification
- **Linter** (`pkg/linter/`): SQL linting engine with 30 built-in rules (L001-L030)
- **LSP** (`pkg/lsp/`): Language Server Protocol for IDE integration
- **GoSQLX** (`pkg/gosqlx/`): High-level simple API (recommended for most users)
- **Compatibility** (`pkg/compatibility/`): API stability testing

### Token Processing Pipeline

```
Raw SQL bytes → tokenizer.Tokenize() → []models.TokenWithSpan
             → parser.ParseFromModelTokens() → *ast.AST
```

### Object Pooling (Critical for Performance)

The codebase uses extensive sync.Pool for all major data structures:
- `ast.NewAST()` / `ast.ReleaseAST()` - AST container
- `tokenizer.GetTokenizer()` / `tokenizer.PutTokenizer()` - Tokenizer instances
- Individual pools for SELECT, INSERT, UPDATE, DELETE statements
- Expression pools for identifiers, binary expressions, literals

### Module Dependencies

Clean hierarchy with minimal coupling (verified against production imports):
```
# Core parsing chain
models     → (no deps)
errors     → models
metrics    → (no deps)
keywords   → (no deps)
token      → (no deps)
tokenizer  → models, errors, metrics, keywords
ast        → models, metrics
parser     → models, errors, keywords, token, tokenizer, ast

# Higher-level / product packages
formatter  → models, sql/ast, sql/parser, sql/tokenizer
transform  → formatter, sql/ast, sql/keywords, sql/parser, sql/tokenizer
fingerprint→ formatter, sql/ast, sql/parser, sql/tokenizer
security   → sql/ast            (scanner; tests also pull parser, tokenizer)
linter     → sql/parser, sql/tokenizer
           # rule sub-packages additionally import: linter, models, sql/ast
lsp        → errors, models, gosqlx, sql/keywords, sql/parser, sql/tokenizer
cbinding   → gosqlx, sql/ast    (requires CGO; excluded from task test:race)

# High-level wrapper
gosqlx     → all of the above (top-level convenience API)
```

Notes:
- `pkg/cbinding` requires `CGO_ENABLED=1`. The Taskfile splits this out: `task test:race`
  runs everything except cbinding, and `task test:cbinding` runs cbinding with CGO on.
  CI workflows must follow the same split or cbinding is silently skipped.
- `keywords` has no intra-module deps — it's a pure keyword table.
- `ast` depends on `models` (spans, locations) and `metrics` (pool instrumentation),
  NOT on `token` in production code.

## Development Commands

This project uses [Task](https://taskfile.dev) as the task runner:
```bash
go install github.com/go-task/task/v3/cmd/task@latest
# Or: brew install go-task (macOS)
```

### Essential Commands
```bash
task                    # Show all available tasks
task build              # Build all packages
task build:cli          # Build CLI binary
task install            # Install CLI globally
task test               # Run all tests
task test:race          # Run tests with race detection (CRITICAL)
task test:pkg PKG=./pkg/sql/parser  # Test specific package
task bench              # Run benchmarks with memory tracking
task coverage           # Generate coverage report
task quality            # Run fmt, vet, lint
task check              # Full suite: format, vet, lint, test:race
task ci                 # Full CI pipeline
```

### Running a Single Test
```bash
go test -v -run TestSpecificName ./pkg/sql/parser/
go test -v -run "TestParser_Window.*" ./pkg/sql/parser/
go test -v -run "TestParser_TupleIn/Basic" ./pkg/sql/parser/  # Run specific subtest
```

### CLI Tool
```bash
./gosqlx validate "SELECT * FROM users"
./gosqlx format -i query.sql
./gosqlx analyze "SELECT COUNT(*) FROM orders GROUP BY status"
./gosqlx parse -f json query.sql

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajitpratap0/GoSQLX](https://github.com/ajitpratap0/GoSQLX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
