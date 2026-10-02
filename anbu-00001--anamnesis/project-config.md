---
trigger: always_on
description: This file exists so a future Claude (or any agent) can re-grasp this repo in one
---

# CLAUDE.md — orientation for an agent working on Anamnesis

This file exists so a future Claude (or any agent) can re-grasp this repo in one
read instead of re-deriving it from source every session. Keep it true; update it
when the shape of the code changes.

## What this is

`ana` is a local-first, offline, no-LLM CLI for fighting self-deception. You log a
falsifiable belief with a probability (binary) or a credible interval (numeric)
*before* the outcome is known; you resolve it later; the engine scores your
calibration. The thesis: **being able to tell true from false (discrimination) is
not the same as knowing how sure to be (calibration).** The report shows both.

## Architecture (one line each)

- [src/scoring.rs](src/scoring.rs) — **pure `std` math, no I/O.** Brier, log score,
  the CORP Brier decomposition (`corp_brier`, isotonic, reported against its
  noise floor; the exact-value Murphy grouping survives only as a deprecated
  alias), rank-based AUC,
  Lichtenstein–Fischhoff overconfidence, Winkler interval score, coverage, Wilson
  interval, empirical-Bayes shrinkage, an **anytime-valid calibration e-process**
  (`calibration_eprocess`: a mixture of betting martingales, valid under
  continuous peeking — answers "is the miscalibration real, or n-too-small noise?")
  and a **ridge-shrunk logistic recalibration map** (`fit_recalibration` →
  `Recalibration::apply`: `p ↦ σ(a+b·logit p)`, the mechanical self-correction),
  a **bootstrap Brier band** (`brier_ci_bootstrap`, seeded SplitMix64 → reproducible),
  a **recency-weighted EWMA Brier** (`ewma_brier`, a descriptive "lately" trend — *not*
  a control-chart alarm, which false-alarms at an agent's n), a
  **confidence-vocabulary** count (`distinct_forecasts`), a **selective-prediction
  risk–coverage curve** (`risk_coverage`: error among your surest calls vs all — when
  to trust your own judgement), a **stake-weighted Brier** (`brier_weighted`: are you
  miscalibrated on the calls that *matter*?) and a **dialectical-bootstrapping**
  aggregator (`dialectical_mean`: average a first estimate with a "consider the
  opposite" second — an elicitation aid, not a score) and a **conformal interval
  recalibration** (`conformal_width_factor`: the multiplier on your credible-interval
  half-widths that makes them hit nominal coverage — the numeric analogue of the
  recalibration map, the split-conformal quantile of standardized residuals). This is
  the load-bearing core; everything else is plumbing. Types: `Sample` (binary),
  `NumericSample` (interval). Tier 3 reuses the e-process two more ways: per-`kind:`
  (multicalibration — which prediction *type* is really miscalibrated, anytime-valid
  so tiny subgroups can't false-alarm) and on interval coverage (`prob=level`,
  `outcome=contained`) to gate the width correction; `fit_recalibration`'s `(a,b)`
  doubles as the Cox calibration slope/intercept the report reads aloud. The
  **decision gate** `decide` (`Act::{Proceed,Verify,Abstain}` + `Decision`) is the
  operational end: recalibrate the stated `p`, then apply Chow's reject threshold
  `τ = 1 − verify_cost/stake` (proceed iff `p̂ ≥ τ`; abstain below even odds) — a
  number becomes an action, and the bar climbs with the stakes. `mean_boldness`
  (outcome-free `mean(max(p,1−p))`) and `asmd` (absolute standardized mean
  difference, the covariate-balance / missing-not-at-random effect size) feed the
  report's **resolution-discipline** check — is the calibration computed on a fair
  sample of your calls, or a self-selected one?
- [src/evidence.rs](src/evidence.rs) — **the order the sequential test consumes
  claims in**, and the single most subtle thing in the repo. Claims enter by a
  *due key* fixed at creation (`resolve_by`, else creation + a per-kind horizon
  stored on the claim). Ordering by *resolution time* — which shipped before, and
  looks chronological — is not a prefix of any fixed order, and made a perfectly
  calibrated forecaster false-alarm in 100% of simulated runs when the report was
  re-read as claims resolved (0% now). Do not "simplify" this back.
  A due-but-ungraded claim is **priced, not skipped and not fatal**: it multiplies
  in the smallest factor it could possibly have contributed. It used to halt the
  sequence, which was safe but useless — any prefix rule gives `(1−g)/g` usable
  claims, so a 35%-ungraded ledger got 23 of 309 graded calls into the test, and
  partitioning cannot fix that (K short sequences, K-fold mixture penalty).
  Pricing keeps all 236 and reports what the backlog costs. Validity is unchanged:
  the two possible factors average to exactly 1 under the null, so the min is ≤ 1
  and ≤ the true factor, making the wealth a non-negative supermartingale that
  Ville still bounds. Gaps can only *lower* the e-value, never invent an alarm.
- [src/hook.rs](src/hook.rs) — `ana hook <session-start|user-prompt|post-tool|stop>`.
  The Claude Code hooks, in the binary: one code path, no `jq`, and the wording
  comes from `report::verdict` so the hooks cannot disagree with the report. The
  PostToolUse hook auto-resolves `kind:tests-pass` claims **from the command's exit
  status**, recording `resolved_by: "auto"` — the part of a self-graded ledger that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Anbu-00001/Anamnesis](https://github.com/Anbu-00001/Anamnesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
