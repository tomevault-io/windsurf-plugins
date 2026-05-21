---
trigger: always_on
description: WeatherFlow is a Python library for weather prediction using flow matching techniques. It's built on PyTorch and provides a flexible framework for developing weather models that integrate with ERA5 reanalysis data and incorporate physics-guided neural network architectures.
---

# WeatherFlow Copilot Instructions

## Project Overview

WeatherFlow is a Python library for weather prediction using flow matching techniques. It's built on PyTorch and provides a flexible framework for developing weather models that integrate with ERA5 reanalysis data and incorporate physics-guided neural network architectures.

**Version:** 0.4.2  
**Python:** 3.8+  
**License:** MIT

## Architecture & Key Components

### Core Modules

- **`weatherflow/data/`** - ERA5 data loading and preprocessing
  - `ERA5Dataset`: Main dataset class for loading ERA5 reanalysis data
  - `create_data_loaders()`: Helper for creating train/val data loaders
  - Supports both local netCDF files and WeatherBench2 remote data

- **`weatherflow/models/`** - Neural network models
  - `WeatherFlowMatch`: Primary flow matching model for weather prediction
  - `WeatherFlowODE`: ODE solver wrapper for generating predictions
  - `PhysicsGuidedAttention`: Attention mechanisms with physical constraints
  - `StochasticFlowModel`: Stochastic variations for ensemble forecasting

- **`weatherflow/physics/`** - Physics-informed constraints and losses
  - Conservation laws (potential vorticity, mass, energy)
  - Geostrophic balance constraints
  - Energy spectra calculations

- **`weatherflow/utils/`** - Visualization and evaluation utilities
  - `WeatherVisualizer`: Plotting and animation tools
  - `FlowVisualizer`: Vector field visualizations
  - `WeatherMetrics`: Evaluation metrics for weather predictions

- **`weatherflow/training/`** - Training infrastructure
  - `FlowTrainer`: Main training loop with physics losses
  - `compute_flow_loss()`: Flow matching loss computation

- **`weatherflow/education/`** - Educational tools
  - `GraduateAtmosphericDynamicsTool`: Interactive dashboards for teaching

- **`weatherflow/server/`** - FastAPI web service
- **`frontend/`** - React-based interactive dashboard

### Project Structure

```
weatherflow/
├── weatherflow/          # Main Python package
│   ├── data/            # Data loading modules
│   ├── models/          # Neural network architectures
│   ├── physics/         # Physics constraints
│   ├── utils/           # Utilities and visualization
│   ├── training/        # Training loops
│   └── ...
├── tests/               # Test suite
├── examples/            # Example scripts
├── applications/        # Real-world applications (renewable energy, etc.)
├── model_zoo/          # Pre-trained model infrastructure
├── notebooks/          # Jupyter notebooks
├── frontend/           # React web interface
└── docs/               # MkDocs documentation
```

## Coding Conventions

### Style Guidelines

- **Follow PEP 8** for Python code style
- **Line length:** 88 characters (Black formatter default)
- **Use Black** for code formatting
- **Use isort** with Black profile for import sorting
- **Type hints:** Required for all function signatures
- **Docstrings:** Use Google-style docstrings

### Code Style Tools

Run these before committing:
```bash
black .
isort --profile black .
flake8
mypy
```

Or install pre-commit hooks:
```bash
pre-commit install
```

### Import Organization

Imports should be organized by isort with Black profile:
1. Standard library imports
2. Third-party imports
3. Local application imports

### Naming Conventions

- **Classes:** PascalCase (e.g., `WeatherFlowMatch`, `ERA5Dataset`)
- **Functions/methods:** snake_case (e.g., `create_data_loaders`, `compute_flow_loss`)
- **Constants:** UPPER_SNAKE_CASE (e.g., `R_EARTH`)
- **Private members:** Prefix with underscore (e.g., `_apply_physics_constraints`)

### Type Hints

Always use type hints for function signatures:

```python
def create_data_loaders(
    variables: List[str],
    pressure_levels: List[int],
    train_slice: Tuple[str, str],
    val_slice: Tuple[str, str],
    batch_size: int = 32
) -> Tuple[DataLoader, DataLoader]:
    """Create train and validation data loaders."""
    ...
```

### Docstrings

Use Google-style docstrings:

```python
def compute_flow_loss(x0: torch.Tensor, x1: torch.Tensor, t: torch.Tensor) -> Dict[str, torch.Tensor]:
    """Compute flow matching loss.

    Args:
        x0: Initial state tensor of shape [batch, channels, lat, lon]
        x1: Target state tensor of shape [batch, channels, lat, lon]
        t: Time values of shape [batch]

    Returns:
        Dictionary containing loss components:
            - 'total_loss': Combined loss value
            - 'flow_loss': Flow matching loss
            - 'physics_loss': Physics constraint violation (if enabled)

    Raises:
        ValueError: If input shapes are incompatible
    """
    ...
```

## Testing Practices

### Test Structure

- Tests located in `tests/` directory
- Test files named `test_*.py`
- Use pytest framework
- Test functions named `test_*`

### Running Tests

```bash
# Run all tests
pytest tests/

# Run specific test file
pytest tests/test_models.py

# Run with coverage
pytest --cov=weatherflow tests/
```

### Test Patterns

1. **Use fixtures** for common test data:
```python
@pytest.fixture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monksealseal/weatherflow](https://github.com/monksealseal/weatherflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
