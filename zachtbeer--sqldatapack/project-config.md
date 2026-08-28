---
trigger: always_on
description: For agentic coding tools working in this repository. Prefer repo-grounded changes over generic .NET advice. Nothing is published yet; the first release is `1.0.0-preview.1`. The public API and SQLite package format are frozen for 1.0, so treat a change to either as a breaking decision that needs maintainer sign-off. Contributor-level detail on setup, dependencies, and style lives in [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Repository Guidelines

For agentic coding tools working in this repository. Prefer repo-grounded changes over generic .NET advice. Nothing is published yet; the first release is `1.0.0-preview.1`. The public API and SQLite package format are frozen for 1.0, so treat a change to either as a breaking decision that needs maintainer sign-off. Contributor-level detail on setup, dependencies, and style lives in [CONTRIBUTING.md](CONTRIBUTING.md).

## Where things live

Export runs `SqlDataPackExporter.cs` → `Internal/SqlServerSchemaReader.cs` (metadata discovery, export plan, table/column filtering, WHERE handling) → `Internal/BatchPlanner.cs` → `Internal/SqlitePackageWriter.cs` (the row-streaming loop, split out of the exporter so benchmarks can drive it through a `DbDataReader` seam) → `Internal/SqlitePackage.cs`.

Import runs `SqlDataPackImporter.cs` → `SqlitePackage.cs` → `SqlServerSchemaReader.ValidateImportTargetAsync` → `Internal/SqliteCoercingDataReader.cs` → `Internal/ImportPlanner.cs` (dependency ordering).

- `Options.cs` — public options, defaults, enums. Default changes are product decisions.
- `SqlDataPackOperationalModels.cs` — public result, manifest, progress, and exception types.
- `SqlDataPackReader.cs` — the supported read-only view of a package. Don't point consumers at the internal `zsdp_*` tables.
- `Internal/SqlitePackage.cs` — package schema, manifest, validation, import order, warnings, dacpac payload. Metadata tables are named `zsdp_*`; that prefix and `sqlite_` are both reserved against generated data table names by `SqlDataPackIdentifier.ValidateSqliteDataTableNamesNotReserved`.

### Package format version

`SqlDataPackVersion.PackageFormatVersion` and `SqlDataPackVersion.MinimumSupportedPackageFormatVersion` (bottom of `Internal/SqlitePackage.cs`) must **never** be changed without explicit confirmation from a human contributor. Ask; do not infer it from the change you are making. Bumping the write version silently makes every package unreadable by released builds, and raising the minimum drops the ability to read older ones — neither is recoverable from the package itself, and neither is a judgement call an agent should make on its own. Note that the versions do not gate metadata *table names*: a renamed metadata table fails the required-table check before any version check runs, so it cannot produce a useful error.
- `Internal/DacpacSchemaManager.cs` — DacFx extract/deploy and schema-scope safety. When changing deployment, check that a selected-table schema package can't enable object drops against unrelated target objects. Covered by `tests/SqlDataPack.IntegrationTests/Tests/DacpacScopeAndDeployTests.cs`.
- `Internal/ValueConverter.cs` + `ColumnKind.cs` — type conversion. `ColumnKind` is the per-cell dispatch key: adding a SQL Server type means adding it to `KindsByTypeName` *and* to every switch over the enum.
- `Internal/SqlDataPackIdentifier.cs` — SQL identifier quoting.

## Public API changes

Update `Options.cs`, `SqlDataPackOperationalModels.cs`, XML docs, `tests/SqlDataPack.Tests/PublicApiContractTests.cs`, README examples, and `website/docs/options.md`. The package ID and namespace are spelled `SqlDataPack`.

Public records in `SqlDataPack.Models` are deliberately not positional: explicit constructor, `{ get; init; }` properties, no generated `Deconstruct`. A `Deconstruct` freezes a member list that drifts the moment a manifest gains a field, and growing the positional list is a constructor break. Add new members as `init` properties. `PublicApiContractTests.PublicModelRecords_DoNotExposeDeconstruct` enforces it.

## Changelog

`CHANGELOG.md` is a pointer, not a log. Don't add entries to it. Per-version notes are generated from pull requests at release time, so the pull request title and body are what readers eventually see: write those for a reader, not for you.

The narrative changelog is `website/docs/changelog.md` and can be edited freely as part of ordinary work. It is not tied to a version and gates nothing, so a change there ships when the docs site next deploys. `docs/RELEASE.md` covers how a release happens.

## Commands

```bash
# fast unit + API shape
dotnet test tests/SqlDataPack.Tests/SqlDataPack.Tests.csproj

# FsCheck properties over the untrusted-input surface, no Docker
dotnet test tests/SqlDataPack.Fuzzing/SqlDataPack.Fuzzing.csproj

# needs Docker; CI runs the wider SQL Server version matrix
SQLDATAPACK_SQLSERVER_IMAGE=mcr.microsoft.com/mssql/server:2025-latest \
  dotnet test tests/SqlDataPack.IntegrationTests/SqlDataPack.IntegrationTests.csproj

dotnet test SqlDataPack.slnx                                    # everything

# reformat with jb cleanupcode; CI runs the same script and then `git diff --exit-code`
dotnet tool restore && build/cleanup.sh          # whole solution, ~30s
build/cleanup.sh path/to/Changed.cs              # just these files, ~20s


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zachtbeer/sqldatapack](https://github.com/zachtbeer/sqldatapack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
