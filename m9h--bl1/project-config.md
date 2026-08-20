---
trigger: always_on
description: BL-1 is a JAX-based in-silico cortical culture simulator. It models dissociated cortical neurons on multi-electrode arrays (MEAs) with biologically detailed spiking neurons, conductance-based synapses, four timescales of plasticity, and closed-loop game experiments. The entire simulation loop is JIT-compiled via `jax.lax.scan` and differentiable through surrogate gradients.
---

# BL-1 Development Guide

## Project Overview

BL-1 is a JAX-based in-silico cortical culture simulator. It models dissociated cortical neurons on multi-electrode arrays (MEAs) with biologically detailed spiking neurons, conductance-based synapses, four timescales of plasticity, and closed-loop game experiments. The entire simulation loop is JIT-compiled via `jax.lax.scan` and differentiable through surrogate gradients.

## Environment

- **Platform**: DGX Spark (aarch64), NVIDIA GB10 GPU, CUDA 13, JAX 0.9.2
- **Python**: 3.12, venv at `.venv/`
- **NAS**: TrueNAS at `/data` (NFS mount, 41 TB, shared with fedora-legion)
- **Install**: `pip install -e ".[dev]"` then `pip install trackio pynwb dandi optax`

## Commands

```bash
# Tests (must pass before any commit)
make test                              # 536 tests, ~4 min
.venv/bin/pytest tests/test_validation.py -v  # validation framework

# Full validation suite
bash scripts/run_validation.sh --quick  # tests + benchmarks + bio-validation

# Training
python scripts/train_culture.py --from-recording FILE.nwb --n-neurons 5000 --n-epochs 100
python scripts/train_all_sharf.py      # batch training, all 33 recordings

# Dataset analysis
python scripts/analyze_all_datasets.py  # stats for all downloaded data
python scripts/validate_real_data.py    # real vs simulated comparison
```

## Architecture

```
src/bl1/
  core/          # Izhikevich/AdEx neurons, AMPA/NMDA/GABA synapses, integrator
  plasticity/    # STP, STDP, homeostatic, structural
  network/       # Topology, connectivity, Culture factory
  mea/           # Virtual MEA (64-ch, HD-MEA 26K electrodes)
  training/      # Differentiable training loop (trainer.py, loss.py)
  validation/    # Dataset catalog, loaders (NWB/HDF5), comparison framework
  analysis/      # Bursts, criticality, connectivity, information theory
  visualization/ # Raster plots, rates, MEA heatmaps
  games/         # Pong, Doom closed-loop environments
```

## Data on NAS (`/data`)

All large files live on the NAS, visible from both DGX Spark and fedora-legion:

```
/data/datasets/bl1/
  dandi_001611_rat_cortical/   # 2,700 NWB files, 20 GB — rat cortical HD-MEA
  zenodo_sharf_2022/           # 33 HDF5 files, 67 GB — human brain organoid
  osf_dishbrain/               # DishBrain spike data
  results/
    sharf_2022/                # Training results (organized by condition)
      baseline/                #   10 recordings
      development/             #   4 recordings
      drug_dose_response/      #   19 recordings
      trackio/                 #   Experiment tracking logs
      summary_*.csv            #   Spreadsheet-ready results
    dataset_analysis/          # Cross-dataset statistics (JSON)
```

## Validated Parameters (Wagenaar 2006)

These are the calibrated simulation parameters that pass 6/6 bio-validation metrics. Do not change without re-running validation:

- `n_neurons=5000`, `p_max=0.21`, `g_exc=0.12`, `g_inh=0.36`
- AMPA/NMDA split: `nmda_ratio=0.37`
- STP: `U_exc=0.30`, `tau_rec=800ms`
- Burst detection: `threshold_std=1.5`
- Duration: 60s for robust IBI statistics

Config file: `configs/wagenaar_calibrated.yaml`

## Training Pipeline

### How it works

1. Load real recording (NWB or Maxwell HDF5) via `bl1.validation.loaders`
2. Extract targets: firing rate and burst rate from the activity window
3. Build network with `build_connectivity` (sparse BCOO), convert to dense
4. Scale weights down by `init_weight_scale=0.1` (training runs without STP)
5. Forward pass: `simulate(..., surrogate=True)` through `jax.lax.scan`
6. Loss: log-scale FR + differentiable burst proxy + synchrony + weight reg
7. Backward pass: `jax.grad` with SuperSpike surrogate (beta=5.0)
8. Update via Adam + gradient clipping + NaN protection + weight clamping
9. Log to trackio every epoch

### Key files

| File | Purpose |
|------|---------|
| `src/bl1/training/trainer.py` | Core training loop, `train_weights()` |
| `src/bl1/training/loss.py` | Loss function components |
| `src/bl1/training/sharding.py` | Multi-GPU neuron-parallel sharding primitives |
| `scripts/train_culture.py` | CLI entry point, `--from-recording` |
| `scripts/train_all_sharf.py` | Batch training with trackio |
| `configs/wagenaar_calibrated.yaml` | Validated simulation parameters |
| `configs/wagenaar_burst.yaml` | Burst-rate training contract |

### Current state of the training-side knobs

- **FR floor (solved).** Auto-noise calibration (`TrainingConfig.auto_noise=True`)
  scales `I_noise_amplitude` and `init_weight_scale` from the target firing rate
  in two regimes (low <1 Hz: `target*5+0.3`, ws=0.05; high ≥1 Hz: `target*1+0.3`,
  ws=0.50).  Final convergence: 93% of target at 0.3 Hz.

- **Burst-rate matching.** `TrainingConfig` already defaults to
  `sim_duration_ms=5000`, `w_burst_rate=0.5`, `use_stp=True`,
  `target_burst_rate_per_min=8.0` (Wagenaar).  The contract lives in
  `configs/wagenaar_burst.yaml`.  Smoke-tested locally

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m9h/bl1](https://github.com/m9h/bl1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
