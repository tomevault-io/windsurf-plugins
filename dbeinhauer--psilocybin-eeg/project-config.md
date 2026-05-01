---
trigger: always_on
description: This project analyzes EEG neural data from participants recorded during listening to two types of music (classical and psytrance) under two conditions (placebo and psilocybin treatment). The major goal is to find and analyze the **inter-subject synchrony (ISC)** between participants.
---

# Psilocybin-EEG Project - Agent Instructions

## Project Context

This project analyzes EEG neural data from participants recorded during listening to two types of music (classical and psytrance) under two conditions (placebo and psilocybin treatment). The major goal is to find and analyze the **inter-subject synchrony (ISC)** between participants.

### Experimental Design
- **EEG System**: EGI GSN-HydroCel 257-channel montage
- **Conditions**: Placebo / Psilocybin (within-subject, counterbalanced A/B)
- **Music Types**: Classical (`CLASSIC`), Psytrance (`PSYTRANCE`)
- **Primary Analysis**: Inter-subject correlation (ISC) per frequency band
- **File Formats**: EDF (raw), MNE `.fif` (processed), NumPy `.npy` (arrays), CSV (metadata)

## Technology Stack
- **Python**: ≥3.12
- **Core Libraries**: MNE-Python (EEG processing), NumPy, SciPy, Pandas
- **ICA/Artifact Removal**: AutoReject, MNE-ICALabel, PyTorch (optional GPU)
- **Visualization**: Matplotlib, Seaborn
- **Testing**: pytest
- **Linting**: Ruff (via GitHub Actions)

## Pipeline Architecture

The analysis pipeline has three main stages:

### 1. Preprocessing (`src/preprocessing/`, `scripts/run_preprocessing.py`)
The pipeline order is: **channel_prep → filtering → ica → time_alignment**

For the full step-by-step details of each preprocessing stage, see [`docs/preprocessing_steps.md`](../docs/preprocessing_steps.md).

### 2. Time Alignment (`src/preprocessing/time_alignment.py`, `scripts/run_time_alignment.py`)
- Extract TAG channel stimulus markers
- Compute pairwise cross-correlation
- Select reference recording
- Shift and crop all recordings to a common time window

### 3. Analysis (`src/analysis/`, `scripts/run_analysis.py`)

The analysis workflow follows three principles:
1. **Exploration first** — each analysis must be sketched in a Jupyter notebook before production implementation.
2. **Statistical plots in Seaborn** — use Seaborn for all statistical visualizations where possible; fall back to Matplotlib only for EEG topomaps and other domain-specific plots.
3. **Reproducible production pipeline** — every analysis must have a structured CLI script (`scripts/`) and a corresponding HPC job template (`jobs/metacentrum/`) so it can be run on the cluster for large-scale processing.

### Notebook Organisation

All exploration notebooks live under `notebooks/` in numbered subdirectories:

```
notebooks/
├── 00-preprocessing/                # Preprocessing inspection and time alignment
│   ├── eeg_data_inspection.ipynb   # Inspect raw EEG at different pipeline stages
│   └── time_alignment.ipynb        # Time alignment, cross-correlation, TAG overlay
├── 01-raw-mean-variance-analysis/   # Mean-variance synchrony analysis
│   ├── mean_variance_broadband.ipynb   # Part 1 — broadband (z-scored raw)
│   └── mean_variance_bands.ipynb       # Part 2 — per-frequency-band
├── 02-<analysis-name>/              # Next analysis (use next available 2-digit ID)
│   └── ...
└── ...                              # Legacy / unorganised notebooks (do not move)
```

**Naming rules:**
- Subdirectory: `NN-<kebab-case-analysis-name>` where `NN` is a two-digit zero-padded integer. Use `00` for preprocessing notebooks (e.g. `00-preprocessing`) and start analysis notebooks at `01` (e.g. `01-raw-mean-variance-analysis`, `02-isc-analysis`).
- Notebook files: descriptive `snake_case` names that reflect the scope (e.g. `mean_variance_broadband.ipynb`, `mean_variance_bands.ipynb`).
- When a single analysis covers multiple distinct sub-scopes (e.g. broadband vs per-band), **split into separate notebooks** — one notebook per logical sub-scope.

**Notebook structure (each notebook must follow this template):**
1. **Setup cell** (code) — **must be the very first cell**. Contains the project-root resolver (`_p`) and **all** imports from `sys`, `os`, `pathlib`, third-party libraries (`numpy`, `matplotlib`, `seaborn`, `pandas`), `src.*`, and `scripts.*`. Imports that follow non-import code (e.g. the `sys.path.insert` resolver loop) must carry `# noqa: E402` so the CI linter passes. Only import what is needed for this notebook's scope. **Never import inside analysis cells.**
2. **Title cell** (Markdown) — analysis name + scope + brief description of what is computed and visualised.
3. **Configuration cell** (code) — all user-tunable parameters (condition, music types, window sizes, thresholds …) in one place; also define `SAVE_PLOTS = True` and `PLOTS_DIR = ProjectPaths.NOTEBOOKS_DIR / "<notebook-dir>" / "plots" / "<scope>"` (import `ProjectPaths` in the setup cell).
4. **Data loading cell** (code) — load / process-and-save data via `load_analyzers` / `analyzers_to_datasets`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbeinhauer/psilocybin-eeg](https://github.com/dbeinhauer/psilocybin-eeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
