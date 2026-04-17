---
trigger: always_on
description: By default the Python environment is found in `.venv`.
---

# JAX-ENT: JAX-based Ensemble Modeling and Optimization Toolkit

By default the Python environment is found in `.venv`.

JAX-ENT is a JAX-based library for ensemble modeling and optimization in structural biology/biophysics, with a strong focus on HDX-MS workflows. This document is a codebase-oriented guide for a mixed audience: contributors who need architectural detail and users who need practical workflow grounding.

## What this guide prioritizes

- **Current implementation reality** in `jaxent/src` (the main codebase).
- **Grounded workflows** from `jaxent/examples` (real, end-to-end pipelines).
- **How components connect** across featurization, prediction, splitting, and optimization.

---

## Repository orientation

### Main implementation surface: `jaxent/src`

- `analysis/` - plotting and analysis helpers (for example `analysis/plots/optimisation.py`).
- `config/` - configuration infrastructure.
- `custom_types/` - typed interfaces/data containers (`ForwardModel`, feature bases, datapoints, keys, settings).
- `data/` - data loading, sparse mapping, dataset splitting.
- `interfaces/` - builder, simulation/model interfaces, topology system.
- `models/` - forward models and model-specific feature/parameter logic.
- `opt/` - loss definitions, optimization state/history, optimizer orchestration.
- `utils/` - JAX helpers, JIT guards, HDF persistence, MDAnalysis utilities.
- Top-level execution helpers:
  - `featurise.py` - orchestration for feature generation.
  - `predict.py` - single-parameter prediction pathway.
  - `predict_forward.py` - batched/multi-parameter forward pathway.

### CLI entrypoints: `jaxent/cli`

- `featurise.py` - generate model input features from topology/trajectory.
- `predict.py` - run one prediction set from saved features/topology.
- `forward.py` - run many simulation parameter sets in one pass.
- `efficient_k_cluster.py` - memory-aware PCA + clustering for trajectory reduction.

### Workflow grounding: `jaxent/examples`

Active example suites include:

- `1_IsoValidation_OMass`
- `2_CrossValidation`
- `3_CrossValidationBV`
- `predict_traj`
- shared helpers in `examples/common`

These are extensive and script-heavy; use the context strategy described later when navigating them.

---

## Core architecture and data flow

JAX-ENT is modular, but the modules are connected by a key-driven contract and topology-aware mapping.

### 1) Experimental data layer

- Experimental points inherit from `ExpD_Datapoint` (`jaxent/src/custom_types/datapoint.py`).
- HDX-specific datapoints live in `jaxent/src/custom_types/HDX.py`:
  - `HDX_peptide` (`m_key("HDX_peptide")`)
  - `HDX_protection_factor` (`m_key("HDX_resPf")`)
- `ExpD_Dataloader` (`jaxent/src/data/loader.py`) stores raw datapoints and builds JAX-traceable train/val/test `Dataset` objects.

### 2) Forward model contract and key dispatch

- `ForwardModel` base (`jaxent/src/custom_types/base.py`) defines:
  - `initialise(ensemble)`
  - `featurise(ensemble)`
  - `forward` map keyed by `m_key`
- Model config objects (for example `BV_model_Config`, `NetHDXConfig`) in `jaxent/src/models/config.py` determine whether outputs are residue-level (`HDX_resPF`) or peptide-level (`HDX_peptide`) via `num_timepoints`/key selection.

### 3) Featurisation

- `run_featurise` (`jaxent/src/featurise.py`) orchestrates feature generation from an `Experiment_Builder`.
- `Experiment_Builder` (`jaxent/src/interfaces/builder.py`) links universes, forward models, optional data, and can construct a `Simulation`.
- For HDX BV, `BV_model.featurise` computes heavy/acceptor contacts from MD trajectories and returns `BV_input_features`.

### 4) Topology alignment and sparse mapping

- Topology subsystem (`jaxent/src/interfaces/topology/`) is central for robust fragment/residue alignment:
  - `Partial_Topology` (`core.py`)
  - `TopologyFactory` (`factory.py`)
  - `PairwiseTopologyComparisons` (`pairwise.py`)
  - `mda_TopologyAdapter` (`mda_adapter.py`)
  - `PTSerialiser` (`serialise.py`)
- `create_sparse_map` (`jaxent/src/data/splitting/sparse_map.py`) maps residue-level model outputs to experimental fragments.
- This map is the critical bridge for comparing residue-wise predictions to peptide-wise HDX observations.

### 5) Simulation and prediction

- `Simulation` (`jaxent/src/models/core.py`) is the execution core:
  - validates input/model dimensions
  - normalizes simulation parameters
  - manages JIT/non-JIT forward execution
- `run_predict` (`jaxent/src/predict.py`) executes one parameter set.
- `run_forward` (`jaxent/src/predict_forward.py`) iterates over many `Simulation_Parameters` sets.

### 6) Optimization

- `run_optimise` (`jaxent/src/opt/run.py`) controls the training loop.
- `OptaxOptimizer` (`jaxent/src/opt/optimiser.py`) wraps Optax update logic and masking behavior.
- `OptimizationState` / `OptimizationHistory` (`jaxent/src/opt/base.py`) track parameter/loss trajectories.
- Losses are composed from `jaxent/src/opt/losses.py` and `jaxent/src/opt/loss/`.

### 7) Persistence and analysis

- `jaxent/src/utils/hdf.py` serializes/deserializes optimization history, states, and parameters to HDF5.
- `jaxent/src/analysis/plots/optimisation.py` provides plotting utilities for loss and weight evolution.

---

## Forward model families


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexisiddiqui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
