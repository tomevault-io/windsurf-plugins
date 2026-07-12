---
trigger: always_on
description: Can we find a training/optimization strategy that learns QI-like solutions, closing the gap between explicit construction (~10^-15) and training (~10^-10)?
---

# precisionMLPs

## Research Question

Can we find a training/optimization strategy that learns QI-like solutions, closing the gap between explicit construction (~10^-15) and training (~10^-10)?

Three violations in trained networks explain the gap:
1. **Gamma scaling**: gamma stays O(1) instead of growing as O(N)
2. **Weight blowup**: outer weights diverge instead of staying O(1)
3. **Rank saturation**: features collapse instead of uniform utilization

## How to Succeed
1. Always get context of the research! The papers/theory guiding the experiments are in the /papers/ folder. The main paper is `papers/QIs_workshop.pdf` ("Constructing Machine-Precision Neural Networks with Quasi-Interpolants"). NOTE: Section 3 (the construction) is NOT yet updated in that PDF -- read `papers/Section_3_Rewrite.pdf` IN ADDITION to `papers/QIs_workshop.pdf`, for the current construction (do not just read the rewrite on its own, it is only a single section rewritten), and `papers/practical_implementation.tex` for the fp64/mpmath implementation details. We are trying to complete this paper by finding an optimization strategy.
2. Use the additional repo 'continuous-mlps' (next door neighbor to this repo in the file structure) as inspiration or a resource when you need it (it is a correct implementation of the paper), but not as something to just copy exactly.
3. read docs/future_experiments.md every time. This is our main design spec doc that I will be working with you through.
4. When implementing new machinery or experiments, always write and clearly communicate to me the tests that verify your implementation actually matches the research (e.g. show me the QI construction reaches machine eps precision after being first built, etc.)

## Architecture

```
papers/                       Source material guiding everything
src/                          Core library (PyTorch, all computation in float64)
  config/
    schema.py                 ExperimentConfig and sub-configs (dataclasses)
    loader.py                 YAML load/save, sweep expansion
  models/
    layers.py                 GammaLinear, GammaExpLinear, StandardLinear
    mlp.py                    QIMlp: single-hidden-layer tanh MLP
    freeze.py                 requires_grad freezing utilities
  construction/
    qi_mpmath.py              High-precision QI via mpmath Toeplitz solve
    readout.py                Feature matrix Phi, exact readout solve (numpy/scipy)
    initialize.py             Project construction into model params
  data/
    targets.py                TargetFn registry (6 categories)
    sampling.py               Sampling functions (equispaced, uniform, Chebyshev, QI grid)
    dataset.py                build_dataset() -> Dataset dataclass
  training/
    optimizers.py             Optimizer dispatch (Adam, LBFGS, SGD)
    losses.py                 MSE, Lp, hybrid boundary
    train_loop.py             Multi-stage training orchestration
    metrics.py                MetricsCollector: uniform metric set across experiments

experiments/                  One FLAT folder per experiment (expXNN_name), each with config.yaml + run.py.
                              Flat (not nested under checkpoints) because run.py uses REPO_ROOT = parents[2].
  # Checkpoint A -- numerical validation, method justification
  expA01_numerics_sanity/        Numerics sanity checks
  expA02_qi_vs_lstsq/            QI construction vs least-squares readout (lstsq is superior)
  expA03_coeff_nullspace/        Coefficient closeness / readout null space
  expA04_activation_conditioning/ tanh O(1) vs GELU O(N) null-space regimes
  expA05_weight_blowup/          QI vs lstsq readout norm (no blowup; norm decays with width)
  # Checkpoint B -- scaling laws + noise robustness
  expB01_sampling_and_noise/     Centers vs samples; y-noise; 1/sqrt(n) law
  expB02_scaling_laws/           Width + data scaling, multi-activation (linear-then-floor)
  # Checkpoint C -- how much the geometry matters
  expC01_lambda_tradeoff/        U-shaped error curve in lambda (QI + lstsq)
  expC02_lambda_vs_frequency/    Optimal lambda constant across frequency
  expC03_lambda_basin/           Robust basin: lambda* ~ 0.25, gamma*/N ~ 0.10
  expC04_center_geometry/        Center-placement comparison (uniform vs others)
  expC05_geometry_interpolation/ center/weight/bandwidth perturbation; one-way coupling; reparam argument
  expC06_soft_neuron_interp/     soft-neuron hump (low-degree polynomial basis); cascaded-geometry lead
  # Checkpoint D -- can optimizers find the geometry
  expD01_geometry_ladder/        Adam on frozen geometry stalls; lstsq solves (Phase 1)
  expD02_adam_geometry/          Init x training-regime cube (QI-init + refit wins)
  expD03_reparameterization/     Log-gamma / dimensionless coordinates (stub, live future)
  expD04_varpro/                 Variable Projection reduced objective (stub, future)
  exp13_solution_basins/         Hessian / basin landscape (stub, DEPRIORITIZED -- curvature ruled out)
  # Checkpoint E -- extend to 2D
  expE01_geometry_zoo_2d/        Six 2D ridge geometries head-to-head (hex folded in; ex-exp11)
  # Checkpoint F -- applications (STUB, experiments TBD)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samlayton99/precision-mlps](https://github.com/samlayton99/precision-mlps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
