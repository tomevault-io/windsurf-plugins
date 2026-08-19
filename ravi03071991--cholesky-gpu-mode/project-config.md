---
trigger: always_on
description: This is the operating manual the AI agents worked from, lightly trimmed for
---

# AGENTS — the standing instructions

This is the operating manual the AI agents worked from, lightly trimmed for
publication: stale mid-campaign anchors and pointers to archive-only
documents were removed, everything else is as written. Section references
like §7.29 cite the campaign's working ledger, preserved in the private
archive.

Goal: improve GPU Mode leaderboard 776, Cholesky on B200.

**Read `README.md`, then `RULES.md`, then `experiments.md` before doing
anything.** `RULES.md` is binding and is not repeated here.

## Context budget — read this before opening any file

The documentation is deliberately cheap; the code is not. The maps exist so you
never have to open a candidate whole.

| Tier | What | Cost | When |
|---|---|---:|---|
| Always | `AGENTS.md` | ~1.5k | auto-loaded |
| Task start | `README.md` + `RULES.md` + `experiments.md` | ~8.8k | every task |
| Before touching code | `ARCHITECTURE.md` | ~3.5k | any change to a candidate |
| Before proposing | `logs/campaigns.md` — all 35 campaigns | ~7.6k | **anything you think is new** |
| Reference | `logs/profiles.md` · `logs/archive/superseded.md` | 9.1k · 100k | hardware data · provenance |
| **Never bulk-read** | `candidates/*.py` | **~230k for `108`** | grep / ranged-read only |

Everything you routinely need is **~24k tokens including the full experimental
history**. The champion alone is ~230k.

**The rule that makes this work: never `Read` a candidate without `offset` and
`limit`.** The champion (`108`) is **19,482 lines and 55% embedded
CUDA**.
`ARCHITECTURE.md` §2 gives the line span of every block — pick the range, read
that. To find a gate or a symbol, `grep -n` first.

Two facts that cost time if you learn them the hard way, both in
`ARCHITECTURE.md`:

- `custom_kernel` is defined **twice**; the second definition shadows the first.
  The one you find first is not the entry point. (Line numbers in
  `ARCHITECTURE.md` are `071`'s — the champion is 18,565 lines. **`grep -n`.**)
- `_CLU_CUDA` is **not** the dead cluster family — it is the base translation
  unit every engine appends onto.

---

## The two objectives

You are optimising **ranked geomean SUBJECT TO an 8/8 validation pass**. Both
must hold. A file that is faster but validates <8/8 is not an improvement, it is
a regression — `submission_063` is 6.8% faster than the champion and unshippable.

| | Ranked | Validation |
|---|---|---|
| Harness | `harness/modal_eval.py` | `harness/modal_validate.py` |
| Shapes | 15 benchmark entries | 8, a subset |
| Metric | geometric mean of runtime | pass/fail per shape |

Full contract detail: `RULES.md` §3.

## Problem / metric

Input: `batch × n × n` CUDA `float32` SPD matrices.
Output: lower-triangular `float32` factor `L` with positive diagonal, `A = L Lᵀ`.

The checker validates shape, dtype, device, finiteness, lower-triangular
structure, positive diagonal, and reconstruction residual against the original
FP32 input. Correctness is property-based, not elementwise against one library.

Ranking is the **geometric mean** over 15 benchmark entries. Because the geomean
is scale-free, a 2× improvement on any one shape gives the same benefit
regardless of absolute microseconds — so **rank levers by how many cases one
mechanism reaches, not by microseconds**, and never let one route regress.

## Benchmark grid

Case index mapping for `modal_eval.py --cases`:

| Case | Shape | | Case | Shape |
|---:|---|---|---:|---|
| 0 | `4096 × 32` | | 8 | `2 × 2048` |
| 1 | `1024 × 64` | | 9 | `8 × 2048` |
| 2 | `256 × 128` | | 10 | `1 × 4096` |
| 3 | `64 × 256` | | 11 | `2 × 4096` |
| 4 | `16 × 512` | | 12 | `1 × 8192` |
| 5 | `640 × 512` | | 13 | `1 × 16384` |
| 6 | `4 × 1024` | | 14 | `1 × 32768` |
| 7 | `60 × 1024` | | | |

Validation shapes are cases 0, 1, 2, 3, 4, 6, 8, 10 — but with a completely
different data distribution. See `RULES.md` §3.

## Experiment discipline

1. Read `experiments.md` §5 (what worked) and §6 (what did not) **first**, then
   `logs/campaigns.md` for the measurement behind any verdict. §6 is long because
   each row cost real time; re-running any of it is pure waste.
2. Pick one exact route / shape / hypothesis.
3. Clone the current champion into a new candidate file.
4. Make **one** focused change, as an **additive default-OFF gate**, and prove
   gate-OFF translation-unit byte-identity.
5. Validate correctness before benchmarking.
6. Benchmark subset cases first, full grid only if promising.
7. Run `modal_validate.py` — 8/8 or it does not ship.
8. Record the result in `experiments.md` **including negatives**, with candidate,
   hypothesis, result, verdict, next action.
9. Never submit officially. Tell the user which file is ready.

## Before proposing anything

Check it against `experiments.md` §6 and `logs/campaigns.md` §IV — "closed with
data". Twenty-plus mechanisms are already closed with measurements. Do not
re-propose one without new information, and say what the new information is.

Check it against §7 — "measurement landmines". In particular:

- **Run `-Xptxas -v` on every candidate BEFORE quoting any A/B number** (§7.29).
  If **any persistent flow kernel's** register count moves — ⚠ **including the
  one your change deliberately modifies** (§7.33) — the A/B is unconfirmed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ravi03071991/cholesky-gpu-mode](https://github.com/ravi03071991/cholesky-gpu-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
