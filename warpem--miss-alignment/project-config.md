---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

miss-alignment is a deep learning package for tilt-series alignment in cryo-electron tomography. It uses a contrastive learning approach with PyTorch and PyTorch Lightning to iteratively train 3D convolutional neural networks that optimize tilt-series alignment by minimizing shift artifacts in reconstructions.

**Key concept**: The system alternates between (1) training a model to score reconstruction quality and (2) using that model (with frozen weights) to optimize tilt-series alignment parameters through gradient descent with the reconstruction quality as a target function.

### Scientific publication

This software is a research project with a preprint publication on biorxiv. People are actively using it and it is supposed to be a software solution, so we need to carefully consider breaking changes.

## Environment Setup

This project has specific CUDA and PyTorch dependencies. Use the conda environment setup from README.md:

```bash
conda create -n miss-alignment -c conda-forge python=3.11 cuda-toolkit=12.9 -y
conda activate miss-alignment
```

Install with: `python -m pip install -e .[dev,test]`

## Common Commands

### Testing
```bash
# Run all tests with coverage
pytest --color=yes --cov --cov-report=xml --cov-report=term-missing

# Run tests in a specific directory
pytest tests/alignment/

# Run a single test file
pytest tests/data/test_training_dataset.py

# Run a specific test function
pytest tests/test_miss_alignment.py::test_something
```

### Linting
```bash
# Run ruff linter with auto-fix
ruff check --fix

# Run ruff formatter
ruff format

# Run pre-commit hooks manually
pre-commit run --all-files
```

### CLI Usage
The package provides a `miss-alignment` CLI command:
```bash
# Train a model (primary workflow)
miss-alignment --config-file config.yaml
```

### Development
```bash
# Build the package
python -m build

# Install in editable mode with dependencies
python -m pip install -e .[dev,test]
```

## Architecture

### Core Modules

1. **`models/`** - Neural network architectures
   - `MissAlignment`: Main PyTorch Lightning module that wraps the 3D CNN
   - Model variants: `Compact3DConvNet` (default), `Compact3DConvNetGELU`, `Compact3DConvNetWide`, `Compact3DConvNetDeep`, `CompactResNet3D`
   - Uses `TripletMarginRankingLoss` for contrastive learning (comparing aligned vs. misaligned reconstructions)

2. **`data/`** - Data loading and synthetic shift generation
   - `MissAlignmentDataModule`: PyTorch Lightning data module managing the reconstruction pool
   - `ReconstructionPoolDataset`: Consumes from a pool of pre-computed 3D patches
   - `shift_generation.py`: Creates synthetic alignment errors (trajectories, jitter, outliers, fractures) for training data
   - `_reconstruction_worker.py`: Multiprocessing workers that generate 3D reconstruction patches in parallel
   - **Architecture note**: Uses a producer-consumer pattern where reconstruction workers populate a temporary pool directory that the dataloader consumes from

3. **`alignment/`** - Tilt-series alignment optimization
   - `tilt_series.py`: Core optimization logic using gradient descent on shift parameters
   - `parallel.py`: Distributes alignment across multiple GPU devices
   - `correlation.py`: Traditional correlation-based alignment methods
   - **Key functions**: `optimize_shifts()` supports three modes:
     - `"global"`: Single shift per image
     - `(3, 3, 41)`: 2D warping field per image
     - `(3, 3, 2, 10)`: 3D volume warp grid

4. **`train.py`** - Iterative training loop
   - Alternates between model training and tilt-series alignment
   - Each iteration: train model → align tilt-series → use aligned data for next iteration
   - Configured via YAML file (see `config_template.yaml`)

### Important Dependencies on External Libraries

- **`warpylib`**: Provides `TiltSeries` and `CubicGrid` for tilt-series geometry and warping
- **`torch-fourier-*`**: Suite of PyTorch-based Fourier transform utilities (rescale, slice, shift, filter)
- **`torch-tiltxcorr`**: Cross-correlation utilities for tilt-series

### Configuration System

Training is configured via YAML files (template: `src/miss_alignment/config_template.yaml`):
- **`general`**: Training directory, CTF settings, iteration-specific parameters (downsample, alignment mode)
- **`model_training`**: Architecture selection, learning rate, loss margin, weight decay, scheduler
- **`data_loading`**: Batch size, patch size, steps per epoch
- **`shift_generation`**: Probabilities and magnitudes for synthetic shifts (trajectories, jitter, outliers, fractures)
- **`tilt_series_alignment`**: Patch size, overlap, batch size for alignment

### Data Flow

1. **Training**: Tilt-series XML files (`.xml`) → Reconstruction workers → Pool directory (`.pickle` patches triplet) → DataLoader → Model
2. **Alignment**: Trained model checkpoint → Load tilt-series data → Gradient-based optimization → Output aligned parameters to XML files (`.xml`)

## Working with Tilt-Series Data


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [warpem/miss-alignment](https://github.com/warpem/miss-alignment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
