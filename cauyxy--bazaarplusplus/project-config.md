---
trigger: always_on
description: Operating rules for AI agents working in this repository (`AGENTS.md` is a symlink to this file). Process rules live here; durable knowledge in `docs/MEMORY.md`, structure in `docs/ARCHITECTURE.md`.
---

# CLAUDE.md

Operating rules for AI agents working in this repository (`AGENTS.md` is a symlink to this file). Process rules live here; durable knowledge in `docs/MEMORY.md`, structure in `docs/ARCHITECTURE.md`.

## Build & Test Commands

The mod targets `netstandard2.1` (C# 12). Game assemblies are resolved via `ManagedPath` — auto-detected from common Steam install paths, or pass explicitly:

```powershell
# Build the mod (Debug, auto-copies to BepInEx/plugins/ if game found)
dotnet build src/BazaarPlusPlus/BazaarPlusPlus.csproj

# Build with explicit game assembly path
dotnet build src/BazaarPlusPlus/BazaarPlusPlus.csproj -p:ManagedPath="D:\Steam\steamapps\common\The Bazaar\TheBazaar_Data\Managed"

# Build both Debug + Release (Release copies to installer repo if present)
dotnet build src/BazaarPlusPlus/BazaarPlusPlus.csproj -t:BuildAll

# Run a single xUnit test project (has Microsoft.NET.Test.Sdk)
dotnet test tests\Architecture.Tests\Architecture.Tests.csproj

# Run an exe-runner test project (no Microsoft.NET.Test.Sdk)
dotnet run --project tests\ChoiceScreenPedestalResolver.Tests\ChoiceScreenPedestalResolver.Tests.csproj

# Format with the repo-pinned CSharpier version
./run.sh format
```

`run.sh` works on macOS and Windows (Git Bash). Subcommands:

- `./run.sh build [--with-bazaaragent] [--fast]` — Debug build (`--fast` skips NuGet restore)
- `./run.sh publish [--with-bazaaragent] [-p:Name=Value ...]` — production build: fetch remote embedded data, run seed gates, then `-t:BuildAll` (Debug + Release) with installer packaging
- `./run.sh fetch-data [-p:Name=Value ...]` — refresh remote embedded data
- `./run.sh restore-locks` — refresh committed NuGet lock files for the six published assemblies
- `./run.sh restore-locked` — validate published-assembly restores in locked mode without updating lock files
- `./run.sh test` — run all test projects under `tests/`
- `./run.sh format` — restore the repo-pinned CSharpier tool and format the source tree
- `./run.sh format-check` — restore the repo-pinned CSharpier tool and fail on unformatted files
- `./run.sh decompile [DllName]` — decompile a single game DLL (default: Assembly-CSharp)
- `./run.sh decompile-all` — decompile all tracked game DLLs
- `./run.sh decompile-ptr [DllName]` — decompile a single PTR game DLL into `decompiled-vptr/`
- `./run.sh decompile-all-ptr` — decompile all tracked PTR game DLLs
- `./run.sh snapshot-managed` — archive the installed Managed dir under `game-libs/`
- `./run.sh build-matrix` — build the source tree against every archived Managed snapshot

Test projects under `tests/` are split per-feature. Some use xUnit + `Microsoft.NET.Test.Sdk` (run via `dotnet test`), others are executable (run via `dotnet run --project`). Check whether the csproj has `Microsoft.NET.Test.Sdk` to determine which.

When changing a direct dependency, edit `Directory.Packages.props`, run `./run.sh restore-locks`, and review the six changed `src/**/packages.lock.json` files with the version change. Do not generate lock files for test projects. Before committing, run `./run.sh restore-locked` so dependency graph drift fails locally, then run the standard build and test commands.

## Logs & Debugging

This mod is a **BepInEx 5.x plugin** (`BepInEx.Core` 5.\*). At runtime, BepInEx writes all console output to disk at `<GameDir>\BepInEx\LogOutput.log` — the sibling of the `BepInEx\plugins\` folder the build copies into. To debug, read that file; mod log lines are structured events shaped `[BPP][<Scope>] event=<id> field=value ...` (logged via `BppLog` → BepInEx `ManualLogSource`). `Debug`-level events only emit from Debug builds; `Info`/`Warning`/`Error` always emit.

For runtime validation that needs launching the game, always launch The Bazaar through Steam (App ID 1617400) so Steam runtime state is present. On macOS: `open "steam://run/1617400"`. On Windows: `start steam://run/1617400`. Do not launch `TheBazaar.app` directly or use `run_bepinex.sh` on macOS — these bypass Steam runtime and cause subtle failures.

## Architecture

Structure lives in `docs/ARCHITECTURE.md`; durable knowledge in `docs/MEMORY.md` (load first); vocabulary in `CONTEXT.md`; rationale in `docs/adr/`. This section keeps only the layering rules — traps, not maps:

- Put reusable adapters over The Bazaar/Unity runtime surfaces in `GameInterop/`. Keep feature workflows, UI state, product policy, filtering/classification rules, upload decisions, and storage orchestration in `Game/`. Do not move logic into `GameInterop/` only because it mentions game enums or DTOs.
- If two features need the same runtime/prefab/static-data behavior, extract the adapter to `GameInterop/<Concept>/` and have both features consume that seam. Do not make one feature import another feature's internal implementation only to reuse a game-runtime adapter.
- Patches may target feature services through `BppPatchHost` (the static service locator; never constructor injection), but shared Harmony reflection helpers or native runtime adapters live in `GameInterop/` or `Infrastructure/`, not inside a feature directory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cauyxy/BazaarPlusPlus](https://github.com/cauyxy/BazaarPlusPlus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
