---
trigger: always_on
description: Guidance for AI agents (and humans) working **in this repository**. If you are instead trying to *use* this library in your own application, read the skills under [`.claude/skills/`](.claude/skills/), they are written for consumers and are the canonical guide to the key/value and transaction APIs.
---

# CLAUDE.md

Guidance for AI agents (and humans) working **in this repository**. If you are instead trying to *use* this library in your own application, read the skills under [`.claude/skills/`](.claude/skills/), they are written for consumers and are the canonical guide to the key/value and transaction APIs.

## What this repository is

A C#/.NET binding for [FoundationDB](https://www.foundationdb.org/) (a distributed, ordered key/value store), plus the general-purpose libraries it is built on. The main solution is **`FoundationDB.Client.slnx`** (the new XML `.slnx` format). Current version: see [`Common/VersionInfo.props`](Common/VersionInfo.props) (`7.4.x`).

The repo holds **two distinct product families**:

- **`SnowBank.*`**, general-purpose foundation, *not* FoundationDB-specific. `SnowBank.Core` is the bedrock (Slices, the Tuple encoding, the CrystalJson stack, collections, async LINQ, UUIDs). Also `SnowBank.Shell`, `SnowBank.Serialization.Json.CodeGen` (a Roslyn source generator), `SnowBank.Networking.*`, `SnowBank.Testing.*`.
- **`FoundationDB.*`**, the actual binding. `FoundationDB.Client` is the core (native interop, transactions, keys/values, subspaces, the Directory layer, tenants, DI). `FoundationDB.Layers.Common` holds demo layers (Map, Index, Vector, Queue, Counter, Blob, …). Aspire, FakeDb, Testing, BindingTester, and the `Fdb*` tools sit around it.

### Dependency direction (do not invert)

```
SnowBank.Core  ◄─  FoundationDB.Client  ◄─  FoundationDB.Layers.Common  ◄─  Layers.Experimental, Linq.Providers
     ▲                                                                  
SnowBank.Shell, SnowBank.Networking.*, SnowBank.Testing.*, SnowBank.Serialization.Json.CodeGen
```

`SnowBank.Core` has **no** project references and must never depend on FoundationDB. `FoundationDB.Client` depends only on `SnowBank.Core`. Tooling/tests reference downward only.

## Build & test

```bash
dotnet build FoundationDB.Client.slnx                  # DEBUG build of everything
dotnet test <Project>.csproj -f net11.0                # ONE suite; always scope the TFM, see "Tests"
```

- **SDK**: pinned in [`global.json`](global.json) to a **.NET 11 preview** SDK (`rollForward: latestMinor`). `LangVersion` is `preview`.
- **Target frameworks**: libraries multi-target `net11.0;net10.0;net8.0` (see [`Directory.Build.props`](Directory.Build.props)). Each project builds once per target. Several projects **also build a `netstandard2.0` "lite" variant**, see [The netstandard2.0 / net472 "lite" targets](#the-netstandard20--net472-lite-targets) below; keeping it green is part of any change to those projects.
- **Build output** goes to `artifacts/` (`ArtifactsPath`), not per-project `bin/obj`.
- **Central package management**: all versions live in [`Directory.Packages.props`](Directory.Packages.props). Add/bump packages there, not in `.csproj` files. **Version policy**: this SDK is a BOM for downstream applications, each pin should be the **latest version that still supports its TFM bucket** (the .NET platform wave is multi-TFM, so net8/net9/net10 usually share one version; ASP.NET Core packages are single-TFM and stay capped per bucket). Deliberate exception: `Microsoft.CodeAnalysis.*` stays low, because it sets the minimum Roslyn/SDK for consumers of the `SnowBank.Serialization.Json.CodeGen` source generator.
- **As a submodule**: a parent repo can override targets via `CoreSdkVersions` (or the finer `CoreSdkRuntimeVersions` / `CoreSdkToolsVersions` / `CloudSdkRuntimeVersions`), and disable the lite targets via `CoreSdkNetStandardEnabled=false`, in its own `Directory.Build.props`. The override import is gated on a `.git` *file* check; bypass with `FDB_BUILD_PROPS_OVERRIDE=1`.
- **Standalone (isolated) build**: to build, test, or pack THIS repo with its own COMPLETE target set (net8/net10/net11 + netstandard2.0 + the net472 validation targets) while it sits under a target-trimming parent, force `CORESDK_STANDALONE_BUILD=true`, which makes `Directory.Build.props` ignore the parent import. **Pass it as an MSBuild `-p:` property, NEVER as an `$env:`/`export`**: a persistent env var leaks into the shell and silently forces every later `dotnet` command into a standalone build (this bit us once; the pack/build scripts were fixed to pass `-p:`). Use [`scripts/build.ps1`](scripts/build.ps1) / [`scripts/build.sh`](scripts/build.sh) for an isolated `restore`+`clean`+`build`, and [`scripts/pack.ps1`](scripts/pack.ps1) / [`scripts/pack.sh`](scripts/pack.sh) for packaging (packing MUST be standalone, or the `.nupkg` ships an incomplete target set).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SnowBankSDK/foundationdb-dotnet-client](https://github.com/SnowBankSDK/foundationdb-dotnet-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
