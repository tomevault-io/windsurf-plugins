---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A C# engine for **Belot** (Bridge-Belote), a 4-player (2v2) 32-card trick-taking game. The
core engine ships as the `BelotGameEngine` NuGet package. The repository's real purpose is to
**evolve a card-playing AI (`SmartPlayer`) and measure each change in ELO** against the
previously committed version — see "The ELO benchmark workflow" below, which is the single most
important thing to understand here. The full rules are in `etc/Rules.md`.

## Commands

The solution lives in `src/`. Library projects target `netstandard2.0`; runnable/test projects
target `net10.0`.

```bash
# Run the unit tests (xUnit)
dotnet test src/Tests/Belot.Engine.Tests/Belot.Engine.Tests.csproj

# Run a single test or class (xUnit filter on fully-qualified name)
dotnet test src/Tests/Belot.Engine.Tests/Belot.Engine.Tests.csproj --filter "FullyQualifiedName~ScoreManagerTests"

# Run the ELO benchmark / simulator — ALWAYS in Release, and it needs internet (see below)
dotnet run -c Release --project src/Tests/Belot.GamesSimulator/Belot.GamesSimulator.csproj

# Play in the console (you are South vs three SmartPlayers)
dotnet run --project src/UI/Belot.UI.Console/Belot.UI.Console.csproj
```

Build cross-platform projects individually with `dotnet build`. Do **not** run `dotnet build` on
the whole `src/Belot.sln`: it includes `UI/Belot.UI.Windows`, a UWP (x86) project that only builds
with Visual Studio / full MSBuild. CI (`.github/workflows/build.yml`) builds the projects
individually for exactly this reason and leaves the UWP project out.

## The ELO benchmark workflow (read this before touching the AI)

The git history is unusual: **most commit messages are the simulator's output**, because every
meaningful change is judged by ELO delta, not by hand. `Belot.GamesSimulator` plays 200,000 games
for each of several matchups (SmartPlayer vs previous version / Dummy / Random) and prints ELO.

The headline matchup is `TwoSmartVsTwoPreviousVersionGames`: the current `SmartPlayer` plays
against `SmartPlayerPreviousVersion`, which **downloads the committed `master` version of
`SmartPlayer.cs` and its strategies from raw GitHub URLs and compiles them at runtime with Roslyn**
(`Microsoft.CodeAnalysis.CSharp`). Consequences:

- The "previous version" is whatever is on GitHub `master`, **not** your local working tree. The
  simulator therefore **requires network access**; offline runs fail to build that opponent.
- The loop for improving the AI is: edit `SmartPlayer` / its strategies → run the simulator in
  Release → confirm `TwoSmartVsTwoPreviousVersion` ELO is meaningfully positive (≈0 means no
  change) and the other matchups don't regress → commit, conventionally pasting the output as the
  message.
- The simulator sets `ProcessPriorityClass.RealTime`, warms up before timing, and runs at
  `Environment.ProcessorCount / 2` parallelism, so numbers are comparable run-to-run. Only Release
  numbers are meaningful.

## Engine architecture

Everything flows through the `IPlayer` interface (`Belot.Engine/Players/IPlayer.cs`). The engine
drives the game and calls players via six callbacks: `GetBid`, `GetAnnounces`, `PlayCard`,
`EndOfTrick`, `EndOfRound`, `EndOfGame`. **To add an AI or a UI, implement `IPlayer`** — the engine
owns all rules and state; players only make decisions.

Control flow, outer to inner:

- `BelotGame.PlayGame(firstToPlay)` — loops rounds until a team reaches ≥151 points (with the
  capot/contract guards in the win check) and returns a `GameResult`.
- `GameMechanics/RoundManager.PlayRound(...)` — deals 5 cards, runs bidding, deals 3 more, plays
  the tricks, scores. Owns the `Deck` and the four players' `CardCollection`s.
- `GameMechanics/ContractManager.GetContract(...)` — the bidding loop (suit ladder, no-trumps,
  all-trumps, double/redouble) until three consecutive passes.
- `GameMechanics/TricksManager.PlayTricks(...)` — 8 tricks. Handles announces (trick 1, resolved in
  trick 2 via `ValidAnnouncesService.UpdateActiveAnnounces`), Belote, trick winners, and
  accumulating each team's won cards.
- `GameMechanics/ScoreManager.GetScore(...)` — full scoring: no-trumps doubling, last-10, capot
  (+90), double/redouble coefficients, hanging points, and the suit/all-trumps **rounding** rules
  (`RoundPoints`).

Decisions returned by players are **always validated** by the engine; an illegal card or bid throws
`BelotGameException`. Rule logic lives in stateless services under `GameMechanics/`:
`ValidCardsService` (follow-suit / must-trump / must-overtrump → returns the legal `CardCollection`;
if only one card is legal the engine auto-plays it), `TrickWinnerService`, `ValidAnnouncesService`.

Player callbacks receive context objects that all extend `BasePlayerContext`
(`PlayerGetBidContext`, `PlayerGetAnnouncesContext`, `PlayerPlayCardContext`) carrying the player's
hand, the bids, the current contract, and the trick/round history.

## Performance is the primary design constraint

The simulator plays millions of rounds, so the hot path is allocation-averse and bit-twiddly.
Match this style when extending the engine:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NikolayIT/BelotGameEngine](https://github.com/NikolayIT/BelotGameEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
