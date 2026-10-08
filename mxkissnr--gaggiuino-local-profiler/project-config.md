---
trigger: always_on
description: Working rules for this repo. Follow these in every session.
---

# CLAUDE.md — Gaggiuino Local Profiler

Working rules for this repo. Follow these in every session.

## Language rules

- **Code, comments, commit messages, GitHub issues, PR descriptions** → always English
- **DOCS.md, README.md** → English (primary)
- **DOCS.de.md** → German (supplementary, always kept in sync with DOCS.md)
- **UI strings in index.html** → translated via `t()` + `TRANSLATIONS` object (DE/EN/IT/FR/ES/NL); add new keys to **all 6 language files** when adding UI text

## Workflow

> **STOP — issue first, then code. This rule has been violated repeatedly.**
> Do not write a single line of implementation before the issue exists.
> No exceptions for bug fixes, security fixes, refactors, or "small" changes.
> The only exception is a typo or single-word change.

**Step 1 — create the issue (always, before anything else):**
```
gh issue create --repo mxkissnr/gaggiuino-local-profiler --title "..." --label "bug|enhancement" --body "..."
gh project item-add 2 --owner mxkissnr --url <issue-url>
```

**Step 2 — implement the fix/feature.**

**Step 3 — close the issue in the commit message:** `Closes #N`

If you catch yourself writing code without an issue number in hand: stop, create the issue first, then continue.

## Branch workflow

**Work happens on `dev`, not `main`.** Feature/fix branches for a round are cut from
`origin/dev` and PR'd back into `dev` — `main` only receives merges from `dev` at release
time (see "Release & documentation rules" below). This applies to every PR unless the round
is itself the release round. When setting up a worktree, `glp-worktree` auto-detects this
(branches from `origin/dev` when it exists) — no manual `--base` needed for this repo.

## Regression policy

> **A feature or fix must never break already-working functionality.** This
> is a hard constraint, not a trade-off against shipping speed.

Concretely:
- Before "fixing" something that looks wrong, verify against real ground
  truth (an actual shot/profile/data export from Max, existing passing
  tests, or the machine's own documented protocol) — not just plausible
  general theory. A plausible-sounding fix that isn't checked against real
  data can silently introduce a regression that looks like an improvement.
  (Precedent: #323's "fix" to the bean-to-profile suggestion's Decline Flow
  phase direction was based on general extraction-theory reasoning and
  reverted real behavior that matched Max's own live-verified profiles —
  caught only when Max supplied the actual profile JSON for comparison.)
- When changing a function that's called from multiple places (e.g. a
  shared repository/data-layer helper), check every call site's assumptions
  before changing the function's contract — don't assume the call site you
  're looking at is the only one. (Precedent: #327 — `saveOrders()` silently
  deleted any DB row not present in its argument array; every caller passed
  it an already-filtered subset, so the function's real behavior was "wipe
  the whole table" even though no single call site looked destructive in
  isolation.)
- If a fix could plausibly have been silently losing/corrupting data before
  the fix (not just showing a wrong value), say so explicitly and check for
  a recovery path (e.g. `/api/backup` exports) — don't just fix and move on
  as if it were a display bug.
- Run the full existing test suite after every change, not just tests
  related to the change, and treat any newly-failing test as a stop
  condition, not noise to explain away.
- (Precedent: #638/#641/#643/#648 — machine host/switch entity resolution
  was copy-pasted into five files instead of living in one place, and each
  copy accumulated its own version of the same bug; a user-reported bug
  shipped in v2.29.0 anyway because no test proved a *setting change* changed
  behavior, only that saving succeeded.) Two rules that follow from it:
  (a) If a setting is editable in the UI, a test must prove that *changing*
  it changes behavior — a test that only checks the save/round-trip is not
  enough.
  (b) If the same resolution logic is needed in more than two places, it
  belongs in one shared helper — copied logic multiplies its own bugs (#643:
  five copies of the same three-line function).
- **This app supports multiple concurrent machines.** Any new state that
  tracks an in-progress, per-operation value — sync/import progress, live
  counters, displayed totals, timers, "which machine is this about" — must
  be keyed by `machineId`, never a shared scalar/global that the
  last-writing machine overwrites. Design it explicitly for N concurrent
  machines from the start, don't default to a single-machine mental model
  and fix it after review. (Precedent: this exact bug class shipped three
  times before being caught — #730/#732's sync-progress toast tracked one
  scalar "last progress" instead of a per-machine map, and #742/#743's
  live shot-counter display had the same issue with two machines
  backfilling concurrently.)

## Versioning

**The version number only ever changes at release time, on `main`.** Dev/
feature commits never touch it — see #890: bumping per dev commit made
`main`'s version jump by however many increments had piled up on `dev`
since the last release (e.g. 2.34.0 → 2.37.1 in one step), which is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mxkissnr/gaggiuino-local-profiler](https://github.com/mxkissnr/gaggiuino-local-profiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
