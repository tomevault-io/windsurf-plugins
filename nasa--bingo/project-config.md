---
trigger: always_on
description: Bingo is an open-source Python package developed by NASA for symbolic regression and general-purpose evolutionary optimization using genetic programming. The project implements parallel island evolution strategies with MPI support and includes optional C++ performance optimizations via BingoCpp.
---

# Bingo Copilot Instructions

## Repository Overview

Bingo is an open-source Python package developed by NASA for symbolic regression and general-purpose evolutionary optimization using genetic programming. The project implements parallel island evolution strategies with MPI support and includes optional C++ performance optimizations via BingoCpp.

## Technology Stack

### Primary Language & Version
- **Python**: >=3.9 (supports 3.9, 3.10, 3.11, 3.12)
- **C++**: Used for performance-critical components via pybind11

### Core Dependencies
- `numpy`: Numerical computations
- `scipy`: Scientific computing
- `mpi4py>=4.0`: Parallel computing with MPI
- `sympy`: Symbolic mathematics
- `scikit-learn>=1.1`: Machine learning utilities
- `dill>=0.2.9`: Serialization
- `smcpy>=0.1.4`: Statistical methods
- `pybind11`: Python-C++ bindings

### Build System
- **setuptools** with CMake for C++ extensions
- **pytest** for testing
- **Sphinx** for documentation

## Project Structure

```
bingo/
├── bingo/                          # Main Python package
│   ├── chromosomes/                # Genetic representation
│   ├── evaluation/                 # Fitness evaluation
│   ├── evolutionary_algorithms/    # EA implementations
│   ├── evolutionary_optimizers/    # Optimization strategies (Island, Archipelago)
│   ├── local_optimizers/           # Local optimization methods
│   ├── selection/                  # Selection operators (Tournament, etc.)
│   ├── stats/                      # Statistics and tracking
│   ├── symbolic_regression/        # Symbolic regression specific code
│   │   ├── agraph/                # Acyclic graph representation
│   │   └── benchmarking/          # Benchmark problems
│   ├── util/                       # Utilities
│   └── variation/                  # Genetic operators (crossover, mutation)
├── bingocpp/                       # C++ performance library (submodule)
├── tests/                          # Test suite
├── examples/                       # Usage examples
├── docs/                           # Sphinx documentation
└── .github/workflows/              # CI/CD pipelines
```

## Coding Standards

### Python Style
- Follow **PEP 8** conventions
- Use **numpy-style docstrings** (configured in `.pydocstyle`)
- Maintain compatibility with Python 3.9+
- Use type hints where appropriate but not required

### Linting & Code Quality
- **pylint**: Configuration in `.pylintrc`
- **pydocstyle**: Configuration in `.pydocstyle`
  - Convention: numpy
  - Ignored rules: D204, D401, D400, D205, D404, D104, D105, D210, D403, D200, D209

### Documentation
- Use numpy-style docstrings for all public APIs
- Include docstring sections: Parameters, Returns, Raises, Examples where applicable
- Documentation built with Sphinx and hosted at https://nasa.github.io/bingo/

## Design Patterns & Architecture

### Evolutionary Algorithm Components
- **Abstract base classes**: Use ABCMeta for interfaces (e.g., `FitnessFunction`, `Variation`)
- **Genetic operators**: Implement as callable classes with `__call__` method
- **Variation strategies**: 
  - `VarOr`: Mutation OR crossover OR replication
  - `VarAnd`: Mutation AND crossover
- **Fitness evaluation**: Vector-based with aggregation metrics (MAE, RMSE, MSE, etc.)

### Code Organization Principles
- Each module should have a clear, single responsibility
- Use composition over inheritance where practical
- Maintain separation between evolutionary framework and symbolic regression specifics
- Support both serial and parallel execution modes

## Testing Requirements

### Test Framework
- Use **pytest** for all tests
- Test files located in `tests/` directory
- Run tests with: `tests/.run_tests.sh` or `pytest tests`

### Test Structure
- Unit tests in `tests/unit/`
- Integration tests as needed
- Use `pytest-mock` for mocking
- Use `pytest-timeout` to prevent hanging tests
- Target high code coverage (tracked with coverage.py and Coveralls)

### Testing Guidelines
- Write tests for all new functionality
- Maintain or improve existing code coverage
- Test both Python and C++ components when applicable
- Include tests for parallel execution paths where relevant

## Build & Development

### Installation from Source
```bash
git clone --recurse-submodules https://github.com/nasa/bingo.git
cd bingo
pip install -r requirements.txt
./.build_bingocpp.sh  # Optional: build C++ extensions
pytest tests
```

### Building C++ Extensions
- BingoCpp is a git submodule providing performance optimizations
- Built via CMake integration in setup.py
- Optional but recommended for performance
- If build fails, Python fallback implementations are used

### Environment Setup
- Set `PYTHONPATH` to bingo root directory for development
- Set `RDMAV_FORK_SAFE=1` for MPI environments
- Use conda or pip for dependency management

## Key Components & Usage

### Symbolic Regression
- Primary use case: symbolic regression via genetic programming
- **SymbolicRegressor**: scikit-learn compatible interface
- **AGraph**: Acyclic graph representation of equations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nasa/bingo](https://github.com/nasa/bingo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
