---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and other coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) and other coding agents when working with code in this repository.

## Overview

This is the Haskell library for Hegel, a universal property-based testing framework. The library drives a Hypothesis-style engine in-process via FFI to the `libhegel` C library.

```bash
just check                                           # run check-format + build + test (CI gate)
just test                                            # run tests
just test <name>                                     # run a specific test suite (e.g. just test ffi)
just lint                                            # STUB: run linters (add hlint to flake.nix first)
just format                                          # run formatters (cabal, Haskell, Nix)
just check-format                                    # check formatting without modifying files
just docs                                            # build API docs via haddock
just check-coverage                                  # STUB: check coverage (add hpc-codecov to flake.nix first)
just profile-run <scenario>                          # smoke-run a profiling scenario on the dev build
just profile-space <scenario>                        # capture .prof/heap/eventlog into profiles/O<n>/ (prof_opt=0 for -O0)
just profile-time                                    # hyperfine wall-clock of all scenarios on the default -O1 build
just profile-time-compare                            # per-scenario -O1 vs -O0 A/B into profiles/compare/
cabal test zizek:unit --test-options='--pattern "name"'  # run a single test (tasty --pattern glob)
```

Minimum supported GHC version is 9.10 (enforced in CI and `zizek.cabal`). If you bump it, also bump `ci.yml`.

## Package Structure

- `library/Hegel.hs` — Public API: `prop`/`forEach`/`forEachWith`; re-exports `Gen`, settings, database, reports, phases, and assertions
- `library/Hegel/Property.hs` — Property monad public API: `PropertyT`/`Property`, `forAll`/`forAllWith`/`forAllWithLabel`/`forAllSilent`, `annotate`/`footnote`, `assume`/`discard`, `registerFinalizer`, `check`/`check_`, `assert`/`failure`, `(===)`/`(/==)`. Internals in `library/Hegel/Property/Internal.hs`. `registerFinalizer act` queues per-case cleanup drained (LIFO) at the case boundary on every exit and replay; a throwing finalizer aborts the run as `Errored`. `forAllWithLabel "qty" g` labels a draw so the report reads `restock item="apple" qty=5` instead of a bare positional value — the fix for cryptic rule rows (no source parsing).
- `library/Hegel/Stateful.hs` — stateful (model-based) testing: `Machine`/`Rule`/`Invariant` and `run`, layered on `PropertyT` (see Stateful Testing below)
- `library/Hegel/Pool.hs` — engine-managed pools of values for stateful rules to draw from; an empty-pool draw discards the case. `named` labels a pool's values for the report; `transfer` moves a value between pools with the identity link declared (one lifeline across pools)
- `library/Hegel/Report.hs` — `Report`/`Result`/`Stats` plus the plain/ANSI renderers: what a property run produces; for step-structured (stateful) failures the rich path composes the report (event log + splice + footer)
- `library/Hegel/Report/*.hs` — the rich renderers: `Ann` (annotations/styles), `Discovery` (declaration lookup), `Source` (splicing/layout), `Span`, `Note` (journal entries; also `renderValue`), `Journal` (depth regrouping + structured rendering), `Stateful` (the failing step's source splice), `Style` (glyphs, phrases, and layout budgets in one record; `HEGEL_GLYPHS` + encoding detection pick ascii); and `Trace` (the IR zipping journal + pool events on their shared `Tick`) with `Layout` (the flat chronological event log: one row per step, a bare `✗`/blank gutter, and touch-irrelevant runs collapsed into a single elision row; also owns `displayName`). Eyeball via `just gallery` (plain splice → stateful splice → pool-free flat log → multi-root flat log); design rationale in `notes/design/slim-stateful-reporting.md`
- `library/Hegel/Diff.hs` — structural and line-level diffs backing `(===)` failures
- `library/Hegel/Assertion.hs` — `assert`/`failure` (`MonadIO`-polymorphic, call-stack-aware), failure-origin formatting
- `library/Hegel/Hspec.hs`, `library/Hegel/Tasty.hs` — framework integrations with automatic database keying (see Framework Integrations below)
- `library/Hegel/Settings.hs` (with `Backend`, `Database`, `HealthCheck`, `Phase`, `Verbosity`) — run configuration
- `library/Hegel/Runner.hs` — `check`: drives the `libhegel` engine, applies `Settings`, pumps test cases, replays reproduction blobs
- `library/Hegel/Gen.hs` — Umbrella re-export; designed for `import Hegel.Gen qualified as Gen`
- `library/Hegel/Gen/Internal.hs` — `Gen` GADT, combinators (`oneOf`, `filtered`, `assume`, `draw`), `enumerate`
- `library/Hegel/Gen/Builder.hs` — `Build`, `HasMin`, `HasMax`, `HasSize` typeclasses
- `library/Hegel/Gen/*.hs` — per-category builders (bool, integer, float, binary, char, text, regex, uri, uuid, list, set, map, …)
- `library/Hegel/Collection.hs` — `libhegel`-managed variable-length collection handle, used by the list/set/map generators

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MercuryTechnologies/zizek](https://github.com/MercuryTechnologies/zizek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
