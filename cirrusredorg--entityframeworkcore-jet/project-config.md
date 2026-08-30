---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What This Is

EntityFrameworkCore.Jet is an EF Core provider for Microsoft Jet/ACE databases (Microsoft Access `.mdb`/`.accdb` files). It runs **Windows only** and bridges EF Core to the Access database engine via either ODBC or OLE DB.

Current version: `10.0.x` targeting EF Core 10 and `net10.0`.

## Build

```powershell
dotnet build EFCore.Jet.sln
```

Assemblies are **strong-name signed** using `Key.snk`. `TreatWarningsAsErrors=True` is set globally — fix all warnings.

### Local EFCore Repository (optional)

To develop against a local EF Core build instead of NuGet packages, copy `Development.props.sample` to `Development.props` and set `LocalEFCoreRepository` to your EF Core checkout. That local build must be compiled with `AssemblyVersion=10.0.0.0` to avoid binding conflicts.

## Tests

Tests require a real Microsoft Access driver installed (ODBC or OLE DB) and an actual `.accdb` file — no mocks. The connection string is configured via:
- `test/EFCore.Jet.FunctionalTests/config.json` (OLE DB example present)
- `test/EFCore.Jet.Tests/config.json` (bare filename; picks up default provider)
- Or env var `EFCoreJet_DefaultConnection`

**Run all tests** (requires x86 or x64 matching your driver bitness):

```powershell
dotnet test EFCore.Jet.sln --configuration Debug
```

**Run a single test class:**

```powershell
dotnet test test\EFCore.Jet.FunctionalTests\EFCore.Jet.FunctionalTests.csproj --filter "FullyQualifiedName~NorthwindQueryJetTest"
```

**Run a single test method:**

```powershell
dotnet test test\EFCore.Jet.FunctionalTests\EFCore.Jet.FunctionalTests.csproj --filter "FullyQualifiedName=EntityFrameworkCore.Jet.FunctionalTests.Query.NorthwindQueryJetTest.Where_simple"
```

**When running a suite, capture the failing test *names* in the same run** — don't
reduce the output to just the `Passed!/Failed!` count line and then re-run the whole
suite to find which failed. Grep a pattern that catches both, e.g.
`grep -iE "Passed!|Failed!|\[FAIL\]|error CS"` (xUnit prints `… [FAIL]` and
`Failed <FullyQualifiedName>` lines as it goes), or tee the full output to a file and
inspect it. Only re-run after changing something.

Tests run in **fixed order by default** (`FIXED_TEST_ORDER` compile constant). All tests lock culture to `en-US` via a module initializer.

Tests that require features Jet doesn't support are skipped with a reason on the test.

## Project Structure

```
src/
  EFCore.Jet.Data/      ADO.NET driver — JetConnection, JetCommand, JetDataReader,
                        schema management, DUAL table simulation, connection pooling
  EFCore.Jet/           EF Core provider — query pipeline, migrations, scaffolding,
                        type mappings, value generation, conventions
  EFCore.Jet.Odbc/      Provider factory for ODBC data access
  EFCore.Jet.OleDb/     Provider factory for OLE DB data access
  Shared/               Shared source files compiled into multiple src projects
  LibRed/               Native, fully-managed Jet/ACE engine (cross-platform) —
                        see "LibRed" section below and src/LibRed/README.md

test/
  EFCore.Jet.Data.Tests/          Unit tests for the ADO.NET driver layer
  EFCore.Jet.FunctionalTests/     EF Core specification tests (adapted from EF Core's own suite)
  EFCore.Jet.Tests/               Additional functional tests
  EFCore.Jet.IntegrationTests/    Integration scenario tests
  JetProviderExceptionTests/      Exception-path tests
  Shared/               Shared test infrastructure (xunit framework customizations,
                        test orderers, conditional test attributes)
```

## Architecture: Two-Layer Design

**Layer 1 — `EFCore.Jet.Data`** wraps the raw ODBC/OLE DB driver:
- `JetConnection` detects whether the connection string is ODBC or OLE DB and delegates to the appropriate inner `DbConnection`.
- `JetCommand` rewrites SQL at runtime: handles `SELECT SKIP`, emulates `@@ROWCOUNT`, rewrites `TOP @param`, parses `IF NOT EXISTS ... THEN ...` syntax, and intercepts stored-procedure creation.
- `JetConfiguration` holds global settings: `TimeSpanOffset` (Jet has no TimeSpan; dates are offset from 1899-12-30), `CustomDualTableName`, `IntegerNullValue`, `UseConnectionPooling`.
- Schema operations (create/drop database, list tables) have three implementations: ADOX, DAO, and Precise, selected based on available COM libraries.

**Layer 2 — `EFCore.Jet`** is the EF Core provider:
- `JetServiceCollectionExtensions.AddEntityFrameworkJet()` registers all provider services.
- `JetQuerySqlGenerator` extends `QuerySqlGenerator` to produce Jet-compatible SQL — converts `CAST` to Jet VBA functions (`CBOOL`, `CINT`, `CLNG`, etc.), handles boolean/numeric null semantics.
- `JetQueryTranslationPostprocessor` applies Jet-specific query rewrites in order: skip/take transformation → base postprocessing → optional millisecond support → ORDER BY lifting. `JetSkipTakePostprocessor` emulates `SKIP`/`OFFSET` since Jet only supports `SELECT TOP n`.
- `JetMigrationsSqlGenerator` generates DDL for Access (no `ALTER COLUMN`, limited constraint support).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CirrusRedOrg/EntityFrameworkCore.Jet](https://github.com/CirrusRedOrg/EntityFrameworkCore.Jet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
