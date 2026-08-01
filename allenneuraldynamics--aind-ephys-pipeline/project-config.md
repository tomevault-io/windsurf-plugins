---
trigger: always_on
description: This project is an **Electrophysiology Analysis Pipeline** developed by the Allen Institute for Neural Dynamics (AIND). It processes extracellular electrophysiology recordings using [SpikeInterface](https://github.com/SpikeInterface/spikeinterface) and orchestrates multiple processing steps through [Nextflow](https://www.nextflow.io/).
---

# AIND Ephys Pipeline

## Project Overview

This project is an **Electrophysiology Analysis Pipeline** developed by the Allen Institute for Neural Dynamics (AIND). It processes extracellular electrophysiology recordings using [SpikeInterface](https://github.com/SpikeInterface/spikeinterface) and orchestrates multiple processing steps through [Nextflow](https://www.nextflow.io/).

## Architecture

### Dual Deployment Model

The pipeline supports two deployment environments with different Nextflow implementations:

1. **Code Ocean Deployment** (`pipeline/main.nf`)
   - Uses Nextflow DSL1
   - Deployed on AIND's Code Ocean platform
   - Clones code from Code Ocean private Git repositories
   - Pre-configured branches for different spike sorters
   - Uses Code Ocean container registry

2. **SLURM/Local Deployment** (`pipeline/main_multi_backend.nf`)
   - Uses Nextflow DSL2 (modern syntax)
   - Supports both SLURM clusters and local execution
   - Clones code from public GitHub repositories
   - Dynamic sorter selection via parameters
   - Uses GitHub Container Registry (ghcr.io)

### Key Difference

Both pipelines execute the **same processing steps**, but:
- `main.nf` is Code Ocean-specific with hardcoded capsule IDs
- `main_multi_backend.nf` is flexible, supporting multiple executors and configurable via parameters

## Pipeline Steps

The pipeline consists of 11 processing steps executed sequentially (with parallelization where applicable):

### 1. Job Dispatch
- **Purpose**: Generate JSON job files for parallel processing
- **Repo**: [aind-ephys-job-dispatch](https://github.com/AllenNeuralDynamics/aind-ephys-job-dispatch)
- **Parallelization**: Creates jobs per probe and per shank (NP2-4shank probes)
- **Container**: `aind-ephys-pipeline-base`

### 2. Preprocessing
- **Purpose**: Data preparation and cleaning
- **Repo**: [aind-ephys-preprocessing](https://github.com/AllenNeuralDynamics/aind-ephys-preprocessing)
- **Operations**: Phase shift, highpass filter, denoising (bad channel removal + CMR/destripe), motion estimation/correction
- **Container**: `aind-ephys-pipeline-base`

### 3. Spike Sorting
Three sorter options (selected via parameter or branch):
- **Kilosort2.5**: [aind-ephys-spikesort-kilosort25](https://github.com/AllenNeuralDynamics/aind-ephys-spikesort-kilosort25)
  - Container: `aind-ephys-spikesort-kilosort25`
- **Kilosort4**: [aind-ephys-spikesort-kilosort4](https://github.com/AllenNeuralDynamics/aind-ephys-spikesort-kilosort4)
  - Container: `aind-ephys-spikesort-kilosort4`
  - Requires GPU
- **SpykingCircus2**: [aind-ephys-spikesort-spykingcircus2](https://github.com/AllenNeuralDynamics/aind-ephys-spikesort-spykingcircus2)
  - Container: `aind-ephys-pipeline-base`

### 4. Postprocessing
- **Purpose**: Compute spike/unit metrics and extensions
- **Repo**: [aind-ephys-postprocessing](https://github.com/AllenNeuralDynamics/aind-ephys-postprocessing)
- **Operations**: Remove duplicates, compute amplitudes, locations, PCA, correlograms, template similarity, quality metrics
- **Container**: `aind-ephys-pipeline-base`

### 5. Curation
- **Purpose**: Quality control and unit classification
- **Repo**: [aind-ephys-curation](https://github.com/AllenNeuralDynamics/aind-ephys-curation)
- **Methods**: Threshold-based filtering (ISI violation, presence ratio, amplitude cutoff), UnitRefine classifier
- **ML Models**: Uses pretrained models from Hugging Face:
  - [UnitRefine_noise_neural_classifier](https://huggingface.co/SpikeInterface/UnitRefine_noise_neural_classifier): Classifies units as noise vs neural
  - [UnitRefine_sua_mua_classifier](https://huggingface.co/SpikeInterface/UnitRefine_sua_mua_classifier): Classifies neural units as single-unit (SUA) vs multi-unit (MUA)
- **Container**: `aind-ephys-pipeline-base`

### 6. Visualization
- **Purpose**: Generate plots and interactive Figurl visualizations
- **Repo**: [aind-ephys-visualization](https://github.com/AllenNeuralDynamics/aind-ephys-visualization)
- **Outputs**: Timeseries plots, drift maps, motion visualizations, sorting summary
- **Container**: `aind-ephys-pipeline-base`

### 7. Results Collector
- **Purpose**: Aggregate parallel job outputs
- **Repo**: [aind-ephys-result-collector](https://github.com/AllenNeuralDynamics/aind-ephys-result-collector)
- **Operation**: Copy all results to standardized output structure
- **Container**: `aind-ephys-pipeline-base`

### 8. Quality Control
- **Purpose**: Run quality control checks on results
- **Repo**: [aind-ephys-processing-qc](https://github.com/AllenNeuralDynamics/aind-ephys-processing-qc)
- **Container**: `aind-ephys-pipeline-base`

### 9. Quality Control Collector
- **Purpose**: Aggregate QC results
- **Repo**: [aind-ephys-qc-collector](https://github.com/AllenNeuralDynamics/aind-ephys-qc-collector)
- **Container**: `aind-ephys-pipeline-base`

### 10. NWB Subject (in multi_backend only)
- **Purpose**: Create NWB subject metadata

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AllenNeuralDynamics/aind-ephys-pipeline](https://github.com/AllenNeuralDynamics/aind-ephys-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
