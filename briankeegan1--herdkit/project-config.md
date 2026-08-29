---
trigger: always_on
description: Conventions every agent working in this repo (builder, resolver, coordinator, or a human) must
---

# AGENTS.md — herdkit engine conventions

Conventions every agent working in this repo (builder, resolver, coordinator, or a human) must
follow. This file is the portable, runtime-agnostic sibling of the auto-loaded `CLAUDE.md`: Claude
Code reads `CLAUDE.md` for free, but a non-Claude runtime (grok, codex, …) does not, so the engine
inlines these conventions into every builder's task spec and grounds grok's system prompt from this
file. Keep it short, factual, and current.

## The ship-a-change pipeline (code → PR → gate → merge)

1. **Build ONLY your change in your worktree.** Each item is built in an isolated git worktree; do
   not reach outside it.
2. **Verify your OWN surface, then open a PR.** Run `scripts/herd/healthcheck.sh "<worktree>" --light`
   (per-changed-file syntax) plus any test you added or changed, and get a clean pass. Fix any CODE
   errors; data/env warnings are fine. The whole-project heavy profile is DESCOPED for builders —
   the auto-merge watcher re-runs the full profile as the authoritative merge gate.
3. **The watcher owns the merge.** A builder never merges its own PR. The watcher merges ready PRs
   only after both gates go green (healthcheck + adversarial pre-merge review). If your change needs
   a manual step you cannot perform (a live smoke test, a UI/pane check), declare it in a
   `HUMAN-VERIFY:` block in the PR body — one step per line — which holds the PR for a human approve.

## Ownership boundaries

- **The tracker and `BACKLOG.md` are coordinator-owned.** Builders NEVER edit `BACKLOG.md` and never
  write the work tracker (a Linear/GitHub issue's state, labels, or assignee). A builder that mutates
  tracker state corrupts the queue. The coordinator owns ALL item states.
- **Never read or commit `.herd/secrets`.** Credentials never land in a committed or generated file.
  `DENY_PATHS` stays honored.

## Scratch A/B checkouts

- **To compare your change against a clean base, use a throwaway detached worktree, never `git
  stash`.** A `stash push <pathspec>` + `pop` can strand edits staged-but-reverted; a detached
  checkout is disposable and never touches your live worktree.
- **Name it `scratch-<anything>`** so the sweep's detached-scratch reaper recognizes it on sight
  instead of leaving it to age out:
  ```
  wt="$(mktemp -d)/scratch-ab"
  git worktree add -q --detach "$wt" HEAD
  trap 'git worktree remove --force "$wt" 2>/dev/null || true' EXIT
  ```
- **Pair every `git worktree add --detach` with a `trap … EXIT` remove**, so a crashed or
  interrupted session never strands it. If the trap doesn't fire, it still isn't lost debris: `herd
  sweep` reaps any detached, clean, zero-unique-commit worktree with nothing live inside it — not
  only ones matching a `scratch-*`/`tmp-*` name — right away outside `$WORKTREES_DIR`, or after a
  short age floor inside it. A worktree carrying commits that exist nowhere else is never reaped;
  it's flagged for a human instead.

## Design invariants for new behavior

- **Default-on, kill-switch-off.** New behavior lands with its config key defaulting **ON**. A fix,
  incident containment, or capability withheld behind a default-off flag is a fix nobody gets —
  "merged, default-off, no plan" is an incomplete feature, not a safe landing. The lever exists as
  a HARD no-op **kill-switch**: setting it off restores the prior behavior exactly, so an operator
  can disable a misbehaving change in one config write. This holds for **every** new lever — there
  is no "optional features may ship dormant" exception.
- **Soak is `ENGINE_TRACK`, not default-off.** The safety margin between a change landing and every
  consumer running it is the track split, not a dormant flag: `staging` follows main and gets new
  behavior immediately (it soaks there), `prod` gets it on `herd promote`. Default-off is never a
  substitute for that soak.
- **Byte-identical-when-off.** With the new lever off, output/argv/task-specs/generated files must be
  byte-for-byte identical to before your change. Tests assert this.
- **Fail-soft.** A missing OPTIONAL tool, file, or capability skips SILENTLY — it never produces a red
  row and never aborts a caller running under `set -euo pipefail`. Gate keys fail STRICT (fall back to
  the safest default and warn loudly); cosmetic keys fail soft to the documented default.

## Design invariants for load-bearing infrastructure (GH #964 lessons)

These invariants apply to any change touching retry loops, pane/marker lifecycle, launcher paths,
or health assessment. Violations are the root cause of retry storms and deadlock:

- **Bounded retries on every rail.** Every loop that retries on transient conditions (network timeouts,
  pane spawn failures, verdict collection) must have a NAMED, ENFORCED cap (`REFIX_MAX_ROUNDS`,
  `HEALTH_CONCURRENCY`, `SPAWN_CONCURRENCY`, etc.). A loop that can retry forever is a hanging
  vulnerability. Exhaustion logs `escaped retry cap` or similar, and escalates to a human.
- **Acquire/release pairing for panes and markers.** Every pane allocated (agent spawn, tab create,
  review collector start) must have a matching release (kill, close, reap). Every marker written

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [briankeegan1/herdkit](https://github.com/briankeegan1/herdkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
