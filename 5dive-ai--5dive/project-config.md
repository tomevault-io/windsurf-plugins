---
trigger: always_on
description: `tests/*.sh` went 96 → 267 harnesses in 13 days against four harness deletions
---

# 5dive CLI — contributor rules

## The test corpus is TIERED and BUDGETED IN WALL-CLOCK (DIVE-2525)

`tests/*.sh` went 96 → 267 harnesses in 13 days against four harness deletions
ever. A guard is bought once and paid for on **every future change, forever**,
while its benefit stays fixed — so the ledger tilts by construction and no single
decision to add one is ever wrong on its own merits. See
`community/wiki/guards-compound-in-cost-and-nothing-ever-retires-one.md`.

| tier | runs | budget |
|---|---|---|
| `core` (**the default**) | every PR, both CI environments | **300s** per job |
| `nightly` | `full-sweep.yml` — 03:17 UTC, dispatch, **every push to main** (DIVE-2667), and **every PR touching `build.sh`, `install.sh` or `src/**`** (DIVE-2789) | **1320s** per job, corpus split across 3 shards |

- **The PR trigger is ADVISORY. It does not block a merge**, and it is deliberately not
  a required context: it is paths-filtered, so it is absent from ~28% of PRs, and GitHub
  leaves a required check that never reports pending forever. A red there is a signal to
  read, not a gate. It also covers ONE axis — a regression in the harness corpus outside
  the core tier — and **not** the release path (`release-cut.yml` never runs on a PR) or
  the selfcheck probe class, which the corpus never reaches. Do not read "full-sweep runs
  on branches" as "a tree-level regression can no longer land behind a green PR".
- **The nightly tier is not a cheap subset**: 25% of the corpus by file count and **81%
  of it by wall-clock** (1210s of 1481s, measured DIVE-2789), because membership is
  selected on cost. Cost any plan that leans on the core/nightly split in seconds, not in
  files — `community/wiki/a-demoted-tier-is-not-a-cheap-subset-it-holds-almost-all-the-cost.md`.

- **A new harness is `core` unless you say otherwise.** Nothing to write.
- **Over budget, CI FAILS** (`exit 4`, distinct from a test failure's `exit 1`) and
  names the slowest files in the tier. Past the cap a new guard **replaces or
  merges** an existing one — that is the reverse gear, and it is the point.
- **A budget red confirms itself first** (DIVE-2592). PR #395 went red at 356s and
  green at 289s on the *same commit, no rebase* — a 67s swing, all of it in one
  network-priced harness. So when the sum comes in over, the runner **re-times the
  3 slowest files and keeps the smaller sample per file**: noise is one-sided
  (contention and a slow remote only *add* time), so the low sample is the least
  contaminated estimate, while real growth appears in both samples and survives.
  Paid only on the red path; a green run re-times nothing. `--confirm-top=0`
  disables it, which can only make the gate **stricter** — there is no flag that
  confirms more, and no CI job passes one.
- **A budget red says it is a budget red.** `exit 4` beside "0 failed" reads as
  systemic to the author and as flake to the next reader; it is neither. The
  message now states that no test failed, whether the number was confirmed twice,
  and **the smallest set of harnesses that covers the overage** — the actionable
  set, not the top-10 leaderboard.
- **Three ways out, in order:** merge by subject (hundreds of harness *files* for
  one CLI means the unit of organisation is the incident, not the subject —
  folding two files about one subject reclaims their setup cost and drops no
  assertion); retire a guard whose class can no longer occur; or demote:

  ```sh
  # TIER: nightly — 14.3s measured: does not fit the 300s PR core; the nightly sweep runs it.
  ```

  In the harness header (first 40 lines). **The reason is mandatory** — a bare
  `# TIER: nightly` is a refusal, not a default. Demotion moves the cost, it does
  not delete it, which is why it is third and why it must be argued in the diff.
- **Say WHERE you measured, and expect the number to be graded** (DIVE-2555). The
  runner compares every `Ns measured` claim against the clock in the run it is
  already doing: 10% under is reported with the replacement line, 50%-and-3s under
  is `exit 5` (its own code — the remedy is "correct a number", not "fix a test" or
  "retire a guard"). A figure with no environment on it cannot be refuted by the
  next reading, only silently disagreed with: one header claimed `300.0s` while
  the same file measured 335s and 378s on the control plane.
- **Editing a harness always runs it**, whatever its tier: the `changed-harnesses`
  job runs and verdict-probes every harness your diff touches. Tier membership is
  a default, and a default loses to an explicit signal.
- **The budget is spent in a RELATIVE unit** (DIVE-2728). PR #461 red-gated at 322s
  with 234 of 234 harnesses passing and a diff worth +0.1s, while unrelated files ran
  10-36% slower and the file the diff touched moved +0.3%. With 9% headroom against a
  10-36% platform draw, the cap had stopped measuring the corpus. So the runner now
  **times a small calibration workload in the same job** — process spawn, bash
  startup, the CLI's own startup, small file I/O, auto-sized to ~10s, min of 2 — and
  spends the cap in units of it. A uniformly slow VM scales both sides and cancels.
  - **Clamped to 100-150%.** The floor means a fast VM never *tightens* the agreed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [5dive-ai/5dive](https://github.com/5dive-ai/5dive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
