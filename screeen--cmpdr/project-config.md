---
trigger: always_on
description: This is the cMVDR (Cyclic Minimum-Variance Distortionless-Response) beamformer project - a Python implementation of an advanced beamforming algorithm for audio signal processing. The project extends the classic MVDR beamformer to exploit both spatial and spectral correlations for better suppression of almost-periodic noise (e.g., engines, fans, musical instruments).
---

# GitHub Copilot Instructions for cMVDR

## Project Overview

This is the cMVDR (Cyclic Minimum-Variance Distortionless-Response) beamformer project - a Python implementation of an advanced beamforming algorithm for audio signal processing. The project extends the classic MVDR beamformer to exploit both spatial and spectral correlations for better suppression of almost-periodic noise (e.g., engines, fans, musical instruments).

**Key application areas**: Speech enhancement, hearing aids, smart devices, and acoustic scene analysis.

**Research paper**: https://arxiv.org/abs/2510.18391v1

## Development Environment

### Python Version
- **Required**: Python 3.9+ (for compatibility with librosa)
- **Tested on**: Python 3.11
- **OS tested**: macOS 14.4.1 and Ubuntu 24.04.3 LTS

### Dependencies
- Core dependencies are managed in `pyproject.toml`
- Key libraries: librosa, numpy, scipy, matplotlib, sounddevice, pystoi, pesq, pysepm-evo
- Use pip for package management and installation

### Installation

**Step-by-step installation process:**

1. Clone the repository:
```bash
git clone git@github.com:Screeen/cmvdr.git
cd cmvdr
```

2. Create and activate a Python virtual environment:
```bash
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

3. Install the package in editable mode:
```bash
pip install --upgrade pip
pip install -e .
```

This will automatically install all dependencies defined in `pyproject.toml`.

4. Verify installation:
```bash
python -c 'import cmvdr; print("cMVDR package successfully imported!")'
```

## Project Structure

```
cmvdr/
├── cmvdr/                      # Main package directory
│   ├── beamforming/           # Core beamforming algorithms (MVDR, cyclic MVDR)
│   ├── cli/                   # Command-line interface tools
│   ├── data_gen/              # Synthetic data generation
│   ├── estimation/            # Signal estimation algorithms
│   ├── eval/                  # Evaluation metrics and tools
│   ├── presets/               # Configuration presets
│   └── util/                  # Utility functions
├── tests/                     # Unit and integration tests
├── configs/                   # YAML configuration files
├── script/                    # Helper scripts for running experiments
├── demo/                      # Demo audio files and examples
├── main.py                    # Main experiment runner
└── pyproject.toml            # Package configuration and dependencies
```

## Code Style and Conventions

### General Guidelines
- Follow PEP 8 style guidelines for Python code
- Use type hints where appropriate to improve code clarity
- Write descriptive variable names, especially for signal processing variables
- Keep functions focused and modular

### Signal Processing Conventions
- Use standard variable naming for signal processing:
  - `M`: Number of microphones/channels
  - `K`: Number of frequency bins
  - `P`: Number of cyclic shifts
  - `rtf`: Relative transfer function
  - `mvdr`: Minimum variance distortionless response
- Complex-valued arrays should use `dtype=np.complex128`
- FFT/STFT operations should preserve proper scaling and normalization

### Documentation
- Include docstrings for all public functions and classes
- Document parameters, return values, and any important implementation details
- Reference academic papers or algorithms where applicable

## Testing

### Running Tests
```bash
# Run all tests
python -m unittest discover -s tests

# Run specific test file
python -m unittest tests.test_cyclic_mvdr

# Run with verbose output
python -m unittest discover -s tests -v
```

### Test Structure
- Tests are located in the `tests/` directory
- Test files follow the naming convention `test_*.py`
- Use `unittest` framework for all tests
- Tests should be fast and focused on specific functionality
- Integration tests should use small data sizes to keep tests fast

### Writing Tests
- Set up test fixtures in `setUp()` method
- Use small array sizes (e.g., M=2, K=5) for fast execution
- Test edge cases (single channel, zero inputs, boundary conditions)
- Verify shapes and dtypes of outputs
- Use `np.allclose()` for floating-point comparisons

## Command-Line Tools

The package provides two CLI tools:

### cmvdr
Runs beamforming inference on audio files:
```bash
cmvdr -i INPUT_PATH [-o OUTPUT_PATH] [-n NOISE_PATH] [-v]
```

### cmvdr-eval
Evaluates audio quality metrics:
```bash
cmvdr-eval -d FOLDER_DENOISED [-r FOLDER_REFERENCE] [--sort-by-snr]
```

## Running Experiments

### Configuration
- Experiment parameters are in YAML files in `configs/` directory
- Main configuration files: `cmvdr.yaml`, `default.yaml`
- Inference configuration: `inference_cmvdr.yaml`

### Experiment Scripts
```bash
# Quick demo
source script/run_demo.sh

# Synthetic data experiments
source script/run_synthetic.sh

# All experiments
source script/run_all.sh

# Direct execution
python main.py --data_type synthetic  # or instruments
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Screeen/cmpdr](https://github.com/Screeen/cmpdr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
