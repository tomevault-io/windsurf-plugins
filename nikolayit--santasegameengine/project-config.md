---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

C# implementation of the **Santase / 66 / Schnapsen / Sechsundsechzig** two-player trick-taking card game. The engine in `src/Santase.Logic` is shipped as the [SantaseGameEngine](https://www.nuget.org/packages/SantaseGameEngine) NuGet package; everything else is AI players, a console UI, and a parallel simulator harness used to benchmark AI strength.

Solution: `src/Santase.sln`. The commands below assume the repo root is the working directory.

## Build, test, run

```powershell
# Restore + build the whole solution (Release recommended; the simulator is benchmark-grade).
dotnet build src\Santase.sln -c Release

# Run the AI benchmark / game simulator (net10.0 console app, parallelized to all cores).
# This is the primary regression test for AI changes. Always run in Release.
dotnet run -c Release --project src\Tests\Santase.Tests.GameSimulations\Santase.Tests.GameSimulations.csproj

# Run the human-playable Console UI (net10.0).
dotnet run --project src\UI\Santase.UI.Console\Santase.UI.Console.csproj

# Run the cross-platform MAUI desktop/mobile UI (Santase.UI).
dotnet build src\UI\Santase.UI\Santase.UI.csproj -t:Run

# Run the unit tests via CLI (xunit, ~293 tests across 3 projects).
dotnet test src\Santase.sln -c Release
```

### Unit tests

`Santase.Logic.Tests`, `Santase.AI.SmartPlayer.Tests`, and `Santase.AI.ClaudePlayer.Tests` are xUnit test projects targeting `net10.0` with `Microsoft.NET.Test.Sdk` + `xunit.runner.visualstudio` referenced — they run via both `dotnet test` and Visual Studio's Test Explorer. Approx. counts: Logic.Tests ~268, ClaudePlayer.Tests 26 (neural net / feature encoder / legal-move + player-vs-bot smoke, incl. 3 `ClaudePlayerIsmcts` smoke tests), SmartPlayer.Tests 2.

Tests in `Santase.Tests.GameSimulations/Tests/` (the `*LoggerTests.cs` files) live inside the simulator's `Exe` project and are not invoked by the simulator's `Main` or by `dotnet test` (the simulator csproj is `OutputType=Exe`, not a test SDK project) — they're VS-Test-Explorer artifacts.

### Platform notes

- Every project targets `net10.0`. Before the .NET 10 migration the library + AI projects were `netstandard2.0`, the simulator was `netcoreapp3.1`, and the console UI was `net5.0` — recent commits in `git log` still reference those frameworks if you need to compare.
- `Santase.Logic` is the published [SantaseGameEngine](https://www.nuget.org/packages/SantaseGameEngine) NuGet package — bumping its TFM is a breaking change for downstream consumers (current package version is `3.0.0`, post-migration).
- The third-party AI players in `src/AI/External/*.dll` (`BotskoPlayer`, `NinjaPlayer`, `ProPlayer`) are binary references — no source. They're `.NETPortable` (PCL) assemblies, which load fine from `net10.0`. Treat their `IPlayer` contract as load-bearing for the simulator.

## Architecture

### Layering (dependencies flow downward)

```
   Santase.UI.Console   Santase.UI (MAUI)   Santase.Tests.GameSimulations (+ unit tests)
            │                 │                              │
            └─────────────────┴──────────────┬───────────────┘
                                 ▼
     AI players (ClaudePlayer/ClaudePlayerNeural, SmartPlayer, DummyPlayer, External *.dll)
                                 │
                                 ▼
                          Santase.Logic   ← the engine, the NuGet package
```

`Santase.Logic` has zero non-StyleCop dependencies. AI projects depend only on `Santase.Logic`. **Never let `Santase.Logic` take a dependency on an AI or UI project** — it would corrupt the NuGet package's public surface.

The **MAUI UI (`Santase.UI`) references `Santase.AI.ClaudePlayer`** (alongside `SmartPlayer`/`DummyPlayer`) so it can offer all five AIs as selectable opponents — an `IPlayer`-surface break in ClaudePlayer now also breaks the app build, not just the simulator. The UI ranks the human with an on-device ELO (`Game/PlayerRatingStore.cs`, via MAUI `Preferences`; new players start at 1000 and earn upward) against the fixed AI ratings baked into `Game/AiOpponent.cs` — those numbers come from the simulator's `elo` mode (below) and must be re-pasted if the AIs change. Completed vs-AI games are kept in `Game/MatchHistoryStore.cs` (Preferences, newest-first) and shown on the start page.

Two load-bearing UI facts:
- **Localization is in-code, not `.resx`.** English + Bulgarian live in plain dictionaries under `Localization/` (`AppStrings.cs`), looked up by `LocalizationManager` — no satellite assemblies, so it's trimming/AOT-safe and identical on every platform. XAML uses `{loc:Tr Key}` (a binding to the manager's indexer, so it updates live); C# uses `LocalizationManager.Instance[...]` / `.Format(...)`. Default is the device locale; an in-app toggle (start page) overrides and persists it. Add a string ⇒ add the key to **both** dictionaries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NikolayIT/SantaseGameEngine](https://github.com/NikolayIT/SantaseGameEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
