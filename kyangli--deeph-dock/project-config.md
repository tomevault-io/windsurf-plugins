---
trigger: always_on
description: pip install -e .                  # Editable install (development)
---

# DeepH-dock Development Guide for AI Agents

## Build, Install & Test Commands

### Installation
```bash
pip install -e .                  # Editable install (development)
uv build --wheel -o dist ./       # Build distribution wheel
```
- Python version: **3.12-3.14** required

### Testing
```bash
bash tests/run_test_all.sh                           # Run all tests
bash tests/<module>/<submodule>/run_test.sh           # Run single test
bash tests/convert/siesta/run_test.sh                 # Example: SIESTA converter
bash tests/compute/eigen/run_test.sh                  # Example: Eigenvalue calculation
```
- Tests are **bash scripts** that execute CLI commands and validate outputs
- Test scripts invoke `dock` commands and compare results against reference data
- Clean test artifacts: `bash tests/_clean.sh` (interactive)

### Linting & Formatting
```bash
black .                           # Format code
ruff check .                      # Lint code
ruff check --fix .                # Auto-fix linting issues
```
- Line length: **120 characters** (configured in pyproject.toml)

## Code Style Guidelines

### Imports Organization
```python
# Standard library (alphabetically)
from pathlib import Path
from typing import List, Optional, Tuple

# Third-party (alphabetically)
import h5py
import numpy as np
from click import argument, option

# Local imports (alphabetically)
from deepx_dock.parallel import parallel_map
from deepx_dock.CONSTANT import DEEPX_HAMILTONIAN_FILENAME
from deepx_dock.misc import load_json_file
```
- Separate groups with blank lines
- Alphabetically sorted within each group
- Import specific functions/classes, not entire modules (except numpy)

### Naming Conventions
- **Functions/Variables**: `snake_case` (e.g., `translate_to_deeph`, `n_jobs`)
- **Classes**: `PascalCase` (e.g., `HamiltonianObj`, `DatasetAnalyzer`)
- **Constants**: `UPPER_CASE` (e.g., `DEEPX_HAMILTONIAN_FILENAME`, `HARTREE_TO_EV`)
- **CLI commands**: `kebab-case` (e.g., `to-deeph`, `calc-band`)
- **Private methods**: `_leading_underscore` (e.g., `_read_h5`, `_validation_check`)
- **Parameter naming**:
  - CLI options: `--jobs-num`, `--tier-num` (kebab-case)
  - CLI function parameters: `jobs_num`, `tier_num` (matches click convention)
  - Class `__init__` parameters: `n_jobs`, `n_tier` (unified `n_` prefix)

### Type Hints
```python
# Use for all function signatures
def load_poscar_file(file_path: str | Path) -> dict:
    ...

def analyze_data(
    data_path: str | Path,
    n_jobs: int = -1,
    n_tier: int = 0,
) -> None:
    ...

# Use Optional for optional parameters
from typing import Optional
def process_file(path: Path, encoding: Optional[str] = None) -> str:
    ...
```

### Docstrings
Use **NumPy style** for classes and complex functions:

```python
def r2k(self, ks: np.ndarray) -> np.ndarray:
    """
    Fourier transform from real space to reciprocal space.

    Parameters
    ----------
    ks : np.ndarray, shape (Nk, 3)
        k-points in fractional coordinates.

    Returns
    -------
    MKs : np.ndarray, shape (Nk, N_b, N_b)
        Matrices in reciprocal space.
    """
    phase = np.exp(2j * np.pi * np.matmul(ks, self.Rs.T))
    MRs_flat = self.MRs.reshape(len(self.Rs), -1)
    Mks_flat = np.matmul(phase, MRs_flat)
    return Mks_flat.reshape(len(ks), *self.MRs.shape[1:])
```

### Comments
- **NO inline comments** unless absolutely necessary for non-obvious logic
- Let code speak through clear naming and structure
- Use docstrings for documentation, not comments
- Exception: Brief comments for complex algorithms or physical constants

### Error Handling
```python
# Use assertions for internal invariants
assert self.data_dir.is_dir(), f"{data_dir} is not a directory"

# Raise explicit exceptions for user-facing errors
if symbol not in PERIODIC_TABLE_SYMBOL_TO_INDEX:
    raise KeyError(f"Unknown element symbol: {symbol}")

# Validate inputs early in __init__
def __init__(self, data_dir: str | Path):
    self.data_dir = Path(data_dir)
    assert self.data_dir.is_dir(), f"{data_dir} is not a directory"
    self.data_dir.mkdir(parents=True, exist_ok=True)
```

## Project Architecture

### Module Structure
```
deepx_dock/
├── _cli/              # CLI auto-registration system
│   ├── __init__.py    # Auto-discovery and command building
│   └── registry.py    # Function registration decorator
├── convert/           # DFT software format converters
│   ├── siesta/
│   ├── openmx/
│   ├── fhi_aims/
│   ├── abacus/
│   └── ...
├── compute/           # Electronic structure calculations
│   ├── eigen/         # Band structure, DOS, Fermi level
│   ├── overlap/       # Overlap matrix calculations
│   └── ...
├── analyze/           # Data analysis tools
│   ├── dataset/       # Feature analysis, data splitting
│   ├── error/         # Error analysis
│   └── dft_equiv/     # Equivariance testing
├── design/            # Structure generation
├── CONSTANT.py        # All constants (UPPER_CASE)
└── misc.py            # Utility functions
```

### Adding New CLI Commands

**Pattern**: Each module has a `_cli.py` file for command registration

1. **Create module directory**: `deepx_dock/<module>/<submodule>/`

2. **Implement core functionality**:
```python
# deepx_dock/convert/new_dft/translator.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kYangLi/DeepH-dock](https://github.com/kYangLi/DeepH-dock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
