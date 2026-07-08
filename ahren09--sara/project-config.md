---
trigger: always_on
description: These rules apply to all code changes, experiments, training jobs, evaluation jobs, and generated artifacts in this repository.
---

# Repository Engineering Rules

These rules apply to all code changes, experiments, training jobs, evaluation jobs, and generated artifacts in this repository.

## 1. Rule Priority

When making changes, follow this priority order:

1. Correctness and reproducibility.
2. Reuse of existing implementations.
3. LoRA-first model storage and loading.
4. Code readability and simplicity.
5. Computational efficiency.
6. Minimal changes to the repository structure.

Do not sacrifice correctness or reproducibility merely to reduce the number of changed lines.

---

## 2. Repository Layout

All source code must live under `src/`.

All generated artifacts must use the following repository-relative locations:

```text
src/                     # All library code, training code, evaluation code, and utilities
data/                    # Raw, processed, cached, and generated datasets
outputs/
├── logs/                # Training, evaluation, profiling, and job logs
├── checkpoints/         # LoRA adapters and resumable training states
├── visual/              # Figures, plots, qualitative examples, and videos
└── docs/                # Experiment reports, summaries, tables, and notes
```

Required placement:

* Source code: `src/`
* Logs: `outputs/logs/`
* LoRA checkpoints: `outputs/checkpoints/`
* Visualizations: `outputs/visual/`
* Data and caches: `data/`
* Generated reports and documentation: `outputs/docs/`

Do not save generated files in the repository root or scatter output directories across source-code folders.

Before adding a new output path, verify that one of the directories above cannot be used.

---

## 3. Modify Existing Code Before Adding New Code

Before implementing a feature:

1. Search the repository for related implementations, utilities, configuration fields, and command-line arguments.
2. Extend or generalize the existing implementation whenever practical.
3. Update all relevant callers instead of introducing a parallel implementation.
4. Create a new file only when the functionality is genuinely distinct and cannot be placed naturally in an existing module under `src/`.

Never create files such as:

```text
train_new.py
train_v2.py
train_fixed.py
utils_new.py
evaluation_updated.py
```

unless there is a clear architectural reason.

Do not copy and slightly modify an existing function. If two code paths share substantial logic, extract the shared logic into one reusable implementation.

Before creating a new function, class, CLI option, configuration field, or file, search for an existing equivalent.

---

## 4. Avoid Duplicate Logic

There must be a single source of truth for:

* Dataset loading and preprocessing
* Prompt construction
* Diffusion schedules
* Noise or masking policies
* Model loading
* LoRA configuration
* Generation and sampling
* Metric computation
* Checkpoint saving
* Logging
* Random seed handling
* Device and dtype selection

Do not independently reimplement these behaviors in multiple training or evaluation scripts.

When the same nontrivial logic appears more than once, consolidate it into a shared function or module and update all call sites.

Do not duplicate constants or default hyperparameters across files. Store shared defaults in an existing configuration object, argument parser, or centralized configuration module.

---

## 5. Code Style and Readability

Prefer direct, readable, and explicit code.

* Keep control flow easy to follow.
* Use descriptive variable and function names.
* Avoid unnecessary classes, wrappers, factories, and abstraction layers.
* Avoid deeply nested functions and excessive indirection.
* Do not split a simple operation across many tiny helper functions.
* Extract a helper when it removes meaningful duplication, isolates a coherent operation, or makes the main pipeline substantially easier to read.
* Add comments for non-obvious reasoning, tensor shapes, invariants, and algorithmic decisions—not for code that is already self-explanatory.
* Include tensor shapes in comments or assertions at important boundaries.
* Remove dead code, obsolete branches, stale arguments, and unused imports introduced or exposed by the change.

Prefer extending an existing function over creating a nearly identical alternative. However, do not turn one function into a large collection of unrelated conditional branches merely to avoid creating a new function.

---

## 6. Minimal and Focused Changes

Keep each implementation focused on the requested task.

Do not perform unrelated refactors, rename unrelated files, or reformat the entire repository while implementing a feature.

When changing an interface:

* Update all call sites.
* Update relevant tests and documentation.
* Remove the obsolete interface when backward compatibility is not required.
* Do not silently maintain two equivalent interfaces.

Do not leave temporary debugging scripts, copied code, scratch files, or one-off experiment files in the repository.

All reusable code should be placed under `src/`.

---

## 7. Model Storage and LoRA

All trainable model updates must use LoRA or another explicitly approved parameter-efficient adapter method.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ahren09/SARA](https://github.com/Ahren09/SARA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
