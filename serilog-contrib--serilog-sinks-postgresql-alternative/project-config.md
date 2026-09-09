---
trigger: always_on
description: Serilog.Sinks.Postgresql.Alternative is a Serilog sink that writes log events into a PostgreSQL
---

# Project rules for Claude

## What this is

Serilog.Sinks.Postgresql.Alternative is a Serilog sink that writes log events into a PostgreSQL
table. It is published as the NuGet package
[Serilog.Sinks.Postgresql.Alternative](https://www.nuget.org/packages/Serilog.Sinks.Postgresql.Alternative/),
so `GeneratePackageOnBuild` is on and every build drops a `.nupkg` and a `.snupkg` into
`src/Serilog.Sinks.Postgresql.Alternative/bin/Release`. The project is a maintained fork of
[b00ted/serilog-sinks-postgresql](https://github.com/b00ted/serilog-sinks-postgresql).

One solution `src/Serilog.Sinks.Postgresql.Alternative.sln` with exactly three projects:

- `src/Serilog.Sinks.Postgresql.Alternative/Serilog.Sinks.Postgresql.Alternative.csproj`, the
  library and the only packed project.
- `src/Serilog.Sinks.Postgresql.Alternative.Tests/...csproj`, MSTest, unit tests that need no
  database.
- `src/Serilog.Sinks.Postgresql.Alternative.IntegrationTests/...csproj`, MSTest, tests that write
  into a real PostgreSQL instance.

Layout inside `src/Serilog.Sinks.Postgresql.Alternative`:

- `LoggerConfigurationPostgreSQLExtensions.cs`: the public entry point, four `PostgreSQL` extension
  methods (two for `LoggerSinkConfiguration`, two for `LoggerAuditSinkConfiguration`) plus the
  internal `GetOptions` and `ClearQuotationMarksFromColumnOptions`. Everything a user configures
  ends up in a `PostgreSqlOptions`.
- `Sinks/PostgreSQL/PostgreSQLSink.cs`: `IBatchedLogEventSink`, hands batches to the sink helper.
  `Sinks/PostgreSQL/PostgreSQLAuditSink.cs`: `ILogEventSink`, writes single events.
- `Sinks/PostgreSQL/SinkHelper.cs`: the actual work, shared by both sinks. Opens the connection,
  creates schema and table on demand, writes via `COPY` or `INSERT`, deletes old rows when a
  retention time is set. The query builders `GetCopyCommand`, `GetInsertQuery` and `GetDeleteQuery`
  each do one thing, keep new SQL in that shape.
- `Sinks/PostgreSQL/SchemaCreator.cs` and `TableCreator.cs`: the `CREATE SCHEMA` and `CREATE TABLE`
  statements. `SqlTypeHelper.cs`: `NpgsqlDbType` to SQL type string.
- `Sinks/PostgreSQL/ColumnWriters/`: `ColumnWriterBase` plus one writer per column kind.
  `DefaultColumnWriter` is not a writer at all, it is the DTO that JSON configuration binds to.
- `Sinks/PostgreSQL/ColumnOptions.cs`: the default column set. `DefaultColumnNames.cs`: its keys.
- `Sinks/PostgreSQL/Configuration/`: reads named connection strings out of an `IConfiguration`.
- `Sinks/PostgreSQL/EventArgs/`: the two callback argument types.
- `Sinks/PostgreSQL/Async/`: `AsyncEvent` and `AsyncEventInvocator`, currently unused, see below.
- `GlobalUsings.cs`: all usings of the project, including the alias `SystemEventArgs`.

Layout inside the test projects:

- `Serilog.Sinks.Postgresql.Alternative.Tests/ColumnWritersTests/`: one test class per column
  writer, 12 tests in total, no database and no network needed.
- `Serilog.Sinks.Postgresql.Alternative.IntegrationTests/`: `DbWriteTests`, `DbWriteWithSchemaTests`,
  the two `JsonConfigTest*` classes, the helper `DbHelper` and `BaseTests` with the connection
  string. The four `PostgreSinkConfiguration*.json` files are copied to the output directory with
  `CopyToOutputDirectory=Always`.

Repository root: `README.md` (badges, target frameworks, links), `HowToUse.md` (the actual user
documentation with all configuration options), `Changelog.md`, `Updating.md` (the five release
steps), `License.txt` (MIT), `Icon.png`, `BuildAndPushPackage.bat`, `Delete-BIN-OBJ-Folders.bat`,
`.all-contributorsrc`, `.gitattributes` and `.gitignore`. There is no `.github` folder and no
pipeline file.

## Build

```powershell
dotnet build src/Serilog.Sinks.Postgresql.Alternative.sln
```

```powershell
dotnet test src/Serilog.Sinks.Postgresql.Alternative.Tests/Serilog.Sinks.Postgresql.Alternative.Tests.csproj
```

- The library multi-targets `net8.0;net10.0`, the `## Available for` list in `README.md` has to
  match. Both test projects are single target `net10.0`.
- The test projects are on MSTest 4. `[ExpectedException]` and `Assert.ThrowsException` do not
  exist there any more, use `Assert.Throws<T>` or `Assert.ThrowsExactly<T>`.
- **Restore needs nuget.org.** A private feed is configured globally on this machine and answers
  404 or refuses the connection for public packages, so a plain `dotnet build` fails with `NU1301`
  and, because warnings are errors, additionally with `NU1900`. Always build with an explicit
  source:
  `dotnet build src/Serilog.Sinks.Postgresql.Alternative.sln -c Release --source https://api.nuget.org/v3/index.json`.
- `TreatWarningsAsErrors` is enabled in all three projects, so every warning breaks the build,
  NuGet warnings (`NU****`) from restore included. A clean build reports zero warnings, keep it that
  way.
- `NU1803` (HTTP source usage during restore) is the one warning suppressed via `NoWarn`. Fix
  warnings instead of extending that list. `NuGetAudit` and `NuGetAuditMode=all` are on, so a
  vulnerable transitive package fails the build too.
- `src/Directory.Build.props` exists but sets exactly one property,
  `GenerateDocumentationFile`. Every other build property is written out in each of the three

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serilog-contrib/Serilog.Sinks.Postgresql.Alternative](https://github.com/serilog-contrib/Serilog.Sinks.Postgresql.Alternative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
