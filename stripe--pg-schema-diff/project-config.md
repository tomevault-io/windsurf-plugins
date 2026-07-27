---
trigger: always_on
description: A declarative schema migration tool for PostgreSQL that computes the difference between two database schemas and generates minimal, optimized SQL to migrate from one to the other with zero-downtime where possible.
---

# pg-schema-diff

A declarative schema migration tool for PostgreSQL that computes the difference between two database schemas and generates minimal, optimized SQL to migrate from one to the other with zero-downtime where possible.

## Project Overview

**Problem Solved**: Developers declare their desired database schema in DDL files, and pg-schema-diff automatically generates safe, optimized migration SQL that minimizes downtime and locks.

**Key Features**:
- Computes diffs between schemas (DDL files, databases, or directories)
- Generates SQL using native Postgres online operations (concurrent index builds, online constraint validation)
- Provides hazard warnings for dangerous operations
- Validates migration plans against temporary databases before execution

## Directory Structure

```
cmd/pg-schema-diff/     # CLI entry point (Cobra-based)
├── plan_cmd.go         # 'plan' subcommand - generates migration SQL
├── apply_cmd.go        # 'apply' subcommand - applies migrations
├── flags.go            # Flag parsing and DB connection handling

pkg/                    # Public API packages
├── diff/               # Core diffing and plan generation (main library interface)
├── tempdb/             # Temporary database factory for plan validation
├── log/                # Logging interface
├── schema/             # Public schema API wrapper
├── sqldb/              # Database queryable interface

internal/               # Internal implementation
├── schema/             # Complete schema representation types (schema.go is 46KB)
├── queries/            # SQL queries via sqlc for schema introspection
├── migration_acceptance_tests/  # Comprehensive test suite (24 test files)
├── pgengine/           # Postgres engine management for tests
├── pgdump/             # pg_dump integration
├── graph/              # Dependency graph for statement ordering
```

## Key Packages

### pkg/diff/ - Core Diffing Engine
- `plan_generator.go`: Orchestrates plan generation and validation
- `sql_generator.go`: Generates SQL statements for all object types (2,700+ lines)
- `sql_graph.go`: Dependency graph for correct statement ordering
- `schema_source.go`: Schema sources (DDL files, database, directories)

### internal/schema/ - Schema Representation
Core types in `schema.go`:
- `Schema`: Top-level container for all database objects
- `Table`: Tables with columns, constraints, policies, triggers
- `Index`: Index definitions including partial indexes and expressions
- `Column`, `ForeignKeyConstraint`, `CheckConstraint`, `View`, `Function`, etc.

### internal/queries/ - Database Queries
Uses **sqlc** for type-safe SQL queries. To modify:
1. Edit `queries.sql`
2. Run `make sqlc` to regenerate `queries.sql.go`

## Development Commands

```bash
# Run all tests (requires Docker or local Postgres)
go test -v -race ./... -timeout 30m

# Run specific acceptance tests
go test -v ./internal/migration_acceptance_tests/... -run TestIndexAcceptance

# Lint
make lint

# Fix lint issues
make lint_fix

# Regenerate sqlc code
make sqlc

# Tidy dependencies
make go_mod_tidy
```

## Testing

### Acceptance Tests
Located in `internal/migration_acceptance_tests/`. Each test file covers specific features:
- `index_cases_test.go`: Index operations
- `table_cases_test.go`: Table operations
- `column_cases_test.go`: Column operations
- `check_constraint_cases_test.go`, `foreign_key_constraint_cases_test.go`: Constraints
- `view_cases_test.go`, `function_cases_test.go`, `trigger_cases_test.go`, etc.

Test case structure:
```go
acceptanceTestCase{
    name:                 "test name",
    oldSchemaDDL:        []string{"CREATE TABLE ..."},
    newSchemaDDL:        []string{"CREATE TABLE ... (modified)"},
    expectedHazardTypes: []diff.MigrationHazardType{...},
    expectedPlanDDL:     []string{"ALTER TABLE ..."},  // optional: assert exact DDL
    expectEmptyPlan:     false,                         // optional: assert no changes
    planOpts:            []diff.PlanOpt{...},           // optional: custom plan options
}
```

### Running Tests with Docker
```bash
docker build -f build/Dockerfile.test --build-arg PG_MAJOR=15 -t pg-schema-diff-test .
docker run pg-schema-diff-test
```

## Key Concepts

### Migration Hazards
Operations are flagged with hazard types:
- `MigrationHazardTypeAcquiresAccessExclusiveLock`: Full table lock
- `MigrationHazardTypeDeletesData`: Potential data loss
- `MigrationHazardTypeIndexBuild`: Performance impact during build
- `MigrationHazardTypeIndexDropped`: Query performance may degrade
- `MigrationHazardTypeCorrectness`: Potential correctness issues

### Plan and Statements
```go
type Plan struct {
    Statements        []Statement
    CurrentSchemaHash string  // For validation before applying
}

type Statement struct {
    DDL         string           // SQL to execute
    Timeout     time.Duration    // statement_timeout
    LockTimeout time.Duration    // lock_timeout
    Hazards     []MigrationHazard
}
```

### Online Migration Techniques
- **Concurrent Index Building**: `CREATE INDEX CONCURRENTLY`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stripe/pg-schema-diff](https://github.com/stripe/pg-schema-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
