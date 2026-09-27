---
trigger: always_on
description: Standing instructions for `src/Engine/HarmonicBalance`. Read with the root `CLAUDE.md`.
---

# Harmonic Balance engine — local conventions

Standing instructions for `src/Engine/HarmonicBalance`. Read with the root `CLAUDE.md`.
This is the hardest math in the project — design on paper (see `docs/design/harmonic-balance.md`)
before implementing, and keep the pieces below independently testable.

## Parametric-sweep warm-start + quiet diagnostics (harmonic-balance.md §11.1) — COMPLETE

- **Warm-start (continuation) for the generic HB sweep.** `HbEngine.Run(p, warmStart=null)` takes an
  optional interface-V seed; when supplied it is the Newton guess and the per-point
  `NonlinearDcEngine.Run` DC seed is **skipped**. `HbRunResult` carries `Converged` + `InterfaceV`
  (the converged interface spectrum) + `Trace`. `ParametricSweepEngine.RunInner` returns the converged
  seed via an `out` param; `Run` chains it into the next point — **innermost sweep axis only**
  (nested/outer sweeps return a null seed → each outer step restarts cold), resets on non-convergence,
  falls back to cold on a dimension change. Gated by `AnalysisSettings.HbSweepWarmStart` (**default
  true**). **Every tone count since HB-P3 (2026-08-30)** — the seed is `[N,K+1]` single-tone and `[N,M]`
  over the mixing lattice for two or more tones, and each engine path checks the shape it was handed.
  (`RunTwoTone`/`RunMultiTone` took no seed at all before that, so a swept multi-tone analysis
  cold-started every point.) Benchmark: GaN-PA Pin sweep 22→12 Newton iters, 11→1 DC solves,
  bit-identical interface spectrum; two-/three-tone sweeps 8→1 DC solves. Gate:
  `HbPinSweepWarmStartBenchTests` (iteration/DC counts, multi-tone warm-vs-cold, chain reset, plus
  production warm-vs-cold equivalence through `ParametricSweepEngine.Run`).
- **Quiet by default.** The per-solve stderr traces (`[HB]`/`[HB-DC]`/`[HB2D-DC]`/`[HB trace]` and the
  inductance-regularization notice) repeated once per sweep point. They are now gated behind
  `AnalysisSettings.HbConsoleDiagnostics` (**default false**). The regularization itself always runs
  (it converges to the exact answer as R→0); only its console notice is suppressed. Non-convergence
  warnings still flow through the `AddWarning` channel regardless.

## SDD control-current HB Jacobian `J_cc` (brief-sdd-control-current-hb-jacobian, 2026-06-19) — COMPLETE

Restores quadratic-quality convergence for SDD `_cn` references in HB by adding the
control-current Jacobian coupling, FD-oracle gated (`CompareJacobianNumerical` ≤ 1e-5).

- **Two-pass self-consistent `_c_ref(V)`** (`HbNewton.EvaluateNonlinear` → `RunDevicePass`): when an
  SDD has `C[n]` refs, pass 1 evaluates with `_c_ref` frozen at the entry seed (from `iNlPrev`) →
  `iNl(V)` for the current `V`; then `_c_ref(V)` is back-solved per harmonic from the **TOTAL**
  nonlinear injection `iNl + jωq + ΣH·WNl` (not just w=0); pass 2 re-evaluates with `_c_ref(V)`. The
  inner map is a **single linearization step** (NO inner iteration) so `J_cc` is the exact derivative
  of exactly that one-step residual — that is what the FD oracle differentiates. `cc==null` → byte-
  identical single-pass fast path.
- **`J_cc = B·R·A`** (`HbNewton.AddControlJacobian`, added inside `BuildJ`): `A=∂iNl_total/∂V` (the
  main conversion blocks G + jω·C + ΣH·Dw, no Y_NN/Maas), `R=∂_c_ref/∂iNl_total` (rRef rows, harmonic-
  diagonal), `B=∂F/∂_c_ref` (conversion of the per-w control kernels with H[w] weighting). DC-Im
  rows/cols are zeroed (Maas fictitious-DOF parity); `A`'s DC-Im output row is also zeroed (HbFft
  forces the DC bin real). Composition is a dense real matmul — fine since control refs are rare/small.
- **`HbLinearExtractor.ControlSensitivityRow(omega, branchIdx)`**: `rRef[j] = −(G⁻¹)_{branch,node_j}`
  via N forward solves on the cached LU (sign baked in: `∂_c_ref/∂iNl[j]`). Identity pinned by a test:
  `c0 + Σ rRef·iNl == SolveFullNetwork[branch]`.
- **Per-w control sensitivities** (`SddModel.Evaluate`): `NonlinearResult.DControlCharge` (∂Q/∂_c, w=1)
  and `WeightedTerm.JacCtrl` (∂I[p,w]/∂_c, w≥2) join the existing `DControl` (w=0); defaults keep non-
  control devices unaffected. FFT'd into `ControlJacData.Kernels` for `B`.
- **FD oracle wiring**: `CompareJacobianNumerical` / `HbEngine.RunJacobianDiagnostic` take `cc` and a
  **frozen `iNlPrev` seed** (same seed for analytic + every FD eval). Both also take
  `useControlJacobian` (also on `HbNewton.Solve`) — quasi-Newton fallback + the §3.2 tripwire (oracle
  reports ~0.57 without `J_cc`, ≤1e-8 with it).
- **FD-floor lesson**: tiny ad-hoc control circuits hit the FD relative-error floor on near-zero
  off-diagonal entries (`MaxAbsError ~1e-12` but `MaxRelError ~5e-5`) when the SDD is **linear** (no
  harmonic generation) and the network is **purely resistive** (no phase). Gate tests use a gentle
  quadratic SDD term + a reactive element so all entries are FD-resolvable → margins 1e-10..7e-8.
- **Convergence note**: the one-step `_c_ref` seed lags by an iterate, so for *strong* coupling
  (beta≈0.8) the outer Newton is superlinear, not strictly quadratic (J_cc 28 iters vs quasi-Newton
  31 — J_cc still wins). This is inherent to the brief's single-step design, not a bug.
- 10 gate tests: `SddControlCurrentHbJacobianTests.cs`. **Owed follow-ons — now landed** (brief #4 /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [potatobeanradio/circuitRF](https://github.com/potatobeanradio/circuitRF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
