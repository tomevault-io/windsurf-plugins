---
trigger: always_on
description: Guidance for Claude Code when working in this repo.
---

# CLAUDE.md

Guidance for Claude Code when working in this repo.

## Current state (handoff)

This is where the project is. A fresh session on any machine can pick up from here. Read
`docs/twin-spec.md` and `docs/twin-explained.md` first; they are the locked design.

What works. The twin core is built and wired into the autoresearch loop. The tuning baseline, the
twin encoder, the bits-per-spike metric, the noise-ceiling trust metrics, and the two trust gates are
in place and pass on MC_Maze_Small val. With the strengthened baseline (features velocity, speed,
acceleration, position; glm_alpha 1e-3) the velocity-tuning anchor reaches median cc_abs 0.413. The
bidirectional GRU twin (d_model 64) reaches cc_abs 0.667 with 95 percent of neurons above the 0.4
gate and bps 0.233 at a 60 second budget, so `autotwin gate` passes both gates. The base twin config
is in `configs/base.yaml`.

The metric to optimize. Match the vision digital twins: Poisson loss to fit, correlation-to-average
(and its noise-ceiling-normalized form, feve) to judge, with a 0.4 per-neuron keep gate. feve needs
repeated reaches to estimate the noise ceiling, which MC_Maze_Small val does not have, so on small
data bps is a stable proxy to confirm the machinery. On the full dataset feve is the real objective.

The full dataset is now the base config. `configs/base.yaml` points at MC_Maze full (DANDI 000128,
about 2295 trials over 108 conditions with many repeats each), `task.py` optimizes `feve`, and the
objective is averaged over condition-stratified cross-validation folds (`cv_folds=3`). feve needs
condition repeats in the eval set, which random folds destroy, so `prepare_cv_folds` stratifies
WITHIN each condition: it round-robins every condition's repeats across the folds, so each fold's
eval holds a share of all 108 conditions (5 to 9 repeats each) and feve is computable per fold while
the split rotates. `configs/experiments/twin_full.yaml` and `tuning_full.yaml` remain for a one-off
`train_twin.py --config ...` baseline-vs-twin check.

The metric had a bug, now fixed (read this). The first feve compared single-trial predictions to a
trial-averaged noise ceiling and dropped the /R on the noise floor. Because the twin predicts from
single-trial hand movement, its per-trial prediction captures within-condition variance the ceiling
counted as noise, so feve ran past 1 (test feve 1.098) and swung about 0.06 with the seed. `_ceiling`
now computes the standard Schoppe/Cadena feve at the PSTH level: average the prediction to a PSTH,
noise floor = across-repeat variance / R, feve = 1 - (mse(observed_psth, model_psth) - floor)/signal.
A perfect model gives 1 (synthetic: 1.107 at R=8 to 1.002 at R=100), an imperfect model gives below
1, and cc_norm = sqrt(feve), so cc_norm squared equals feve.

Verified results under the corrected metric. The best config under 3-fold condition-stratified CV
scores val feve 0.924 (fold spread 0.017), cc_abs 0.875, bps 0.353. The same config at seed 1 gives
feve 0.905, cc_abs 0.873, so the seed gap shrank from 0.06 to 0.019 (the residual is finite-sample
bias at 5 to 7 repeats, not a bug). On the held-out test split, scored once off-search: feve 0.952,
cc_abs 0.885, bps 0.358, all neurons above the 0.4 gate. Test slightly beats val on every metric, so
the twin generalizes with no overfitting. This is a strong, trustworthy motor cortex digital twin.

The twin has largely converged (cc_abs about 0.88, near the ceiling). feve is now trustworthy but
still carries small finite-sample noise at this repeat count, while cc_abs and bps are the most
seed-stable, so watch all three. Do not overindex on the test number: it was read once and must not
become a selection target.

Why one worker on the M1: two workers time-slice the one GPU and give each experiment a different
step count under the fixed wall-clock budget, which confounds the objective. On a multi-GPU box this
does not apply.

What has not happened yet. The long search. The movement-captioning experiment (search the movement
that most drives a well-predicted neuron, then describe it in language) is later work, not this phase.

The current run. The search runs on the M1 with one worker on the CV objective:
`autotwin loop --proposer llm --budget 500 --duration-s 43200` (12 hours), after `autotwin seed`.
Each experiment is about 7 to 8 minutes now (3 folds). It is operator-invoked. Stop it any time with
`autotwin stop`, which writes `journal/STOP`; the worker finishes its current experiment and exits.
On a GPU box the same command runs, and `autotwin sweep --workers 2` is fine there.

Launch it detached, or it dies with the session. When an automated session (like this one) starts
the run, a normal background job gets killed when the harness reaps it. Launch through
`scripts/launch_detached.py` (a new-session `Popen`, since macOS has no `setsid`), for example
`python scripts/launch_detached.py /tmp/sweep.log caffeinate -is uv run --extra nlb autotwin loop
--proposer llm --budget 500 --duration-s 43200`. `caffeinate` keeps the Mac awake. A person running
it in their own terminal does not need this.

Which model actually proposed. The llm proposer records the model that truly answered each call, not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [egealtan/motor-digital-twin](https://github.com/egealtan/motor-digital-twin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
