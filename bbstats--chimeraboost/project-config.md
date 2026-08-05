---
trigger: always_on
description: Pure-Python oblivious-tree GBDT (numpy + numba + sklearn only).
---

# ChimeraBoost — working notes for Claude

Pure-Python oblivious-tree GBDT (numpy + numba + sklearn only).

## Speak English in chat (Nathan's standing complaint — honor it)
- Benchmark-speak belongs in files and tables, not in replies. Lead every
  report with the plain-English takeaway ("depth-4 stopped beating the suite —
  it now loses slightly on average"), then the numbers as support.
- Unpack project shorthand on first use in a message. Not "canary slice
  +0.000%@3 clean" — say "the three canary datasets, where any win would mean
  the suite rewards overfitting, came out exactly flat: what we want."
- No stat fragments as sentences ("W61-L57 mean −0.113%"), no arrow chains,
  no @-counts without saying what's counted. Numbers ride inside sentences
  with their referents, or live in a table whose meaning the prose states.
- Self-check before sending: would this sentence survive being read aloud to
  someone who didn't watch the run? If not, rewrite it.
- Docs, verdict files, and memory stay terse — this rule is about talking to
  a human.

## Hard constraints
- **Pure Python, no heavy deps.** No torch/onnx/foundation-model anything. Filter every idea through this first.
- **TabArena is the ONE sealed holdout** (changed 2026-07-27). Report-only: its results — aggregate or per-task — must never influence a source change. The full run is executed by its authors on their defaults, days of turnaround, and is what the README figure shows. Decisions run on synthetic → Grinsztajn + high-card. The `pub:` public suite (`benchmarks/PUBLIC_PLAN.md`) is **no longer sealed** — it is post-hoc validation, read it freely, and it **never blocks a ship**; it may raise a question but Grinsztajn + high-card still answer it. PMLB is the HP-tuning suite only. A published chart must never run on a suite we tune against; that would be in-sample.
- **North star:** strength vs slowdown Pareto (`benchmarks/make_pareto.py`, `/pareto` skill). Headline axis since 2026-08-02 = **skill scores in two panels** — Brier skill for classification, R² for regression, both 0 at no-skill and 1 at perfect, chosen for readability and because they are not field-relative (adding an arm moves nobody). Expected to be superseded by TabArena scores once the wanted runs are uploaded. Head-to-head win rate (`--metric winrate`) and blended-% are now diagnostics; ship-gating (sign tests) unchanged. Ship only what pushes the frontier. (Elo is a person's name — never "ELO".)
- **Always print the aggregate results table after every benchmark run**, unprompted.

## Benchmarks
- Decision tier is ONE command: `python benchmarks/run_benchmarks.py --decide --seeds 3 --save` → Grinsztajn + high-card + their variant families, reported and sign-tested **per stratum, never pooled** (`compare_runs.py BASE NEW --by-suite`). Pooling would count a variant's rows twice, since a variant is a derived view of its parent. Variant families: `@sus25`/`@sus50` (25%/50% of training rows, test set unchanged — the small-data regime) and `@time` (temporal splits on audited datasets — distribution shift, the regime every random split is blind to). See `benchmarks/VARIANTS.md`. `--list-datasets` previews a run without downloading. **The OpenML one-shot gate (`--openml`) was RETIRED 2026-07-27** — eight of its 29 datasets were exact-name Grinsztajn members, so it partly re-scored the very data it was meant to check independently; the `oml:` keys survive only as a registry for the research scripts. Post-hoc validation is `--public` (22 audited datasets, zero overlap with anything we tune on), never a blocker. HP tuning: `--pmlb --pmlb-fold tune|holdout` (dormant — the one study on it found broad tuning buys nothing that generalises).
- **Run-summary scoring is head-to-head win rate** (RMSE regression / Brier classification, near-solved excluded), with a bootstrap CI and a **median** gap — this is the benchmark's own SUMMARY block and the gating view, unchanged; the north-star *chart* axis is separate and is now skill scores. Never the mean of relative gaps — that is what produced the −144% and −8e21% readings.
- `fit_time` excludes prediction and metric computation; runs are stamped `timing="fit_only"`. Older JSONs are not speed-comparable and the tools warn on a mix.
- Mechanism probe (tier 1): `--synth` = frozen SynthGen prior-sampled suite (`benchmarks/synthgen/`, NO TabArena in any form incl. metadata). Attribute deltas: `benchmarks/synth_report.py BASE NEW`. Any generator change ⇒ VERSION bump + re-freeze + `benchmarks/synthgen/backtest.py` re-validation (gate ≥7/9 vs ledger).
- Sign-test two runs: `python benchmarks/compare_runs.py BASE.json NEW.json [--model ChimeraBoost]`.
- Progress / latest table: `python benchmarks/bench_status.py` (the `/bench` skill).
- **One benchmark at a time** — never two concurrently (core contention corrupts timings).
- Full protocol for shipping a change: `/experiment` skill.

## Close the loop — no loose ends
A record must never outlive the thread it describes. Close it out in the SAME
change that resolves the thread, not in a later sweep.
- **Plan files**: when a program ends, the commit that records the verdict also
  clears that file's open items. No `*_PLAN.md` may sit saying "pending",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbstats/chimeraboost](https://github.com/bbstats/chimeraboost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
