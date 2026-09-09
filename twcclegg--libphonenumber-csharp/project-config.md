---
trigger: always_on
description: This file provides guidance to coding agents (including Claude Code, claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents (including Claude Code, claude.ai/code) when working with code in this repository.

## What this repo is

C# port of Google's [libphonenumber](https://github.com/google/libphonenumber). Code was rewritten from the Java source mostly unchanged — when in doubt about behavior, the Java upstream is the source of truth.

The library tracks upstream metadata releases (~every two weeks) via the `create_new_release_on_new_metadata_update.yml` GitHub Action; see commits like "feat: automatic upgrade to vX.Y.Z" for what those changes look like. The action stops when the upstream diff touches `.java` or `.proto` files, since those may need porting by hand; README.md ("Metadata updates") documents the dry-run and check-override options.

## Repository layout

- `csharp/Directory.Build.props` — build settings shared by every project: `LangVersion`, `TreatWarningsAsErrors` (so warnings break the build), the repo-wide `NoWarn` baseline, repository metadata, the NuGet audit settings, symbol packaging (`.snupkg`), and the reproducible-build/Source Link switches. Set things here rather than per-csproj.
- `csharp/Directory.Packages.props` — Central Package Management. Every package version lives here; a `PackageReference` carrying its own `Version` is an error (`NU1008`).
- `csharp/PhoneNumbers/` — main library (NuGet `libphonenumber-csharp`). Multi-targets `netstandard2.0;net8.0;net10.0`.
- `csharp/PhoneNumbers.Test/` — xUnit tests, ported from the Java tests. Multi-targets `net8.0;net10.0`.
- `csharp/PhoneNumbers.Extensions/` — separate NuGet (`libphonenumber-csharp.extensions`) with C#-idiomatic helpers that don't exist in the Java library.
- `csharp/PhoneNumbers.Extensions.Test/` — xUnit tests for the Extensions package.
- `csharp/PhoneNumbers.PerformanceTest/` — BenchmarkDotNet harness.
- `csharp/PhoneNumbers.BenchmarkTools/` — CI-only console tool that compares two `PhoneNumbers.PerformanceTest` JSON result sets (Welch's t-test via MathNet.Numerics) and writes the significant differences for `run_performance_tests.yml`/`post_performance_test_comment.yml`; paired with `lib/fail-on-benchmark-regression.sh`. Not in the solution; run directly via `dotnet run --project`.
- `csharp/PhoneNumbers.MetadataBuilder/` — build-time tool that converts XML metadata + geocoding/timezone text files into per-region binary files. Source-links a small set of files from `PhoneNumbers/` so it doesn't depend on (and can't cycle with) the main library at build time.
- `csharp/PhoneNumbers.Demo/` — Blazor WebAssembly demo, deployed to GitHub Pages by `deploy-demo.yml`. Doubles as proof the library works trimmed under WASM.
- `csharp/PhoneNumbers.Demo.Tests/` — bUnit tests for the demo.
- `csharp/coverlet.runsettings` — keeps the generated data tables out of coverage instrumentation; passed by the coverage workflow.
- `resources/` — XML metadata (`PhoneNumberMetadata.xml`, `ShortNumberMetadata.xml`, `PhoneNumberAlternateFormats.xml`, `PhoneNumberMetadataForTesting.xml`), plus `geocoding/`, `carrier/`, `timezones/`. **These are copied verbatim from upstream** (`locale/` is the exception: it is generated from the local jdk by `DumpLocale.java`) — do not hand-edit. The library no longer reads them at runtime: the build pipeline emits binary equivalents under `obj/metadata/`, `obj/geocoding/`, `obj/timezones/` which are embedded into the published assembly.
- `lib/github-actions-metadata-update.sh` + `lib/DumpLocale.java` — automation that pulls upstream resources and regenerates `resources/locale/country_names.txt`.
- `csharp/PhoneNumbers.Fuzz/` — SharpFuzz/libFuzzer target for the parsing surface, run weekly by `fuzz.yml`. Not in the solution; see its README and the note in its csproj.

## Common commands

All commands below run from the repository root, which is what the `csharp/…` paths in them assume.

Metadata is built from XML/text into per-region binary files at build time by
`csharp/PhoneNumbers.MetadataBuilder/` (see the `BuildBinaryMetadata`,
`BuildGeocodingBins`, and `BuildTimezoneBin` MSBuild targets in `PhoneNumbers.csproj`).
You don't need to run anything by hand — `dotnet build` invokes the tool. At run time those
binaries are read straight out of the assembly's embedded resources (gzip-compressed) via
`IMetadataLoader` / `BuildPrefixMapFromBin` — no XML or text resource is parsed, and no zip
archive or file on disk is involved.

Build / test:

```bash
dotnet restore csharp
dotnet build csharp --no-restore
# Full test matrix:
dotnet test csharp/PhoneNumbers.slnx
# Faster: net10.0 only (matches the Linux PR check):
dotnet test csharp/PhoneNumbers.slnx -p:TargetFrameworks=net10.0
```

Run a single test (xUnit filter syntax):

```bash
dotnet test csharp/PhoneNumbers.Test --filter "FullyQualifiedName~TestPhoneNumberUtil.TestParseNationalNumber"
dotnet test csharp/PhoneNumbers.Test --filter "FullyQualifiedName~TestPhoneNumberUtil"   # whole class
```

Pack the NuGet packages (mirrors `publish_nuget.yml`; the workflow adds `-p:VersionPrefix=<tag minus "v">`):

```bash
dotnet pack -c Release csharp/PhoneNumbers
dotnet pack -c Release csharp/PhoneNumbers.Extensions
```

Benchmarks:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twcclegg/libphonenumber-csharp](https://github.com/twcclegg/libphonenumber-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
