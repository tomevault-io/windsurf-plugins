---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MATLAB codebase supporting the manuscript *"Identification of Digital Twins to Guide Interpretable AI for Diagnosis and Prognosis in Heart Failure"* (Feng et al.). It identifies patient-specific digital twins of cardiovascular physiology by fitting the **TriSeg** biventricular heart model + lumped-parameter circulation to clinical measurements (echocardiography, RHC, CMR) of heart-failure patients.

There is no build system, no test suite, and no package manifest. Everything runs inside MATLAB (`ode15s`, Optimization Toolbox, Global Optimization Toolbox, Statistics and Machine Learning Toolbox, Parallel Computing Toolbox for `UseParallel`).

## Running the code

Open MATLAB in the repo root. The two canonical entry points are sections of [Driver.m](Driver.m):

- **Section 1 — canonical male/female subject**: set `GENDER` and `Sim` (1 = simulate from saved modifiers, else run `Srdopt` to optimize). Reads `modifiers_male.csv` / `modifiers_female.csv`.
- **Section 2 — heart-failure patients** (`load AllPatients.mat`): loop over `PATIENT_NO` (1..370 for UM cohort, example patient in paper is 192). Set `RUNOPT = 0` to load a precomputed fit from `SimsUMFinal/P_NO<n>.mat`, or `RUNOPT = 1` to launch [HFopt.m](HFopt.m) (GA → patternsearch → fminsearch). `MRI_flag` toggles whether CMR-derived targets are included.

Plotting / inspection scripts (run after `runSim`): [NplotFit.m](NplotFit.m) (HF 6-panel), [NplotSrd.m](NplotSrd.m) (canonical 4-panel), [GetMovie.m](GetMovie.m), [See_TriSeg.m](See_TriSeg.m).

There is no "single test" — the closest equivalent is running one patient via Driver.m Section 2 with `RUNOPT=0` and `print_sim=true` (uncomment in [runSim.m](runSim.m) area; the cost-printing block at the bottom of runSim emits per-target percent error and weighted cost).

## Architecture

The pipeline is a **script-based** dataflow where workspace variables (`targets`, `inputs`, `mods`, `params`, `init`, `t`, `y`, `o`) are passed implicitly between MATLAB scripts. This is not a function-based API — many "steps" are scripts that read/write the caller's workspace. Be careful when refactoring: renaming a workspace variable in one script silently breaks downstream scripts.

### Data flow

```
targetVals_*.m  ──►  estiminiParams.m  ──►  optParams.m  ──►  runSim.m  ──►  Nplot*/GetMovie/See_TriSeg
   (clinical                (initial            (apply           (ode15s on
    targets +                params +            modifier         dXdTDAE/dXdTode +
    inputs +                 initial             vector m)        post-processing +
    mods list)               state)                               cost function)
```

1. **`targetVals_male.m` / `targetVals_female.m` / `targetVals_HF.m`** — return `targets` (clinical measurements to fit), `inputs` (sex, height, weight, HR, etc.), and `mods` (the list of parameter names that are adjustable for this patient). `targetVals_HF.m` also returns `Windowdate` and takes `(patients, PATIENT_NO, ModelWin, MRI_flag)`.
2. **`estiminiParams.m`** — produces `INIparams` (a parameter struct seeded from targets/inputs via allometric and physiologic relations) and `INIinit` (initial state for the ODE). Internally calls **`geom_0.m`** (wall/lumen volumes) and **`calc_xm_ym.m`** (TriSeg geometry: solves for `xm_LV/SEP/RV` and `ym` so initial conditions are mechanically consistent).
3. **`optParams.m`** — applies a modifier vector `m` (multiplicative scalings of the entries in `mods`) to `INIparams`/`INIinit`, returning the `params`/`init` actually used by the ODE.
4. **`dXdTDAE.m`** (DAE form, mass matrix) and **`dXdTode.m`** (pure ODE fallback) — define the right-hand side of the cardiovascular state equations (TriSeg geometry + sarcomere mechanics + 0-D circulation). [runSim.m](runSim.m) tries the DAE form first and falls back to the ODE form on failure (the two have **different state orderings and different output row layouts** — see the two parallel `y(:,k)` / `o(k,:)` blocks at lines ~45–117 and ~145–248). Any change to one form must be mirrored in the other.
5. **`runSim.m`** — solves with `ode15s` (custom timeout via `odeWithTimeout1/2` nested at the bottom), keeps the last 2 cardiac cycles as steady-state, post-processes valve flows / regurgitation / stenosis grades / chamber dimensions / pressures, then computes a **weighted, calibrated, normalized squared-error cost** of `o_vals` vs `targets` plus a `tax` term (sarcomere length, E/A ratio, and — when `MRI_flag==0` — empirical RV passive/active stress constraints loaded from `Kconstrain.mat`). The cost printout uses the symbolic exchange rate `EX = 7.1720` (USD→CNY on 2025-06-24) as a fixed scaling — do not "clean this up", it is a deliberate constant.
6. **`evaluateModel.m` / `evaluateModelUmich.m`** — wrap the above into a single `cost = f(m, ...)` that the optimizers in `HFopt.m` / `Srdopt.m` call. `HFopt.m` uses GA → patternsearch (with `searchga` hybrid) → `fminsearch`, iterated until improvement < 10.
7. **Bounds**: [m_bounds.m](m_bounds.m) returns `[ub, lb]` for the modifier vector based on the names in `mods`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beards-lab/TriSeg-Digital-Twins](https://github.com/beards-lab/TriSeg-Digital-Twins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
