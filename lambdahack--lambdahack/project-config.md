---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. Humans read it too, arriving from the file listing rather than from `README.md`, which doesn't link here by design --- so don't "fix" that by adding a link. Read README first: this file assumes it. What it does not retell is README's *reading path* --- which file to open first, and the good-first-issues it ends on; the tour's content it does retell, at a reference altitude rather than a newcome
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. Humans read it too, arriving from the file listing rather than from `README.md`, which doesn't link here by design --- so don't "fix" that by adding a link. Read README first: this file assumes it. What it does not retell is README's *reading path* --- which file to open first, and the good-first-issues it ends on; the tour's content it does retell, at a reference altitude rather than a newcomer's. The two overlap deliberately wherever a session needs resident what a human reads in README, and the zones are these: the coding-style and haddock rules, restated in the portable notes below so a session has them resident, README being where a human should read them; the build, test and run recipes --- the dev-build setup, `make play`, `cabal test`, the doctest block, the WASM targets --- repeated here in command form, the doctest and `cabal test` blocks identically once README's four-space indentation is stripped; the playtest battery, `--sniff`, and the hlint and stylish-haskell policy, the same facts with the rationale added; and the architecture --- the client-server split, one client per faction, the six command datatypes, the state monads and the `*Implementation` modules --- where README's tour and the Architecture map below name the same things in the same roles, down to shared parentheticals like "(mostly in `Handle*` modules)" and "(e.g. frontend drawing)". The pointman/leader mismatch is restated as well, but in Architecture rather than in the portable notes this sentence points at. A rename therefore falsifies both files at once --- a module, a Makefile target, a `--frontend*` flag, one of the six datatypes --- so grep both. What has no README counterpart is not a repetition to keep in sync: the verification stamp, Where to look next, the standing checks and the document passes, the four-library split, the CPP backends, Gotchas, and every portable note from Working style onward.

File:line citations were verified against the tree at commit `7b84ed60d` (2026-09-08); the citation pass proves a cited line exists --- this stamp, that it still says what the claim around it needs.

## What this is

LambdaHack is a Haskell game engine library for ASCII roguelike games, bundled with a sample dungeon crawler (`GameDefinition/`). It targets native (SDL2/ANSI/teletype frontends) and WASM (the browser build); a defunct GHCJS browser build survives only as dead example code (see the Frontends section below). The TypeScript browser-side harness lives in `ts-src/`; built WASM artifacts are deployed into the `lambdahack.github.io` repo, expected as a sibling checkout at `../lambdahack.github.io` (visible to sessions only when the wrapper mounts it --- sandboxing notes at the end of this file).

## Where to look next

One directory-scoped file loads on demand: `test/CLAUDE.md`, when a session opens something under `test/`. A human must open it there; the Haskell unit-test harness section below says what it holds.

Frontend work is planned in `docs/wasm-frontend-unified-plan.md`: the SDL2-parity roadmap for the WASM frontend, recorded decisions --- including don't-do rulings, so ideas aren't re-proposed --- and verified repo facts with file:line citations. Consult it before frontend-touching changes. It is kept indefinitely but is not frozen yet, so it is neither of the two kinds below: its ledger gives every item a state, and it borrows that vocabulary from `docs/leader-desync-bug.md` rather than inventing one --- `landed` in a named commit, `open` for code that is wrong today, `not applied` for a design nothing has built, plus `standing` for the items that recur or are review rules and so never land, a case the pointman records do not have. A landing appends an outcome line and flips the row rather than rewriting the item, so the file matures into a record instead of being deleted. A drifted claim is an error to fix while an item is `open` or `not applied`; once `landed` it is left alone, and only a claim that *resolves* earns a further outcome line. Where the two documents' vocabulary ever disagrees the pointman record is right, so a decision taken there needs no glance at the frontend plan.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LambdaHack/LambdaHack](https://github.com/LambdaHack/LambdaHack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
