---
trigger: always_on
description: Ahtola is an experimental, pure-managed (C#) SQLite-compatible database engine
---

# Copilot instructions for Ahtola .NET

Ahtola is an experimental, pure-managed (C#) SQLite-compatible database engine
vibe-ported from Turso's Rust core. There is **no native companion, no Rust
toolchain, and no P/Invoke SDK** anywhere in this tree. Treat that invariant as
a hard constraint when changing build/package configuration.

## Referencing the Turso source

A read-only `turso-src` git submodule pins the upstream Turso Rust core at a
specific release tag (currently `v0.7.2`, commit `046e9cbf6`) so agents can
read the original Rust sources while porting or comparing behavior, without
cloning ad hoc or guessing at API shape.

```powershell
git submodule update --init --recursive      # first checkout / fresh clone
git submodule update --remote turso-src      # bump to a newer tag/main (see below)
```

When working in this repo:

- Treat `turso-src/` as **read-only reference material**. Never edit files
  there; it is a vendored snapshot of `tursodatabase/turso`, not part of this
  build. Nothing under `turso-src/` is compiled or shipped by Ahtola.
- Prefer it over fetching Turso sources from the web: grep
  `turso-src/core/`, `turso-src/sqlite/`, `turso-src/sync/`, etc. directly to
  find the Rust type/function a C# port mirrors. Cross-reference when a C#
  type's doc comment or the WAL contract names an upstream symbol.
- The submodule pointer is the source of truth for "which Turso version this
  port targets." If you need a newer release, bump the submodule to that tag
  (see below) in the same change that ports the corresponding behavior, and
  note the new tag in the commit message.
- Keep `turso-src/` out of packaging: it must never appear in a nupkg, a
  `Content`/`None` include, or the managed-closure scan. The closure
  validator's native-archive pattern already rejects `runtimes/`/`native/`
  entries; do not add the submodule to any shipped project's item groups.

### Bumping the submodule to a newer release

```powershell
cd turso-src
git fetch --tags origin
git checkout <new-tag>            # e.g. v0.8.0
cd ..
git add turso-src                  # records the new Subproject commit
```

Then update the tag reference in this file and in any commit message. Prefer
release tags over `main` so the reference is reproducible. Do not commit the
submodule pointing at a moving branch tip in a release.

## Build, test, and lint

`build.ps1` (PowerShell 7+) is the canonical entrypoint; there is no Makefile.

```powershell
./build.ps1 restore                    # dotnet restore for the two package roots
./build.ps1 build                      # closure check + restore + build (Debug)
./build.ps1 test                       # full gate: pack, validate, run suite
./build.ps1 pack                       # pack Release nupkgs -> ./artifacts/managed-packages
./build.ps1 validate-package           # pack + consumer restore/build/run/publish across net8/9/10
./build.ps1 validate-project-closure   # regex-scan project files for native/Rust refs
./build.ps1 validate-packed-closure     # validate built .nupkg contents
./build.ps1 format-check                # dotnet format --verify-no-changes
```

Common parameters: `-Configuration Debug|Release` (default `Debug`),
`-Framework net10.0` (default; also `net8.0`/`net9.0`), `-PackageVersion …`,
`-PackageOutput ./artifacts/managed-packages`, `-MinimumExecutedTests 2500`.

`build.ps1 build` always runs the managed-closure check first; it will fail the
build if any `.csproj`/`.props`/`.targets`/`.slnx` references native Ahtola
packages, P/Invoke, or Rust tooling.

### Scripting: cross-platform PowerShell 7

All shell scripting in this repo is **cross-platform PowerShell 7+**
(`pwsh`), not bash/cmd and not Windows PowerShell 5.x. `build.ps1` and
everything under `scripts/` assume `pwsh` runs identically on Windows, Linux,
and macOS. When writing or editing scripts:

- Target `pwsh` 7+ and avoid Windows-only assumptions (no `cmd.exe` chaining,
  no `$env:ComSpec`, no backslash-only paths in string literals that flow to
  cross-platform tools — use `Join-Path`/`Split-Path`).
- Do not introduce bash/sh scripts as build entrypoints; there is no Makefile
  by design. If a CI lane needs a shell, call `pwsh ./build.ps1 …`.
- Avoid aliases that differ across hosts (`wget`, `curl`, `sed`); use
  PowerShell cmdlets (`Invoke-WebRequest`, `Get-Content`, `-replace`) so the
  same script works everywhere.
- Keep line endings consistent with the repo (CRLF is normal here); don't
  mix LF shell scripts into a CRLF tree.

### Running a single test or filtered subset

Prefer the wrapper so the run is still proven to have executed (it parses TRX
and fails on empty/silent runs):

```powershell
pwsh ./scripts/Invoke-ManagedTestSuite.ps1 `
  -Framework net10.0 `
  -Filter "FullyQualifiedName~AhtolaEncryptedStorageTests" `
  -MinimumExecutedTests 1
```

Or directly with the SDK (no execution-count guard):

```bash
dotnet test src/Ahtola.Tests/Ahtola.Tests.csproj -c Debug -f net10.0 \
  --filter "FullyQualifiedName~AhtolaEncryptedStorageTests"
```

The EF Core provider version is pinned. Build the whole graph with:

```bash
dotnet build Ahtola.slnx -c Release
```

`scripts/Invoke-ManagedTestSuite.ps1` supports `-KnownGapFailurePattern` +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Devolutions/ahtola](https://github.com/Devolutions/ahtola) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
