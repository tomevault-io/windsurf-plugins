---
trigger: always_on
description: Never state that a task, feature, fix, or milestone is complete unless `make test`
---

# doover — agent working rules

## Prime directive: no green, no claim

Never state that a task, feature, fix, or milestone is complete unless `make test`
has just been run and exited 0. If tests fail, report the failure output verbatim.
"It should work" is not a result. The CI `honesty-canary` job exists to prove that
failure reporting works; do not remove or weaken it.

## Workflow: tests first

Every build step follows: write/extend tests → confirm they fail (red) → implement →
confirm green → only then claim done. Build order and per-step test gates are in
`../doover-implementation-plan.md`; product behavior is specified in
`../doover-mvp-spec.md`. Do not start step N+1 while step N's gate is red.

## Commands

- `make test` — full local gate: fmt check + clippy (-D warnings) + unit + e2e (bats)
- `make unit` / `make e2e` / `make fmt` / `make clippy` — individual suites
- `make canary` — verifies failures are reported (expects the canary test to FAIL)

## Layout

- `crates/doover-core` — library: registry, parser, snapshot, journal, hooks, undo
- `crates/doover` — CLI binary
- `crates/doover-core/registry/` — reversibility data (YAML, CC0-licensed; code is
  Apache-2.0). Lives inside the crate so `include_str!` embedding survives publish.
- `tests/corpus/parser/` — data-driven parser cases (YAML)
- `tests/fixtures/hook-events/` — golden Claude Code hook payloads
- `tests/e2e/` — bats scenarios; ALL run inside mktemp jails with HOME overridden

## Hard rules

- Never pipe a gate command through grep/tail/head when deciding success —
  the pipe replaces the exit code and a red gate ships as green. Capture to a
  log (`make test > target/gate.log 2>&1`), check `$?`, then read the log.
  This shipped a red commit to main once already.

- E2E tests must never touch the real `$HOME`, `~/.claude`, `~/.doover`, or any
  user data. Fixture jails only.
- docs/CLEANROOM.md lists unlicensed repos (ccundo, DiffBack) we may study but
  must never copy code from. Clean-room only. (Moved out of NOTICE.md in round 2:
  NOTICE propagates downstream and must carry attribution only.)
- Unknown/opaque shell constructs must never classify as `safe` — `unknown` or
  stricter. This is a load-bearing safety invariant with property tests behind it.
- Exit codes: 0 ok, 1 runtime error, 3 undo conflict. Exit 2 is RESERVED
  (it was specced as "hook block decision"; blocking was retired with
  ask-mode and no code path produces it — if blocking ever returns, the
  fail-open catch_unwind wrapper needs a deliberate carve-out). 64
  not-implemented is retired — every subcommand is implemented.

## The user-#1 trial (2026-07-14) — read this before writing another audit

Twenty-one adversarial audit rounds, a clean 15/15 mutation campaign, and a green
gate. Then ONE live agent used doover on a real machine for ten minutes and found
four bugs, two of them serious. The audits were not sloppy — they were looking in
the wrong place. **Correctness of the parts was never the bottleneck; contact with
reality was.** Prefer one real trial to another audit round.

The headline bug is the one to remember: `doover undo` REFUSED to restore a user's
files ("cannot be undone: status is Undone") while the snapshot sat intact in the
store. A `--force` undo of a LATER action had restored a world state that
re-applied an earlier action's effect, so the row said `Undone` while the files
were gone again. Both directions refused; the user was told their data was
unrecoverable when it was not.

**The lesson, and the rule: the status column records what doover DID. Only the
filesystem knows what is TRUE.** Any gate that refuses recovery must be answered by
the conflict oracle against the live filesystem, never by bookkeeping. If you find
yourself writing `if status == X { return Err(...) }` on a recovery path, stop.

- **DONE (trial): the `Undone` refusal is gone** (undo.rs `select_undo_target`).
  Undo of an undone action defers to the oracle: world == POST → the effect is back
  in force, restore it; world == PRE → genuinely already undone, a clean no-op
  (`UndoReport::already_satisfied`, exit 0, nothing journaled); anything else → a
  real conflict. `record_undo` keeps its in-transaction race guard by default and
  takes `allow_reundo`, which ONLY the engine sets, and only after proving the world
  no longer matches PRE. Two concurrent re-undos would both be admitted and write a
  redundant undo row — benign (restores are idempotent) and vastly preferable to
  stranding a user's data. Pinned by `tests/trial_regressions.rs`.
- **DONE (trial): bare `doover undo` picked a command that changed nothing.** It was
  a SQL `LIMIT 1` on "newest row with a pre-manifest", so it landed on a read-only
  command that had merely been given a defensive cwd snapshot. Now
  `pick_latest_undoable` skips actions whose POST == PRE (changed nothing) and whose
  PRE already matches the live filesystem (nothing to restore). Both predicates are
  about REALITY, not status — which is what lets a KILLED destructive command still
  be found while a killed read-only one is skipped.
- **DONE (trial): redaction moved to WRITE time** (hooks.rs). `doover show` printed
  `Authorization: [redacted]` while journal.db held the bearer token in plaintext,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CaydenChik/doover](https://github.com/CaydenChik/doover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
