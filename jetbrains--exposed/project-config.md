---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Exposed is a lightweight ORM framework for Kotlin that provides two APIs:
- **DSL API**: Type-safe SQL-wrapping Domain Specific Language (in `exposed-core`)
  - Works with both JDBC (`exposed-jdbc`) and R2DBC (`exposed-r2dbc`)
- **DAO API**: Lightweight Data Access Object API (in `exposed-dao`)
  - **Only works with JDBC** - does not support R2DBC

## Module Architecture

### Core Modules
- **exposed-core**: Foundation layer with DSL API, database abstractions, column types, and vendor dialects
- **exposed-dao**: DAO API with entity classes and relationships (JDBC only, does not work with R2DBC)
- **exposed-jdbc**: JDBC implementation with blocking transactions
- **exposed-r2dbc**: R2DBC implementation with suspending transactions

### Extension Modules
- **exposed-java-time**, **exposed-jodatime**, **exposed-kotlin-datetime**: Date/time support
- **exposed-json**: JSON/JSONB column types
- **exposed-crypt**: Encrypted column types
- **exposed-money**: JavaMoney MonetaryAmount support
- **exposed-migration-core**: Common migration functionality
- **exposed-migration-jdbc**: JDBC-based schema migrations
- **exposed-migration-r2dbc**: R2DBC-based schema migrations
- **exposed-spring-boot-starter**: Spring Boot integration
- **spring-transaction**: Spring Framework transaction manager

### Test Modules
- **exposed-tests**: Main JDBC-based test suite
- **exposed-r2dbc-tests**: R2DBC-specific test suite
- **exposed-jdbc-r2dbc-tests**: Cross-compatibility tests

## Build & Development

### Building
```bash
./gradlew compileKotlin  # Compile the projects code
./gradlew detekt         # Validate code style
./gradlew apiDump        # Update dokka API docs after changing public API
```

### Running Tests

Tests are organized by database and dialect. Each module has database-specific test tasks:

#### Quick test with H2 (no Docker required)
```bash
./gradlew test_h2_v2                              # All modules with H2
./gradlew :exposed-tests:test_h2_v2               # JDBC Tests with H2
./gradlew :exposed-r2dbc-tests:test_h2_v2         # R2DBC Tests with H2
```

#### Test with Postgres
```bash
./gradlew test_postgres                           # All modules with Postgres
./gradlew :exposed-tests:test_postgres            # JDBC Tests with Postgres
./gradlew :exposed-r2dbc-tests:test_postgres      # R2DBC Tests with Postgres
```

#### Test with specific database (requires Docker)
```bash
# Start database containers first
./gradlew mariadbComposeUp        # Start MariaDB
./gradlew postgresComposeUp       # Start PostgreSQL
./gradlew mysql8ComposeUp         # Start MySQL 8
./gradlew oracleComposeUp         # Start Oracle
./gradlew sqlserverComposeUp      # Start SQL Server

# Run tests
./gradlew :exposed-tests:test_postgres
./gradlew :exposed-tests:test_mysql_v8
./gradlew :exposed-tests:test_mariadb

# Stop containers
./gradlew postgresComposeDownForced
```

#### Run specific test class or method with H2
```bash
./gradlew :exposed-tests:test_h2_v2 --tests "org.jetbrains.exposed.v1.tests.shared.dml.InsertTests"
./gradlew :exposed-tests:test_h2_v2 --tests "*.InsertTests.testBatchInsert"
```

#### Available test databases
- `test_h2_v2`, `test_h2_v2_mysql`, `test_h2_v2_psql`, etc. (H2 with different dialect emulations)
- `test_sqlite`
- `test_mysql_v5`, `test_mysql_v8`
- `test_mariadb`
- `test_postgres`, `test_postgresng`
- `test_oracle`
- `test_sqlserver`

## Testing Infrastructure

### Test Base Classes and Utilities

Tests inherit from different base classes depending on the driver:

**JDBC Tests** - inherit from `DatabaseTestsBase` (in `exposed-tests/src/main/kotlin/org/jetbrains/exposed/v1/tests/`):
- Tests are parameterized by database dialect using `@ParameterizedClass` and `@MethodSource("data")`
- Each test automatically runs against all enabled dialects
- Available dialects are determined by system properties set by Gradle test tasks

**R2DBC Tests** - inherit from `R2dbcDatabaseTestsBase` (in `exposed-r2dbc-tests/src/main/kotlin/`):
- Similar parameterized testing pattern as JDBC
- Uses suspending functions and coroutine context
- Test methods use `= runTest { }` for coroutine support, or utils methods like `withDb`, `withTables`,

### TestDB Enums

There are separate `TestDB` enums for JDBC and R2DBC tests:

**JDBC TestDB** (`exposed-tests/src/main/kotlin/org/jetbrains/exposed/v1/tests/TestDB.kt`):
- Connection strings using JDBC URLs (e.g., `jdbc:h2:mem:...`, `jdbc:postgresql://...`)
- JDBC driver class names
- Before/after connection hooks
- Database-specific configuration (e.g., H2 dialect emulation modes)

Available JDBC TestDB values:
- `H2_V2`, `H2_V2_MYSQL`, `H2_V2_PSQL`, `H2_V2_MARIADB`, `H2_V2_ORACLE`, `H2_V2_SQLSERVER`
- `SQLITE`, `MYSQL_V5`, `MYSQL_V8`, `MARIADB`, `POSTGRESQL`, `POSTGRESQLNG`, `ORACLE`, `SQLSERVER`

**R2DBC TestDB** (`exposed-r2dbc-tests/src/main/kotlin/org/jetbrains/exposed/v1/r2dbc/tests/TestDB.kt`):
- Connection strings using R2DBC URLs (e.g., `r2dbc:h2:mem:...`, `r2dbc:postgresql://...`)
- R2DBC isolation levels
- Suspend-aware before/after connection hooks

Available R2DBC TestDB values:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JetBrains/Exposed](https://github.com/JetBrains/Exposed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
