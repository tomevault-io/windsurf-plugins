---
trigger: always_on
description: This file brings an agent up to speed on Yoke's purpose, structure, conventions, and
---

# AGENTS.md — Orientation for AI Agents Working on Yoke

This file brings an agent up to speed on Yoke's purpose, structure, conventions, and
workflows. Read it fully before making changes.

---

## 1. What Yoke is

**YOKE** (Yielding Optimal Knowledge Enhancement) is a PyTorch prototyping, training, and
testing harness developed at LANL for multi-physics / multi-material ML applications
(projects: ArtIMis, ASC-PEM-EADA).

**Core philosophy:** define models, datasets, losses, metrics, and training/eval routines as
*modular, reusable, installable* Python components under `src/yoke`, then *codify their use*
as **harnesses** under `applications/harnesses`. A harness pins together a training script,
its hyperparameters, and its job-submission configuration so that a training/eval study is
**reproducible, trackable, and easy to vary**.

Data is **not** stored in the repo. Data locations are passed as command-line arguments to
the training/eval scripts.

---

## 2. Repository layout

Only **`./src`** is the installable package. Everything else is project material but not an
installable module.

```
src/yoke/                      # THE installable package
  __init__.py
  datasets/                    # torch Dataset classes (lsc_, jwl_, nestedcyl_, diffusion_, load_npz_, transforms)
  models/                      # torch models (CNNmodules, hybridCNNmodules, policy/surrogate CNNs, mnist_model, vit/)
  losses/                      # masked_loss.py, NormMSE.py
  metrics/                     # shaped_charge_metrics.py
  helpers/                     # cli.py (argparse builders), strings.py (template subst),
                               #   logger.py, training_design.py
  utils/                       # checkpointing.py, dataload.py, parallel.py, parameters.py, ema.py
    training/
      datastep/                # per-batch step fns (loderunner, lsc_policy, lsc_reward, scalar_output, array_output, diff_loderunner)
      epoch/                   # per-epoch train/val fns (same family as datastep/)
    diffusion/
  lr_schedulers.py             # e.g. ConstantWithWarmupScheduler
  scheduled_sampling.py
  cli/                         # console-script entry points
    start_study.py             # `yoke-start-study` launcher
  harnesses/                   # HarnessStudy base class (study workflow + continuation)
    base.py

applications/                  # NOT installable; scripts + study definitions
  harnesses/                   # one subdir per study (see Section 4)
  evaluation/  viewers/  normalization/  filelists/
  makefilelists.py

tests/                         # pytest; mirrors src layout (datasets/, models/, helpers/, utils/, ...)
docs/                          # sphinx (source/, Makefile)
docker/                        # Dockerfile for containerized builds
dev_plans/                     # design/planning docs (markdown)
```

---

## 3. Environment, install, and tooling

- **Installed interpreter:** Yoke is installed in a dedicated Python environment (typically a
  conda env). **Ask the user for the path to the Python in which Yoke is installed** before
  running Yoke, its tests, or its CLI.
- **Build backend:** `flit` (`pyproject.toml`). Python `>=3.11`.
- **Note on PyTorch:** conda uses `pytorch`; pip/flit use `torch`/`torchvision`/`torchaudio`.
  Yoke is primarily used via conda envs, so torch deps are often handled separately.
- **Dev install:** `flit install --symlink --deps develop` (editable + test/dev deps).

### Testing (run from repo root)
```bash
pytest -Werror
pytest --cov --cov-report term-missing
pytest --cov=. --cov-report=html      # HTML report
```
CI (`.github/workflows/yoke_install_test_lint.yml`) runs, essentially:
`pytest -v --cov=yoke -Werror`, then `ruff` checks. **Tests must pass with `-Werror`** (no
warnings), and coverage is tracked via Coveralls.

### Linting / formatting (must pass in CI)
```bash
ruff check
ruff check --preview
ruff format --check --diff
```
Auto-fix with `ruff check --fix` and `ruff format`.

**Ruff config (pyproject.toml):** line-length **89**, 4-space indent, double quotes.
Enabled rule families: `E, F, D, UP, W, ANN`. Docstring convention: **google**. This means:
- **Every public module/class/function needs a docstring** (`D`).
- **Type annotations are required** on functions/args (`ANN`).
- Keep lines <= 89 chars.

CI lints `src` (non-blocking/`continue-on-error` for the main package preview) plus
`applications/evaluation`, `applications/filelists`, `applications/normalization`,
`applications/viewers`, `tests`, and scheduled sampling (blocking). Keep new code clean
everywhere.

---

## 4. How a harness works (the central workflow)

A harness directory (e.g. `applications/harnesses/ch_DDP_loderunner/`) contains:

- **A training/eval script** (e.g. `train_LodeRunner_ddp.py`) — imports from `yoke.*`, builds
  its argparse from `yoke.helpers.cli` builders, and reads args via
  `fromfile_prefix_chars="@"` (i.e. `python train.py @some.input`).
- **`training_input.tmpl`** — argument file template with `<KEY>` placeholders.
- **A submission template** — a single `training_slurm.tmpl` or `training_shell.tmpl` (a
  complete SLURM/shell script) with conditional blocks (`# <<optional:KEY>>` ... `# <<end>>`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lanl/Yoke](https://github.com/lanl/Yoke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
