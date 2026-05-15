---
trigger: always_on
description: These instructions apply to the whole repository.
---

# Repository Instructions

These instructions apply to the whole repository.

## Working Style

- Be direct and specific.
- Read the local code before changing it. Match the patterns already used by the
  target paper when those patterns do not conflict with this file.
- When a test fails, determine the root cause before changing code.
- Do not hide failures with broad default fallbacks during development. If a value,
  dependency, file, or config key is required, let the code fail clearly. Add a
  `# TODO` only when the missing behavior is intentional and should be handled
  later.
- Use clear, descriptive variable and function names. Prefer self-explanatory
  code over comments that restate each line.
- Keep changes scoped to the paper or shared module being refactored.
- Do not rewrite generated artifacts, notebooks, cached files, or unrelated
  results unless the task explicitly asks for it.

## Repository Model

This repository is a catalogue of reproduced quantum machine learning papers.
Each runnable reproduction lives under `papers/` and is executed by the
repository-level `implementation.py`.

The shared runtime discovers a paper project by these files:

```text
papers/<paper_name>/
|-- configs/defaults.json
|-- cli.json
`-- lib/runner.py
```

`lib/runner.py` must expose the project entry point:

```python
train_and_evaluate(cfg, run_dir)
```

`cfg` is the resolved configuration after defaults, config overlays, global CLI
flags, and paper-specific CLI flags have been applied. `run_dir` is the
timestamped output directory created by the shared runtime.

Some older paper folders currently keep `cli.json` under `configs/cli.json`.
Treat that as a legacy layout. When refactoring one of those papers, migrate the
CLI schema to paper-root `cli.json` so `implementation.py --list-papers` and
`implementation.py --paper <name>` can discover it.

Nested paper suites are allowed. For example:

```text
papers/fock_state_expressivity/<subproject>/
```

Each runnable subproject must still satisfy the same project markers:
`configs/defaults.json`, paper-root `cli.json`, and `lib/runner.py`.

Do not add alternative runtime entry systems such as `runtime.json` or
`runtime_entry.py`. Use the shared runtime.

## Paper Folder Structure

New and refactored papers should follow this structure:

```text
papers/<paper_name>/
|-- README.md
|-- requirements.txt
|-- notebook.ipynb
|-- cli.json
|-- configs/
|   |-- defaults.json
|   `-- <experiment>.json
|-- lib/
|   |-- __init__.py
|   |-- runner.py
|   `-- <paper modules>.py
|-- tests/
|   `-- test_<behavior>.py
|-- utils/
|   `-- <analysis or launch helpers>.py
|-- models/
|-- outdir/
`-- assets/ or images/ or figures/
```

Directory roles:

- `README.md`: paper explanation, reproduction scope, run instructions, and
  obtained results.
- `requirements.txt`: dependencies needed by this paper. Keep paper-specific
  dependencies here instead of assuming another paper's environment.
- `notebook.ipynb`: interactive exploration or reproduction notebook when useful
  for the paper.
- `configs/defaults.json`: default runnable config.
- `configs/<experiment>.json`: named experiments, ablations, or paper settings.
- `cli.json`: paper-specific CLI schema. Global options like `--config`,
  `--outdir`, `--seed`, `--dtype`, `--device`, and `--log-level` are injected by
  the shared runtime.
- `lib/`: importable implementation used by both the runner and notebooks.
- `lib/runner.py`: runtime entry point. Keep orchestration here; keep model,
  data, plotting, and training details in separate modules when they grow.
- `tests/`: smoke tests, config tests, import tests, and focused behavior tests.
- `utils/`: optional command-line helpers for plotting, aggregation, sweeps, or
  batch launching. Utilities should call into `lib/` instead of duplicating core
  logic.
- `models/`: trained model artifacts that are intentionally kept. Use `.gitkeep`
  if the directory should exist without committed models.
- `outdir/`: raw generated run directories. Treat this as disposable runtime
  output and avoid committing it unless a task explicitly curates a small file.
- `assets/`, `images/`, or `figures/`: static README images, paper diagrams, or
  curated visual material. Prefer one of these names per paper; keep references
  in README relative and valid.

Data does not belong inside each paper by default. Use the repository-level data
root:

```text
data/<paper_name>/
```

Reusable dataset or helper code belongs under:

```text
papers/shared/<topic_or_paper>/
```

Paper-local modules should import shared helpers through thin wrappers in
`papers/<paper_name>/lib/` when that keeps the paper API stable.

## Config and CLI Rules

- Configs are JSON-only.
- `configs/defaults.json` must be runnable or intentionally lightweight.
- Named configs should overlay `defaults.json` and represent clear experiments:
  dataset variants, model variants, ablations, smoke runs, or paper-accurate
  runs.
- Do not leave placeholder values such as `<<PATH>>` in runnable configs.
- Do not silently invent values when required config keys are missing. Validate
  the key and fail with a clear error.
- Keep CLI flags in `cli.json`; do not hard-code argument parsing in a paper
  runner unless there is no shared-runtime path for it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [merlinquantum/reproduced_papers](https://github.com/merlinquantum/reproduced_papers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
