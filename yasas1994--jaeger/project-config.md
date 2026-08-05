---
trigger: always_on
description: This file is written for AI coding agents working on **Jaeger** (`jaeger-bio`). It assumes no prior knowledge of the project and only describes what is actually present in the repository.
---

# AGENTS.md — Jaeger Developer Guide

This file is written for AI coding agents working on **Jaeger** (`jaeger-bio`). It assumes no prior knowledge of the project and only describes what is actually present in the repository.

---

## Project Overview

Jaeger is a command-line bioinformatics tool that uses homology-free deep learning to identify bacteriophage and prophage sequences in metagenomic assemblies. It is distributed as the Python package `jaeger-bio` on PyPI and Bioconda.

- **Package name on PyPI / Bioconda:** `jaeger-bio`
- **CLI entry point:** `jaeger`
- **Current version:** `1.26.4` (defined in `pyproject.toml`, `.cz.toml`, `recipes/jaeger-bio/meta.yaml`, and the release tooling)
- **License:** MIT
- **Python support:** `>=3.11, <3.14`
- **Repository:** https://github.com/Yasas1994/Jaeger
- **Documentation:** https://jaeger.readthedocs.io/

The CLI is implemented with `click` and exposes subcommands such as `predict`, `train`, `download`, `health`, `register-models`, and nested groups `utils` and `taxonomy`.

---

## Technology Stack

- **Language:** Python 3.11–3.13
- **Build backend:** `pdm-backend` (configured in `pyproject.toml`)
- **Package layout:** `src/jaeger/`
- **Deep-learning framework:** TensorFlow 2.21–2.22 + Keras 3.12+
- **CLI framework:** `click`
- **Progress / logging:** `rich` progress bars, standard `logging`
- **Sequence I/O:** `pyfastx`, `pydustmasker`
- **Numerical / scientific:** `numpy`, `scipy`, `pandas`, `polars`, `scikit-learn`, `h5py`
- **Alignment:** `parasail` (pinned to `==1.3.4`)
- **Visualization:** `matplotlib`, `seaborn`, `pycirclize`
- **Change-point / stats:** `ruptures`, `kneed`
- **Configuration templating:** `jinja2` + `pyyaml`
- **Documentation:** Sphinx with `myst-parser` and the `furo` theme
- **Versioning / releases:** Commitizen (`cz`) with conventional commits
- **Linting / formatting:** `ruff` (no configuration in `pyproject.toml`; uses defaults)

Platform-specific TensorFlow extras are declared in `pyproject.toml`:

- `jaeger-bio[cpu]` — CPU-only TensorFlow
- `jaeger-bio[gpu]` — TensorFlow with CUDA
- `jaeger-bio[darwin-arm]` — TensorFlow + `tensorflow-metal` for Apple Silicon
- `jaeger-bio[onnx]` — ONNX Runtime, `tf2onnx`, and dependencies for ONNX inference
- `jaeger-bio[test]` — `pytest`, `pytest-mock`
- `jaeger-bio[taxonomy]` — `taxopy`, `faiss-cpu`

---

## Repository Layout

```
.
├── src/jaeger/                 # Main Python package
│   ├── cli.py                  # Click CLI definition and entry point
│   ├── commands/               # Implementation of CLI subcommands
│   │   ├── predict.py          # Modern SavedModel inference pipeline
│   │   ├── predict_legacy.py   # Legacy .h5 / pickled-model pipeline
│   │   ├── train.py            # Model training
│   │   ├── tune.py             # Fine-tuning
│   │   ├── health.py           # Installation health checks
│   │   ├── downloads.py        # Model download from CKAN
│   │   ├── utils.py            # Utility commands (mask, convert, stats, ...)
│   │   ├── utils_models.py     # Model combination / ensemble helpers
│   │   ├── quantize.py         # TFLite quantization
│   │   ├── convert_graph.py    # XLA / TFLite / ONNX / TensorRT conversion
│   │   ├── taxonomy.py         # Experimental taxonomy pipeline
│   │   └── configs/            # Default bundled config
│   ├── nnlib/                  # Neural-network library
│   │   ├── builder.py          # DynamicModelBuilder from YAML configs
│   │   ├── inference.py        # Inference builders and engines
│   │   ├── conversion.py       # Graph-format conversions
│   │   ├── metrics.py          # Per-class Keras metrics
│   │   ├── v1/                 # Legacy model layers
│   │   └── v2/                 # Current model layers, losses, maps
│   ├── seqops/                 # Sequence operations
│   │   ├── io.py               # FASTA reading, fragment generation
│   │   ├── encode.py           # One-hot / codon / frame encoding
│   │   ├── maps.py             # Codon / amino-acid lookup tables
│   │   ├── transform.py        # Sequence transformations
│   │   ├── stats.py            # Sequence statistics
│   │   ├── synthetic.py        # Synthetic sequence generation
│   │   └── validate.py         # Sequence validation
│   ├── dataops/                # Dataset operations
│   │   ├── dataset.py          # Non-redundant fragment database creation
│   │   ├── convert.py          # Format conversions
│   │   ├── split.py            # Train/val/test splitting
│   │   └── ood.py              # Out-of-distribution / shuffle data
│   ├── preprocess/             # Training preprocessing pipelines
│   │   ├── v1/                 # Legacy preprocessing
│   │   └── v2/                 # Current preprocessing
│   ├── postprocess/            # Prediction aggregation
│   │   ├── collect.py          # Collect predictions into tables
│   │   ├── helpers.py          # Helper transforms
│   │   └── prophages.py        # Prophage extraction
│   ├── data/                   # Bundled data
│   │   ├── config.json         # Default model registry
│   │   ├── models/             # Bundled models (default, experimental, test)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yasas1994/Jaeger](https://github.com/Yasas1994/Jaeger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
