---
trigger: always_on
description: **ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information here is incomplete or found to be in error.**
---

# Prophetverse Development Instructions

**ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information here is incomplete or found to be in error.**

Prophetverse is a Python library for time series forecasting built on sktime and numpyro. It provides Bayesian forecasting models with custom priors, non-linear effects, and multiple likelihood distributions for Marketing Mix Modeling and time series analysis.

## Working Effectively

### Bootstrap and Environment Setup
- Install Poetry (dependency manager): `pip install poetry`
- **NEVER CANCEL**: Install dependencies: `poetry install --extras dev` -- takes 13+ minutes to complete. Set timeout to 20+ minutes.
- Set Python path for development: `export PYTHONPATH=$PWD/src:$PYTHONPATH`
- Install pre-commit hooks: `poetry run pre-commit install`

### Testing
- **Fast smoke tests**: `poetry run pytest -m "smoke"` -- takes ~1 minute
- **Faster smoke tests (non-CI)**: `poetry run pytest -m "smoke and not ci"` -- takes ~1 minute  
- **NEVER CANCEL**: Full test suite: `poetry run pytest -m "not smoke"` -- takes 90+ minutes. Set timeout to 120+ minutes.
- **CI-style tests**: `poetry run pytest --cov=prophetverse --cov-report=xml -m "not smoke" --durations=10`
- Test with coverage: `poetry run pytest --cov=prophetverse --cov-report=xml`

### Code Quality and Linting
- Format code: `poetry run black src tests`
- Check formatting: `poetry run black --check src tests` -- takes ~1 second
- Sort imports: `poetry run isort src tests`
- Type checking: `poetry run mypy src` (excludes tests)
- Run all pre-commit checks: `poetry run pre-commit run --all-files` -- may fail due to network issues, use individual tools instead
- Lint specific files: `poetry run pre-commit run --files <modified_files>`

### Validation
- **ALWAYS run a complete validation scenario after making changes**:
```python
# Test basic functionality
poetry run python -c "
from prophetverse.sktime import Prophetverse
from prophetverse.datasets import load_peyton_manning

# Load sample data
df = load_peyton_manning()
print('Data loaded, shape:', df.shape)

# Test model creation and fitting
model = Prophetverse()
y_train = df.iloc[:50]  # Use small subset for quick test
model.fit(y=y_train)
print('✓ Model fitting successful')

# Test prediction
pred = model.predict(fh=[1, 2, 3])
print('✓ Prediction successful, shape:', pred.shape)
"
```

### Documentation
- Documentation uses Quarto (not mkdocs): builds happen in GitHub Actions
- Development docs: `docs/development.qmd`
- Reference docs: `docs/reference/index.qmd`
- Tutorials: `docs/tutorials/`

## Critical Timing and Timeout Requirements

| Command | Expected Time | Minimum Timeout | Notes |
|---------|---------------|-----------------|-------|
| `poetry install --extras dev` | 13+ minutes | 20+ minutes | **NEVER CANCEL** - Downloads many ML dependencies |
| `poetry run pytest -m "smoke"` | 1-2 minutes | 5+ minutes | Fast tests for development |
| `poetry run pytest -m "not smoke"` | 90+ minutes | 120+ minutes | **NEVER CANCEL** - Full CI test suite |
| `poetry run black --check src tests` | <5 seconds | 30 seconds | Quick formatting check |
| `poetry run pre-commit run --all-files` | 5-30 minutes | 45+ minutes | May fail due to network issues |

## Repository Structure

### Key Directories and Files
```
/home/runner/work/prophetverse/prophetverse/
├── src/prophetverse/           # Main source code
│   ├── sktime/                 # Sktime-compatible models (Prophet, HierarchicalProphet)
│   ├── effects/                # Exogenous effects and likelihoods
│   ├── engine/                 # Inference engines (MCMC, MAP, VI)
│   ├── datasets/               # Built-in datasets
│   └── budget_optimization/    # Marketing Mix Modeling tools
├── tests/                      # Test suite with pytest markers
├── docs/                       # Quarto documentation source
├── .github/workflows/          # CI pipelines
└── pyproject.toml             # Poetry configuration
```

### Important Files to Check After Changes
- Always verify `src/prophetverse/sktime/` models after core changes
- Check `tests/sktime/test_univariate.py` and `tests/sktime/test_multivariate.py` for model tests
- Review `tests/effects/` when modifying effects or likelihoods
- Examine `.pre-commit-config.yaml` for code quality requirements

## Common Tasks and Validation Steps

### After Making Code Changes
1. **ALWAYS** run fast validation: `poetry run pytest -m "smoke and not ci"`
2. **ALWAYS** format code: `poetry run black src tests`
3. **ALWAYS** run the validation scenario (see above)
4. **ALWAYS** check imports work: `poetry run python -c "from prophetverse.sktime import Prophetverse; print('✓ Imports OK')"`
5. Before committing: `poetry run black --check src tests && poetry run isort --check-only src tests`

### Debugging Failed Tests
- Use `poetry run pytest -v -s tests/path/to/specific_test.py::test_name` for detailed output
- Some tests may fail due to probabilistic nature - re-run to confirm
- **Known failing tests** (not your responsibility to fix):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felipeangelimvieira/prophetverse](https://github.com/felipeangelimvieira/prophetverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
