---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DbUnit is a JUnit extension for database-driven testing. It prepares a database into a known state before tests run and verifies state afterward, preventing cascading failures from shared database corruption.

## Build and Test Commands

Unit tests (`**/*Test.java`) run with Surefire; integration tests (`**/*IT.java`) run with Failsafe.
Tests run with `user.timezone=Europe/Berlin`.

Use the Maven wrapper (`./mvnw`) for all build operations.

```bash
# Build and install (default goal)
./mvnw install

# Generate the Maven site (japi-compliance-checker needs the jar, so install first)
./mvnw clean install site

# Run unit tests only
./mvnw clean test

# Run all tests including integration tests
./mvnw clean verify

# Run a single test class
./mvnw clean test -Dtest=ClassName

# Run a single test method
./mvnw clean test -Dtest=ClassName#methodName

# Run integration tests against a specific database
./mvnw clean verify -Pderby-10-14
./mvnw clean verify -Ph2-1-4
./mvnw clean verify -Phsqldb-2-7
./mvnw clean verify -Ppostgresql-16
./mvnw clean verify -Pmysql-9-20
./mvnw clean verify -Pmssql-2022
./mvnw clean verify -Poracle-18
./mvnw clean verify -Poracle-23

# Run integration tests against all databases
build-with-all-DBs.ps1 or build-with-all-DBs.sh
## It requires Docker daemon already running.
## It runs the core build and then the ITs with each database, sequentially, as listed in the database-profiles.txt file located in the root.
## The script writes all build output to log files, one per database Maven profile, in the build-logs directory in the root.
```

## Architecture

### Core Abstractions

**`IDatabaseConnection`** — wraps a JDBC connection; creates datasets and tables from live DB. Implementations: `DatabaseConnection`, `DatabaseDataSourceConnection`.

**`IDataSet`** — a collection of `ITable` instances representing database tables. Many implementations for different sources and transformations:
- Format-based: `FlatXmlDataSet`, `XmlDataSet`, `CsvDataSet`, `YamlDataSet`, Excel (`XlsDataSet`)
- Decorator-based: `CachedDataSet`, `FilteredDataSet`, `CompositeDataSet`, `SortedDataSet`, `ReplacementDataSet`
- Live: `DatabaseDataSet` (from `IDatabaseConnection`), `QueryDataSet`

**`DatabaseOperation`** — abstract base for database mutations. Static constants: `INSERT`, `UPDATE`, `DELETE`, `DELETE_ALL`, `TRUNCATE_TABLE`, `REFRESH` (upsert), `CLEAN_INSERT` (truncate + insert). Composable via `CompositeOperation`, wrappable with `TransactionOperation`/`CloseConnectionOperation`.

**`IDatabaseTester`** — the preferred composition-based test helper (use this over extending `DBTestCase`). Manages setup/teardown lifecycle. Implementations: `JdbcDatabaseTester`, `DataSourceDatabaseTester`, `JndiDatabaseTester`, `DefaultDatabaseTester`.

**`IOperationListener`** — hooks for `connectionRetrieved()`, `operationSetUpFinished()`, and `operationTearDownFinished()` events during test lifecycle.

### Assertion API

Located in `org.dbunit.assertion`. `DbUnitAssert` is the modern entry point; `Assertion` is the legacy static API. Key types:
- `Difference` — a mismatch between expected and actual tables
- `DifferenceListener` / `FailureHandler` — customize how mismatches are reported
- `ValueComparer` — compare individual cell values (16+ implementations in `assertion/comparer/value/`)

`ValueComparer` implementations support: equality, greater-than/less-than, tolerance (numeric and timestamp), null checks, string containment, and ignore-millis for timestamps.

### Dataset Filtering and Ordering

`org.dbunit.dataset.filter` — exclude/include tables from a dataset.
`org.dbunit.database.DatabaseSequenceFilter` — orders tables respecting FK constraints.
`org.dbunit.database.PrimaryKeyFilter` — filters rows by primary key values.

### Data Types

`org.dbunit.dataset.datatype` — maps SQL types to Java types. `DefaultDataTypeFactory` handles standard types. Database-specific factories (e.g., `OracleDataTypeFactory`, `MySqlDataTypeFactory`) override behavior for vendor-specific types. Register via `DatabaseConfig.PROPERTY_DATATYPE_FACTORY`.

### Database Extensions

`org.dbunit.ext.*` — database-specific implementations for DB2, H2, HSQLDB, McKoi, MSSQL, MySQL, Netezza, Oracle, PostgreSQL. Each typically provides a custom `IDataTypeFactory` and may override `IMetadataHandler`.

### DatabaseConfig

`org.dbunit.database.DatabaseConfig` — central configuration for a connection. Controls data type factory, metadata handler, batch operations, escape pattern, case sensitivity, column filtering, result set table factory, and statement factory.

### Package Structure

```
org.dbunit
├── database/       IDatabaseConnection, DatabaseConfig, QueryDataSet
├── dataset/        IDataSet, ITable, Column, data formats, datatype/, filter/
├── operation/      DatabaseOperation implementations
├── assertion/      DbUnitAssert, ValueComparer implementations
├── ext/            Database-specific overrides (db2/, h2/, hsqldb/, mssql/, mysql/, oracle/, postgresql/)
├── ant/            Apache Ant task support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbunit/dbunit-extension](https://github.com/dbunit/dbunit-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
