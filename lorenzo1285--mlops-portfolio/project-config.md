---
trigger: always_on
description: > Agent context for the MLOps Learning Portfolio — VAE-Based Crash Severity Pipeline.
---

# AGENTS.md

> Agent context for the MLOps Learning Portfolio — VAE-Based Crash Severity Pipeline.
> This file complements CLAUDE.md (architecture deep-dive) and tasks.md (execution checklist).

---

## Project Identity

- **Name**: MLOps Learning Portfolio — Crash Severity Use Case
- **Dataset**: CGR Crash Data (Grand Rapids, ~74k rows, 142 cols)
- **Target**: 3-class crash severity — PDO (0), Injury (1), Fatal (2)
- **Constitution**: v3.4.0 | **Architecture**: 10-stage DVC pipeline
- **Macro F1 Gate**: > 0.35 | **Fatal Recall Gate**: > 0.50 (PDO sacrifice accepted)

---

## What Is This Project

A production-grade MLOps reference implementation demonstrating the full toolchain:

- **DVC** — data / model versioning + reproducible pipeline DAG
- **Great Expectations** — data contract validation before any training
- **MLflow** — experiment tracking, model registry, champion alias
- **Optuna** — active hyperparameter search engine (local, continuous space, pruning)
- **Katib** — Kubernetes-native HPO (portfolio reference, retained but not active)
- **Kubeflow Pipelines** — containerised orchestration on Docker Desktop K8s
- **PyTorch + XGBoost** — competing classifiers on VAE-learned latent representations
- **CTGAN/TVAE** — generative augmentation of Fatal-class rows

Pipeline DAG:

```
validate -> ingest -> featurize -> [train_vae || augment] -> encode -> [train_ml || train_dl] -> evaluate -> tune -> register
```

---

## Current State (as of last update)

**Position**: Phase O3.5 (Optuna HPO) + Phase M5 (VAE Fatal Recall Fixes).

**Current metrics**: val recall=52.4% OK, test recall=33.3% FAIL -- 4 more correct Fatal predictions needed on Z_test.

**Next work**: See `specs/002-mlops-portfolio/tasks.md` for the canonical step-by-step execution order. Key open items:

- T129: Wire `OptunaTuner` into `src/tune/run.py` (update `params.yaml` with best params + `dl.input_dim` sync)
- T130: Update `dvc.yaml` tune stage params list
- T131: `uv run dvc repro tune` smoke test
- T132: `uv run dvc repro` full pipeline -- check gates
- Decision: gates pass -> T051 (Register). Gates fail -> M5 fixes in order (Fix A focal loss -> Fix B cyclical KL -> Fix C danger index -> Fix D XGBoost focal -> Fix E supervised latent -> Fix F Tomek links).

---

## Directory Quick Reference

| Path | Purpose |
|------|---------|
| `src/<stage>/` | One package per pipeline stage: `<module>.py` (business-logic class) + `run.py` (thin entry point) |
| `src/config.py` | Typed dataclasses for all `params.yaml` sections; `load_config()` reads `PARAMS_PATH` env var |
| `src/metrics.py` | Shared helpers: `make_eval_dataset`, `per_class_matrix`, `compute_class_weights`, `BalancedFocalLoss` |
| `params.yaml` | Single source of truth for all parameters, column lists, thresholds, HPO search spaces |
| `dvc.yaml` | 10-stage DVC pipeline DAG with deps / outs / params |
| `docs/` | Human-readable artifacts: `data_contract.md`, `evaluation_report.json`, `ab_test_comparison.json` |
| `tests/` | Boundary tests only; real data fixtures; no internal mocking |
| `great_expectations/gx/` | GE v1 file context; suites in `expectations/`; Data Docs in `uncommitted/data_docs/` |
| `k8s/` | Kubernetes manifests: `pvc.yaml`, `katib/vae_experiment.yaml` |
| `pipelines/kubeflow/` | KFP pipeline definition (`pipeline.py`) |
| `docker/` | `Dockerfile` for container-native stage execution |
| `airflow/` | Tutorial DAGs only; not part of active pipeline |
| `.specify/memory/constitution.md` | 18 non-negotiable principles; amendment requires version bump + rationale |
| `CLAUDE.md` | Full architecture reference (GE layer, VAE details, MLflow conventions, skills list) |
| `UBIQUITOUS_LANGUAGE.md` | Canonical domain glossary; must be updated before speckit.plan |

---

## Essential Commands

### Environment
- Use `uv` for all Python operations: `uv add <pkg>`, `uv sync`, `uv run ...`
- Virtualenv is at `.venv/`; Python version pinned in `.python-version`

### Tests (Windows — obey Constitution XVII)
```powershell
uv run python -m pytest tests/ -v
uv run python -m pytest tests/test_optuna_tuner.py -v
```
> Do NOT use `uv run pytest` (console-script form fails on Windows with "Failed to canonicalize script path").

### DVC
```bash
uv run dvc repro              # full pipeline
uv run dvc repro <stage>      # single stage
uv run dvc status             # cache state
uv run dvc push               # sync artifacts to remote
uv run dvc pull               # restore artifacts
```

### MLflow
```bash
uv run mlflow ui              # http://localhost:5000
```

### Katib (portfolio reference)
```bash
kubectl apply -f k8s/katib/vae_experiment.yaml
```

### Airflow (tutorial only)
```powershell
cd airflow
.\setup.ps1                   # one-time
uv run airflow standalone     # http://localhost:8080
```

---

## Development Rules (Constitution Highlights)

These are non-negotiable. Violations block task completion.

1. **TDD for all `src/` code** — red -> green -> refactor; vertical slices only; tests written BEFORE implementation.
2. **Real data fixtures only** — boundary tests must use real pipeline artifacts (e.g. `data/processed/raw.csv`, `Z_train_augmented.npy`). No `np.random.randn` or hardcoded dummy frames as primary fixtures.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lorenzo1285/mlops-portfolio](https://github.com/lorenzo1285/mlops-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
