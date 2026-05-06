---
trigger: always_on
description: SQG (SQL Query Generator) is a **type-safe SQL code generator** that reads SQL queries from `.sql` files with special annotations and generates type-safe database access code in multiple target languages (TypeScript, Java, and Python). It introspects SQL queries at build time against real database engines to determine column types and generates strongly-typed wrapper functions.
---

# AGENTS.md - SQG Project Guide for AI Agents

## Project Overview

SQG (SQL Query Generator) is a **type-safe SQL code generator** that reads SQL queries from `.sql` files with special annotations and generates type-safe database access code in multiple target languages (TypeScript, Java, and Python). It introspects SQL queries at build time against real database engines to determine column types and generates strongly-typed wrapper functions.

**Repository:** https://github.com/sqg-dev/sqg
**Website:** https://sqg.dev

**Key capabilities:**
- Parse SQL files with metadata annotations
- Execute queries against SQLite, DuckDB, or PostgreSQL to introspect types
- Generate type-safe TypeScript, Java, or Python code
- Support complex types: structs, lists, maps (especially for DuckDB)

## Project Structure

This is a **pnpm monorepo workspace**:

```
sqg/
├── sqg/                          # Core code generator package (@sqg/sqg)
│   ├── src/
│   │   ├── sqg.ts               # CLI entry point
│   │   ├── sqltool.ts           # Main orchestration, validation
│   │   ├── sql-query.ts         # SQL parsing logic
│   │   ├── constants.ts         # Centralized generator definitions and parsing
│   │   ├── errors.ts            # Structured error classes
│   │   ├── init.ts              # Project scaffolding (sqg init)
│   │   ├── type-mapping.ts      # Type system mapping
│   │   ├── db/
│   │   │   ├── sqlite.ts        # SQLite adapter
│   │   │   ├── postgres.ts      # PostgreSQL adapter
│   │   │   └── duckdb.ts        # DuckDB adapter
│   │   ├── generators/          # Language-specific generators
│   │   │   ├── typescript-generator.ts
│   │   │   ├── java-generator.ts
│   │   │   └── python-generator.ts
│   │   ├── parser/
│   │   │   ├── sql.grammar      # Lezer grammar definition
│   │   │   └── sql-parser.ts    # Generated parser (do not edit)
│   │   └── templates/           # Handlebars code templates
│   │       ├── typescript-duckdb.hbs
│   │       ├── better-sqlite3.hbs
│   │       ├── java-jdbc.hbs
│   │       ├── java-duckdb-arrow.hbs
│   │       └── python.hbs
│   ├── tests/                   # Test files and fixtures
│   │   ├── sqltool.test.ts      # Integration tests
│   │   ├── generator.test.ts    # Unit tests
│   │   ├── test-duckdb.yaml     # Test project configs
│   │   ├── test-duckdb.sql      # Test SQL files
│   │   └── __snapshots__/       # Snapshot test files
│   ├── java/                    # Java test project (Gradle)
│   ├── python/                  # Python test project (uv + pytest)
│   └── justfile                 # Task runner recipes
├── website/                     # Astro + Starlight documentation site
├── examples/
│   ├── typescript-sqlite/       # Example: SQLite + TypeScript
│   └── typescript-duckdb/       # Example: DuckDB + TypeScript
└── pnpm-workspace.yaml
```

## Key Files Reference

| File | Purpose |
|------|---------|
| `sqg/src/sqg.ts` | CLI entry point with commands (init, syntax) and options (--validate, --format) |
| `sqg/src/sqltool.ts` | Main orchestrator: parses YAML config, validates, coordinates generation |
| `sqg/src/sql-query.ts` | Custom SQL parser using Lezer, extracts query metadata |
| `sqg/src/constants.ts` | Centralized generator definitions, parsing functions (parseGenerator, resolveGenerator) |
| `sqg/src/errors.ts` | Structured error classes with codes, suggestions, context |
| `sqg/src/init.ts` | Project scaffolding logic for `sqg init` command |
| `sqg/src/type-mapping.ts` | Maps SQL types to target language types |
| `sqg/src/parser/sql.grammar` | Lezer grammar for annotated SQL syntax |
| `sqg/src/templates/*.hbs` | Handlebars templates for generated code |

## Tech Stack

- **Runtime:** Node.js >= 20
- **Package Manager:** pnpm
- **Build:** tsdown, tsc
- **Testing:** vitest
- **Linting:** biome
- **Parsing:** @lezer/lr (LR parser generator)
- **Templates:** handlebars
- **DB Drivers:** better-sqlite3, pg, @duckdb/node-api
- **Validation:** zod

## External Documentation

- **DuckDB Node.js API:** https://duckdb.org/docs/stable/clients/node_neo/overview
- **DuckDB Java API:** https://duckdb.org/docs/stable/clients/java

## Build & Test Commands

**From the `sqg/` subdirectory:**

```bash
# Install dependencies
pnpm install

# Build the project
pnpm build

# Run tests
pnpm test        # Watch mode
pnpm test:run    # Single run
pnpm test:run -u # Update snapshots

# Regenerate Lezer parser (after editing sql.grammar)
pnpm lezer-gen

# Lint and format
pnpm check

# Run SQG directly
pnpm sqg <path>

# Validate configuration without generating
pnpm sqg --validate <path>

# Get JSON output (for programmatic use)
pnpm sqg --format json <path>

# Initialize a new project
pnpm sqg init --generator typescript/duckdb
```

**Using justfile (from `sqg/sqg/`):**

```bash
just                # List all available recipes
just generate       # Generate all code from test SQL files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sqg-dev/sqg](https://github.com/sqg-dev/sqg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
