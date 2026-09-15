---
trigger: always_on
description: This repository is a curated collection of compact, educational **Julia** algorithms that illustrate the core numerical methods enabling real-time digital twins. It supports a course whose goal is to make students familiar with these algorithms through small, self-contained, runnable examples. Every algorithm should be underpinned by a clear worked example.
---

# Project Guidelines

This repository is a curated collection of compact, educational **Julia** algorithms that illustrate the core numerical methods enabling real-time digital twins. It supports a course whose goal is to make students familiar with these algorithms through small, self-contained, runnable examples. Every algorithm should be underpinned by a clear worked example.

## Language and Style

- Implement all algorithms in **Julia**.
- Optimize for **compactness and expressiveness over efficiency or execution speed**. Prefer short, vectorized, idiomatic Julia (broadcasting, comprehensions, `\` for linear solves) to long imperative loops, unless clarity demands otherwise.
- Favor readability for classroom use: an algorithm should read like the math it implements.
- Add comments where they aid understanding, and **separate distinct tasks into clearly marked sections** with a short header comment (e.g. `# --- Assemble conductivity matrix ---`, `# --- Solve ---`, `# --- Update ---`). Keep the number of sections small and meaningful.
- **Keep in-code comments minimal.** Put longer explanations (method, derivations, rationale) in the top-of-file header comment block, and keep inline comments to concise section headers.
- Keep examples self-contained and minimal so they can be read top-to-bottom in one sitting.
- **Naming:** name algorithm files in `CamelCase` after their primary concept (e.g. `FiniteElement.jl`, `TopologyOptimization.jl`). Define **plain functions** in these files—do not wrap algorithms in a `module`. Use `snake_case` for function and variable names.
- **Type association:** express a function's link to a type through first-argument dispatch (`node_id(g::Grid, …)`), not name prefixes like `Grid_node_id`. Add short docstrings tying accessors to their type.

## Dependencies

- **Use existing libraries for standard subtasks** — e.g. solving linear systems or training ML models — unless the task is specifically to implement that primitive. Time-stepping schemes (implicit Euler, linearly-implicit trapezoidal, …) are themselves the algorithms this repo teaches, so they are implemented by hand rather than delegated to an ODE-solver package.
- **Keep the external dependency footprint minimal.** Prefer a few broad, high-coverage packages over many narrow ones. Reach first for: `Optimization` (with `OptimizationOptimisers` for Adam and `OptimizationOptimJL` for LBFGS/BFGS) plus `Zygote` for gradient-based training, `Lux`/`ComponentArrays` for neural-network models (`LuxCUDA` when a script needs the GPU), and `BSON` for persistence. Only add a new dependency when none of the established ones fit — e.g. `GaussianProcesses.jl` (uncertainty quantification) and `Images.jl` (PNG-based material/source fields) were added because no established package covered that capability.
- Persist generated data as **`BSON`** and produce all plots with **`Plots.jl`**.

## Reuse and Consistency

- **Reuse previously built algorithms** rather than reimplementing them. For example, use an existing simulation algorithm to generate training data for a data-driven algorithm.
- **Be consistent across algorithms**: shared naming conventions, function signatures, section structure, data formats, and utility usage. New algorithms should look and feel like existing ones in this repository.
- When there is reasonable doubt or more than one viable alternative, seek user feedback before proceeding.
- Place broadly useful helpers in `util/` and reuse them instead of duplicating code.

## Repository Structure

- `util/` — reusable utility algorithms shared across the collection.
- `models/` — **physical model examples** (organised by domain, e.g. `models/pde/`) used throughout the lecture. Keep these focused on the model/discretisation itself; no plotting. Static input assets (e.g. geometry PNGs) live in `models/data/`.
- `algorithms/` — the **core algorithms**. Keep these focused on the method itself; no plotting or visualization here.
- `data/` — generated datasets, persisted as `BSON` by the scripts that create them. Nothing under it is synchronized with the repository except the folder's `README.md`; every dataset is regenerated on the fly by rerunning the producing script.
- `results/` — **generated outputs** (figures, persisted solutions) written by `scripts/`, organised by domain (e.g. `results/pde/`). Unlike `data/`, this folder IS tracked by git, so avoid leaving stray/superseded files behind when a script's output changes.
- `scripts/` — scripts that run the algorithms and produce **all visualization/plots** with `Plots.jl`.
- `exercises/` — **student exercises** that combine existing algorithms/scripts in a new way (comparisons, sweeps, refinements, …), organised by domain (e.g. `exercises/rom/`) with their own `results/` subfolder; documented in `exercises/<domain>_exercises.md`.
- `tests/` — tests that validate the algorithms.

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dirk-Hartmann/real-time-digital-twins](https://github.com/Dirk-Hartmann/real-time-digital-twins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
