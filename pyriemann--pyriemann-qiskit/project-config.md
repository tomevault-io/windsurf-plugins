---
trigger: always_on
description: This document provides guidance for AI coding agents working on the pyRiemann-qiskit
---

# AI Agent Guidelines for pyRiemann-qiskit

This document provides guidance for AI coding agents working on the pyRiemann-qiskit
repository.

## Project Overview

**pyRiemann-qiskit** is a Python library that bridges Riemannian geometry (via pyRiemann)
with quantum computing (via Qiskit) for machine learning applications. The library focuses
on:

- Quantum-enhanced classification for Brain-Computer Interface (BCI) and EEG data
- Quantum Support Vector Classifiers (QSVC) and Variational Quantum Classifiers (VQC)
- Quantum implementations of the Minimum Distance to Mean (MDM) algorithm
- Integration with Riemannian geometry for covariance matrix processing

## Technology Stack

- **Core**: Python 3.10-3.12
- **Quantum Computing**: Qiskit 1.x, Qiskit Machine Learning 0.7.2, Qiskit Algorithms
  0.3.1
- **Riemannian Geometry**: pyRiemann 0.9
- **Machine Learning**: scikit-learn 1.5.2
- **Optimization**: CVXPY 1.6.5, DOcplex 2.29.245
- **Scientific Computing**: NumPy <2.3, SciPy 1.13.1
- **Testing**: pytest
- **Documentation**: Sphinx, sphinx-gallery
- **Code Quality**: flake8

## Repository Structure

```
pyriemann_qiskit/
├── classification/       # Quantum classification algorithms and wrappers
├── datasets/            # Dataset utilities
├── utils/               # Core utilities (distance, mean, preprocessing, etc.)
├── visualization/       # Visualization tools (manifold, quantum art)
├── autoencoders.py      # Quantum autoencoder implementations
├── ensemble.py          # Ensemble methods
└── pipelines.py         # ML pipeline utilities

examples/
├── ERP/                 # Event-Related Potential examples
├── MI/                  # Motor Imagery examples
├── other_datasets/      # Financial data, Titanic, etc.
├── resting_states/      # Resting state analysis
└── toys_dataset/        # Toy dataset examples

tests/                   # Unit tests
doc/                     # Sphinx documentation
benchmarks/              # Performance benchmarks
```

## Key Design Patterns

- **scikit-learn interface**: All classifiers implement `fit` / `predict` / `score`. New
  classifiers should subclass `QuanticClassifierBase` in `classification/wrappers.py`.
- **Optimizer hierarchy**: `pyQiskitOptimizer` (in `utils/docplex.py`) is the abstract
  base. Subclasses override `spdmat_var` (variable type), `_solve_qp` (solve logic), and
  `get_weights`. Concrete subclasses: `ClassicalOptimizer`, `NaiveQAOAOptimizer`,
  `QAOACVOptimizer`, `QAOACVAngleOptimizer`.
- **Quantum backend abstraction**: `utils/quantum_provider.py` → `get_simulator()` returns
  an AerSimulator by default. Callers should not import backends directly.
- **Docplex ↔ Qiskit bridge**: Optimization problems are defined as Docplex models,
  converted via `from_docplex_mp`, then solved with QAOA or classical Cobyla.
- **Hyperparameter factory**: `utils/hyper_params_factory.py` centralises circuit
  configuration (feature maps, ansätze, mixers). Use `create_mixer_rotational_X_gates` for
  QAOA mixers.
- **Symbolic statevector cache**: Pre-computed statevectors live in `symb_statevectors/`
  (XFeatureMap, ZFeatureMap, ZZFeatureMap). Do not delete or regenerate these unless
  intentional.

## Key Constraints & Considerations

### Quantum Computing Limitations

1. **Qubit Limitations**:

   - Local simulator: ~36 qubits (memory-dependent)
   - Remote simulator: up to 5000 qubits
   - Free quantum hardware: 7 qubits
   - Exploratory hardware: 127 qubits (not public)

2. **Time Complexity**: Quantum algorithms are computationally expensive, especially
   locally. Use limited trials in examples but note this limitation.

3. **Offline Only**: Due to time constraints and queuing on remote backends, this library
   is designed for offline analysis only.

### Python Version Support

- **Supported**: Python 3.10 - 3.12
- Always ensure compatibility across this range

### Platform Support

- Windows, macOS, and Linux
- GPU optimization only available on Linux (qiskit-aer-gpu)
- Symbolic simulation available via qiskit-symb

## Development Guidelines

### Code Style

1. **Formatting**: Use `flake8` for code formatting

   ```bash
   flake8
   ```

2. **Follow scikit-learn conventions**: The library follows scikit-learn best practices
   for estimators and transformers

3. **Documentation**: Always provide full documentation for public methods using NumPy
   docstring format

### Testing

1. **Run tests locally**:

   ```bash
   pip install .[tests]
   pytest
   ```

2. **Test specific files**:

   ```bash
   pytest tests/test_classification.py
   ```

3. **Coverage**: Maintain test coverage for new features

4. **Test fixtures** (from `tests/conftest.py`):
   - `get_covmats(n_matrices, n_channels)` — generates SPD covariance matrices
   - `get_labels(n_matrices, n_classes)` — generates classification labels
   - `get_dataset(kind)` — generates full `(X, y)` datasets; kinds: `rand`, `bin`,
     `rand_cov`, `bin_cov`
   - `requires_matplotlib` / `requires_seaborn` — decorators for optional-dependency tests
   - Use `BinaryTest` / `MultiClassTest` base classes for classifier tests

### Git Workflow

1. **Avoid merge commits**: Use `git rebase` instead of merging master into feature
   branches

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pyRiemann/pyRiemann-qiskit](https://github.com/pyRiemann/pyRiemann-qiskit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
