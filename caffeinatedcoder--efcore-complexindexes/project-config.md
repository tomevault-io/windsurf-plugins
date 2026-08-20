---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build
dotnet build EFCore.ComplexIndexes.slnx

# Test
dotnet test test/EFCore.ComplexIndexes.Tests/EFCore.ComplexIndexes.Tests.csproj

# Run a single test class
dotnet test --filter "ClassName=MigrationModelDifferTests"

# Run a single test method
dotnet test --filter "FullyQualifiedName~MigrationModelDifferTests.SingleIndex_IsCreated"

# Pack NuGet packages
dotnet pack src/EFCore.ComplexIndexes/EFCore.ComplexIndexes.csproj
```

Tests run in parallel at the method level (`Scope = ExecutionScope.MethodLevel`). The
`PostgresIntegrationTests` class (`[TestCategory("Integration")]`, `[DoNotParallelize]`) spins up a
PostgreSQL 18 Testcontainer and applies generated DDL for real; without Docker it needs excluding
via `--filter "TestCategory!=Integration"`.

Locally, no Docker means those tests go **inconclusive**. When the `CI` environment variable is set
they **fail** instead — an unreachable container must not quietly retire the end-to-end layer while
the build reports green.

## CI

`.github/workflows/dotnet.yml` runs on pushes and PRs to `main`: the unit suite, the integration
suite (Docker), then a pack job whose value is partly that packing *is* a check — a package
declaring `PackageReadmeFile` without packing the file fails with NU5019. Everything runs on
ubuntu-latest. Actions in both workflows are pinned to commit SHAs with the tag in a trailing
comment; `.github/dependabot.yml` keeps those pins current (weekly, grouped, minors and patches
only — majors on the publish path are deliberate, see the artifact-action comments in
`release.yml`) and covers the test project's NuGet references monthly — not `src/`, whose EF Core
and provider floors are release decisions. A `src/` entry with no version updates exists only so
security updates there honour its ignore list: Dependabot's fix for the build-only NU1903 would be
a *direct* `System.Security.Cryptography.Xml` reference, shipping to consumers a package they never
had.

The unit job used to fan out across ubuntu/windows/macos for the repository-convention tests, which
do real path work. Dropped in favour of reacting if it ever bites: **no shipped code touches the
filesystem** (the `Path`-looking code under `src/` splits dotted *property* paths), so the other
runners only ever proved that the test harness composes paths portably. That is a contributor-
environment property, not a package one, and it fails loudly on the machine of whoever hits it.
Note what the matrix was *not* buying, in case it looks tempting to restore: whether the packaged
`.targets` works under Windows MSBuild is still untested either way, since the smoke test is
Linux-only and `PackagingConventionTests` reads the `.targets` rather than executing it.

The job deliberately keeps a single-entry `matrix.os` rather than a plain `runs-on`, so it still
reports as `Test (ubuntu-latest)` — a required check in branch protection, where a renamed job stays
required, never reports, and hangs PRs instead of failing them.

Both workflows generate a **CycloneDX SBOM per package** (`*.cdx.json`) alongside the nupkgs, via
the `cyclonedx` tool pinned in `.config/dotnet-tools.json`. `--exclude-filter` drops
`Microsoft.EntityFrameworkCore.Design` and its subtree: that reference is `PrivateAssets=all`, so
without the filter the core package's SBOM would list ~45 MSBuild/Roslyn components against a nuspec
declaring one, handing consumers vulnerability alerts for code they never receive.
`PackagingConventionTests` asserts the filter still covers every `PrivateAssets=all` reference — an
SBOM that misdescribes the package is worse than none. Publishing one is a courtesy: the CRA's SBOM
duty falls on commercial consumers for their own products, not on this package.

That filter test flattens the three projects into one set of ids, so it only ever asks whether a
name is *somewhere* private; a second guard asserts a reference marked `PrivateAssets=all` in one
project is marked so in **every** project. Dropping it from a single csproj is invisible to every
other check — the package still builds, the SBOM test still passes — while that one nuspec starts
declaring the dependency and its consumers restore the whole subtree behind it. That is what makes
"consumers never receive it" true, and it is the premise `NU1903` staying a warning rests on
(`Directory.Build.props`), along with the `System.Security.Cryptography.Xml` ignore under `src/` in
`.github/dependabot.yml`. Confirmed by removing the metadata from one satellite: a consumer of the
resulting package inherits all eight advisories.

`.github/workflows/release.yml` publishes on a `v*` tag. `Directory.Build.props` stays the single
source of truth: the tag is verified against it rather than driving it, packages are discovered from
the pack output (so a future satellite needs no workflow edit), and pushes use `--skip-duplicate` so
a re-run after a partial failure is safe. The release gate runs the full suite, which means
`ChangelogConsistencyTests` blocks a version nothing documents.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CaffeinatedCoder/EFCore.ComplexIndexes](https://github.com/CaffeinatedCoder/EFCore.ComplexIndexes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
