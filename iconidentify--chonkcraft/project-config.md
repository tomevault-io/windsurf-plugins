---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A native Java recreation of Warcraft II whose target is parity with **the
retail Battle.net Edition**: the same numbers, order of operations and behavior,
including the parts that look like mistakes. Formal GPL ancestry and
attribution live in README and nowhere else in the current tree.

The pinned retail executable and authenticated captures are the behavioral
authority. `tools/bne-harness/` is how the binary gets read. A behavior with
only historical-source support is unverified and must say so.
[CONTRIBUTING.md](CONTRIBUTING.md) is authoritative; STATUS.md states the current
release posture, and docs/architecture.md holds the design decisions.

## Commands

Maven 3.9+ must be on `PATH`. Everything else goes through the pinned
JetBrains Runtime 25 SDK, fetched automatically by `scripts/jbr/with-jbr-25.sh`.

```bash
scripts/jbr/with-jbr-25.sh mvn -DskipTests install   # compile everything
scripts/run-tests.sh                                 # full reactor test run on the pinned JDK
scripts/run-tests.sh -pl engine -Dtest=MissileGeometryTest   # one module, one class (args forward to Maven)
scripts/check-setup.sh                               # which external inputs this machine has
scripts/run-launcher.sh                             # first-run, packs, updates, and Play
scripts/run-game.sh [ALAMO.PUD]                      # play; a map name skips the menu
scripts/build-asset-pack.sh                          # installation -> chonkcraft.chonkpack
scripts/check-docs.py --check                        # the documentation gate CI runs
scripts/ci/check-test-skips.py --profile full        # assert the skip count after a run
```

Developer tools may use these external inputs (the property wins):

| Input | Property / env var | Without it |
|---|---|---|
| 1995 Warcraft II installation | `-Dwc2.install.dir` / `WC2_INSTALL_DIR` | most engine and desktop tests skip |
| Asset pack | `-Dchonkcraft.pack` / `CHONKCRAFT_ASSET_PACK` | retail-data tests skip; the player cannot start |
| Opus test vectors | `-Dopus.testvectors` / `OPUS_TESTVECTORS` | the codec conformance tests skip |

## Skips are not passes

Tests needing retail assets call `Assumptions.assumeTrue(...)` and skip; Maven
reports `BUILD SUCCESS` either way. The exit code alone certifies nothing.
The 17-lane playability gate fails if any selected referee skips and runs with
no external source-tree input. Never report a passing run
without saying which pack and gate were used.

## Architecture

Seven Maven modules; reactor order is dependency order:
`assetpack` (pack format + codecs, depends on nothing), `runtime` (vendored
desktop foundation), `data`
(archive/graphics/PUD readers -- the only module that knows what an
installation directory is), `extractor` (installation -> pack),
`launcher` (media import, pack selection, game updates),
`engine` (the deterministic native simulation), `desktop` (screens, input, sound
policy, replaceable game entry point).

Walls enforced by tests, not convention -- do not weaken them:

- `extractor/IsolationTest` fails the build if the extractor mentions
  `engine`, `desktop` or `runtime`. Anything both sides need belongs in
  `assetpack/` or `data/`.
- `NoInstallDirectoryTest` in both `engine` and `desktop` fails on any
  reference to `Warcraft2Install`, `WarArchive`, `CdAudio` or `CdImage`.
  The game reads an `AssetSource`; locating an install is `data`'s job, via
  `InstallSource.fromEnvironment()`. The skip-message strings that quote
  `wc2.install.dir` are fine and load-bearing -- CI greps for them.
- `runtime/` is vendored source-identical from seven-days-to-tomorrow.
  Never edit it here; changes go upstream and sync in via
  `scripts/sync-runtime.sh`. ChonkCraft-specific code goes in `engine/`.

`engine` package names mirror the C++ `src/` tree where practical. The player
runtime is game JAR plus authenticated BNE pack only. Typed catalogs in the JAR
own definitions, presentation, campaigns and native trigger programs; the pack
owns retail maps, media and decoded tables. There is no interpreter, executable
script loader, external source input or separately installed content tree.

## Determinism

Lockstep multiplayer and integer-seed saves depend on it:

- All simulation randomness goes through `World.syncRand()`. `Math.random`,
  `new Random`, `System.nanoTime` are forbidden in `engine/src/main` and
  currently at zero occurrences.
- If any code path can draw from the stream, every path must draw -- an early
  return around a draw is a desync. Never roll at parse time.
- `LinkedHashMap` is the default map; anything map-derived that reaches the
  wire is sorted explicitly; comparators end on a total tie-break.
- Every path into the simulation goes through `CommandApplier`, single player
  included. The simulation never draws or plays sounds.
- `World.CYCLES_PER_SECOND` is 30 and is behaviour, not a rendering choice.

## The recurring bug, and what tests must do

Most defects here share one shape: a field parsed from real data, given an
accessor, documented, and read by nothing. Use the playability gates, retail

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iconidentify/chonkcraft](https://github.com/iconidentify/chonkcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
