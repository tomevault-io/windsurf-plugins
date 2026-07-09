---
trigger: always_on
description: `hex-dev` is the development monorepo where new Hex sublibraries are
---

# Hex repo family

`hex-dev` is the development monorepo where new Hex sublibraries are
incubated before they are split out for release. `hex` is the released
aggregate repo; it depends on released split libraries at exact Lake
revisions.

The split repos published from `hex-dev`, in dependency order, are:

- `hex-test-kit` (shared conformance/bench helpers; source: `Hex/`)
- `hex-matrix`
- `hex-row-reduce`, `hex-determinant`
- `hex-bareiss`
- `hex-matrix-mathlib`
- `hex-row-reduce-mathlib`, `hex-determinant-mathlib`
- `hex-bareiss-mathlib`
- `hex-gram-schmidt`
- `hex-gram-schmidt-mathlib`, `hex-lll`
- `hex-lll-mathlib`

Treat this as the current set, not a permanent exhaustive list: more
sublibraries may be released from `hex-dev` later. Computational
libraries are Mathlib-free; `*-mathlib` repos are the Mathlib bridge
layers and should contain correspondence proofs and Mathlib-facing APIs.

## Source-of-truth model: develop here, publish outward

`hex-dev` is the **single source of truth** for every library. All
development happens in this one tree; a single `lake build` (plus the
`bench/` and `conformance/` sub-projects) builds everything together.
The split repos above are **published mirrors**: a dispatchable CI
workflow regenerates each one from the matching content in `hex-dev`,
rewriting their cross-repo Lake pins and committing to their `main`.
Never hand-edit a released repo; change it here and let the sync publish.

Every library uses the same per-library layout (so the publish step is a
near-mechanical copy):

- `HexX/` — library source plus the `HexX.lean` umbrella.
- `HexX/SPEC/hex-x.md` — the library's SPEC.
- `bench/HexX/Bench.lean` — bench drivers (shared `bench/` sub-project).
- `conformance/HexX/{Conformance,EmitFixtures}.lean` — conformance
  drivers (shared `conformance/` sub-project).
- `conformance-fixtures/HexX/*.jsonl`, `scripts/oracle/<lib>_*.py`.

The publish mechanism is `scripts/release/released.yml` (the per-repo
managed-path + pin manifest), `scripts/release/sync_released.py` (the
driver; supports `--dry-run`), `scripts/release/synced.json` (the
per-repo `main` baseline this monorepo corresponds to), and
`.github/workflows/sync-released.yml` (manual dispatch, dry by default).
A real sync overwrites each released repo's managed paths and rewrites
its Lake pins, so it must only run once this monorepo is at or ahead of
every released repo's `main`. Run `--dry-run` first.

**Uncoordinated-commit guard.** The sync refuses to overwrite a released
repo whose `main` HEAD has moved off the recorded baseline, so an
out-of-band commit on a released repo is never silently clobbered; it
skips that repo and reports the divergence (override with `--force` only
after reconciling). Reconciling means **re-seeding**: bring the affected
library's content here up to the released `main`, rebuild the whole graph
green (a released repo can advance with breaking API changes its
downstream consumers have not adopted — the monorepo build surfaces
that), then re-run the sync.

The baseline lives on a dedicated, unprotected `release-sync-baseline`
branch that the workflow reads and advances on every real run, so a single
`workflow_dispatch` (dry-run first, then `dry_run=false`) drives the whole
publish through with no follow-up. `scripts/release/synced.json` is the
bootstrap seed used only before that branch exists.

# hex — agent-specific conventions

Conventions specifically for LLM agents working on this project.
General project doctrine (Mathlib-free split, SPEC/PLAN structure,
key files) lives in `SPEC/` and `PLAN.md`; start there for
orientation.

## Style

Don't add "research completed" timestamps, progress notes, or
meta-commentary about the history of our research process to any
file. The git history tracks that. SPEC files and `PLAN/` contain
the current state of the design, not a journal of how we got there.

## Per-turn progress files

Start of turn: read the most recent file in `progress/` (ISO-8601
timestamps sort chronologically). If only `progress/0000-init.md`
exists, the repo is freshly initialised — proceed with Phase 0.

End of turn: write `progress/<UTC-timestamp>.md` with sections
**Accomplished** / **Current frontier** / **Next step** / **Blockers**.
Scope these to *your* session — what you touched, where you stopped,
what you think comes next for your corner of the project.
Commits made during the turn should mention the progress file.

## Lean

Check diagnostics after every step; don't continue past errors. Build
via `lake build`, not `lean` directly. `native_decide` is banned (see
SPEC).

Never introduce an `axiom`. This includes converting an existing
`theorem`/`def`/`example` into an `axiom` when a refactor breaks its
proof — fix the proof or fix the API. For unfinished proofs use
`sorry`, which is grep-able and produces a warning; `axiom` is silent.

## Directives are hypotheses, not specs

When you claim a directive issue, the body is the author's *current
best understanding* of what the work looks like. It is not gospel.
Before opening a PR, sanity-check the premise:

- Does the type signature the directive asks for actually admit a
  proof, or is it unsoundly typed (allows non-canonical witnesses,
  missing hypotheses, vacuous quantifiers)?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kim-em/hex-dev](https://github.com/kim-em/hex-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
