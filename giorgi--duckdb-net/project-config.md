---
trigger: always_on
description: This file provides guidance to coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working with code in this repository.

DuckDB.NET is an ADO.NET provider and low-level bindings for DuckDB. The solution (`DuckDB.NET.slnx`) has five
projects: `Bindings` (P/Invoke over the DuckDB C API), `Data` (the ADO.NET provider), `Test` (xUnit),
`Samples` and `Benchmarks` (BenchmarkDotNet). The libraries target `net8.0;net10.0`; the test project targets
**net8.0 only**, so `dotnet test -f net10.0` silently runs nothing. The SDK is pinned by `global.json` (10.0.2xx).
User documentation lives in a separate docfx repository (`S:\src\DuckDB.NET-Docs`, published at https://duckdb.net).

## Commands

```bash
# Build everything the way CI does (Full = download native libraries for all platforms)
dotnet build --configuration Release /p:BuildType=Full

# Run all tests (CI also adds /p:CollectCoverage=true /p:CoverletOutputFormat=lcov)
dotnet test DuckDB.NET.Test/Test.csproj --configuration Release /p:BuildType=Full

# One class, one test, or everything except the slow VARINT parameter tests
dotnet test DuckDB.NET.Test/Test.csproj --filter "FullyQualifiedName~DuckDB.NET.Test.TableFunctionTests"
dotnet test DuckDB.NET.Test/Test.csproj --filter "FullyQualifiedName~DuckDB.NET.Test.TableFunctionTests.TestTableFunction"
dotnet test DuckDB.NET.Test/Test.csproj --filter "FullyQualifiedName!~VarintTest"

# Packages: the plain packages need a separately installed native library; /p:BuildType=Full produces the .Full variants
dotnet pack DuckDB.NET.Data/Data.csproj --configuration Release /p:BuildType=Full

# Benchmarks
dotnet run -c Release --project DuckDB.NET.Benchmarks
```

### Native libraries

- The `DownloadNativeLibs` target in `Bindings.csproj` (with `DownloadNativeLibs.targets`) downloads the DuckDB
  release zips into `DuckDB.NET.Bindings/obj/runtimes/<rid>/native`. The test project always triggers this download,
  whatever `BuildType` is.
- **The cache is version-blind:** a platform is downloaded only if its folder is missing. After changing the DuckDB
  version, delete `DuckDB.NET.Bindings/obj/runtimes`, rebuild, and confirm the version with `SELECT version()`.
- **Nightly builds:** `/p:NightlyBuild=true /p:DuckDbArtifactRoot=https://artifacts.duckdb.org/<branch>` downloads
  DuckDB's `duckdb-shared-libs-<platform>.tar.gz` instead. `<branch>` is a DuckDB branch name, and `latest` means
  DuckDB's `main`. CI does this on the schedule: it uses the workflow file from `develop`, builds the code of the
  `nightly-builds` branch, and currently tracks `v2.0-cyanoptera` until DuckDB 2.0 ships.

## DuckDB version and releases

- **Current target: DuckDB v1.5.6.** The version is set in `DuckDB.NET.Bindings/Bindings.csproj`
  (`DuckDbArtifactRoot`). The "Updated to DuckDB vX" line in the `PackageReleaseNotes` of both `Bindings.csproj` and
  `Data.csproj` also needs updating. Every `LibraryImport` names its `EntryPoint`, so after an update, compare those
  names with the library's exports to catch removed functions.
- **Package version:** comes from git tags through GitVersion (`GitVersion.yml`). DuckDB.NET versions follow the
  DuckDB version (tag `1.5.6`).
- **Release steps:**
  1. Update the release notes in both project files.
  2. Merge `develop` into `main` and tag.
  3. Wait for the "Build" run on `main` to succeed.
  4. Start the `NuGet` workflow (`.github/workflows/NuGet.yml`, manual) on `main`. It publishes the
     `nugetPackages-main` artifact of the latest *successful* main build, so starting it earlier publishes an old
     build. Pushes to `develop` publish to GitHub Packages only.

## Architecture

### Bindings (`DuckDB.NET.Bindings`)

- **`NativeMethods`:** a partial class split into one file per C API area under `NativeMethods/` (Appender, Arrow,
  DataChunks, PreparedStatements, Query, ScalarFunction, StreamingResult, TableFunction, Value, Vectors, ...). Each
  declaration is a `LibraryImport` with an explicit `EntryPoint` and the `DuckDbLibrary` constant; some hot paths
  use `[SuppressGCTransition]`.
- **Lifetimes:** native objects are `SafeHandle`s in `DuckDBWrapperObjects.cs` (database, connection, prepared
  statement, data chunk, logical type, value, ...). The exception is **`DuckDBResult`**, a plain struct in
  `DuckDBNativeObjects.cs` with no finalizer. Every result must be `Close()`d exactly once, or it leaks permanently.
- **Strings:** returned strings use the marshallers in `DuckDBStringMarshallers.cs`. Use
  `DuckDBOwnedStringMarshaller` when DuckDB owns the memory, and `DuckDBCallerOwnedStringMarshaller` when the caller
  must `duckdb_free` it.

### Connections (`DuckDB.NET.Data/Connection`)

- **Pooling:** `DuckDBConnection` gets its native connection from the static `ConnectionManager`. It caches one
  `FileReference` (native database plus a reference count) per file. Returning the last connection disposes the
  database and removes the cache entry.
- **Data sources:**
  - `:memory:` gives every connection its own private database;
  - `:memory:?cache=shared` shares one;
  - `md:<db>?motherduck_token=...` connects to MotherDuck.
- **Configuration:** `DuckDBConnectionStringBuilder` normalizes `Data Source`/`DataSource` and exposes DuckDB
  configuration options as typed properties.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Giorgi/DuckDB.NET](https://github.com/Giorgi/DuckDB.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
