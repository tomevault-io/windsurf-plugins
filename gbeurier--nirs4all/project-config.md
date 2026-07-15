---
trigger: always_on
description: This repository/workspace contains two related projects for Near-Infrared Spectroscopy (NIRS) analysis:
---

````markdown
# AI Instructions (Claude Code + GitHub Copilot) — nirs4all Workspace

This repository/workspace contains two related projects for Near-Infrared Spectroscopy (NIRS) analysis:

1. **nirs4all** (`/home/delete/nirs4all`) — Python library for NIRS data analysis with ML pipelines
---

## nirs4all (Python Library)

**Version**: 0.7.x | **Python**: 3.11+ | **License**: CeCILL-2.1

### Quick Reference (Minimal Example)

```python
import nirs4all
from sklearn.preprocessing import MinMaxScaler
from sklearn.cross_decomposition import PLSRegression

result = nirs4all.run(
    pipeline=[MinMaxScaler(), PLSRegression(10)],
    dataset="sample_data/regression",
    verbose=1,
)
print(f"Best RMSE: {result.best_rmse:.4f}")
````

### Primary API (Module-Level)

Prefer the module-level API functions:

| Function                               | Purpose                 |
| -------------------------------------- | ----------------------- |
| `nirs4all.run(pipeline, dataset, ...)` | Train a pipeline        |
| `nirs4all.predict(model, data, ...)`   | Make predictions        |
| `nirs4all.explain(model, data, ...)`   | SHAP explanations       |
| `nirs4all.retrain(source, data, ...)`  | Retrain on new data     |
| `nirs4all.session(...)`                | Create reusable session |
| `nirs4all.generate(...)`               | Generate synthetic data |

**Result objects**: `RunResult`, `PredictResult`, `ExplainResult` expose `best_score`, `best_rmse`, `best_r2`, `top(n)`, `export()`.

### Commands

```bash
# Tests
pytest tests/                     # All tests
pytest tests/unit/                # Unit tests only
pytest tests/integration/         # Integration tests
pytest -m sklearn                 # sklearn-only tests
pytest --cov=nirs4all             # With coverage

# Examples (from examples/ directory)
./run.sh                          # All examples
./run.sh -c user                  # User examples only
./run.sh -n "U01*"                # By pattern

# Code quality
ruff check .                      # Linting
mypy .                            # Type checking

# Installation verification
nirs4all --test-install
nirs4all --test-integration
```

### Architecture

```
nirs4all/
├── api/           # Primary interface: run(), predict(), explain(), retrain(), session(), generate()
├── pipeline/      # Execution engine (PipelineRunner/Orchestrator), bundle export (.n4a), prediction, retraining
├── controllers/   # Registry pattern for step handlers (@register_controller)
├── data/          # SpectroDataset (core container with X, y, metadata, folds)
├── operators/     # Transforms (SNV, MSC, SG), models (NICON), splitters (KS, SPXY), augmentation
├── sklearn/       # NIRSPipeline wrapper for SHAP compatibility
└── visualization/ # PredictionAnalyzer, heatmaps, candlestick charts
```

**Key classes**: `SpectroDataset`, `PipelineConfigs`, `DatasetConfigs`, `NIRSPipeline`

### Pipeline Syntax

Steps can be classes, instances, or wrapped in dicts:

```python
from sklearn.preprocessing import MinMaxScaler
from sklearn.cross_decomposition import PLSRegression
from sklearn.model_selection import ShuffleSplit

pipeline = [
    MinMaxScaler(),                              # Transformer instance
    {"y_processing": MinMaxScaler()},            # Target scaling
    ShuffleSplit(n_splits=3),                    # Cross-validation splitter
    {"model": PLSRegression(n_components=10)},   # Model step
]
```

#### Special Keywords

| Keyword         | Purpose                     | Example                                               |
| --------------- | --------------------------- | ----------------------------------------------------- |
| `model`         | Define model step           | `{"model": PLSRegression(10)}`                        |
| `y_processing`  | Target scaling              | `{"y_processing": MinMaxScaler()}`                    |
| `branch`        | Parallel pipelines          | `{"branch": [[SNV(), PLS()], [MSC(), RF()]]}`         |
| `merge`         | Combine branches (stacking) | `{"merge": "predictions"}`                            |
| `source_branch` | Per-source preprocessing    | `{"source_branch": {"NIR": [...], "markers": [...]}}` |
| `_or_`          | Generator (variants)        | `{"_or_": [SNV, MSC, Detrend]}`                       |
| `_range_`       | Parameter sweep             | `{"_range_": [1, 30, 5], "param": "n_components"}`    |

### Controller Pattern (Registry)

Custom operators should follow the controller registry pattern:

```python
from nirs4all.controllers import register_controller, OperatorController

@register_controller
class MyController(OperatorController):
    priority = 50  # Lower = higher priority

    @classmethod
    def matches(cls, step, operator, keyword) -> bool:
        return isinstance(operator, MyOperatorType)

    @classmethod
    def use_multi_source(cls) -> bool:
        return False

    @classmethod
    def supports_prediction_mode(cls) -> bool:
        return True  # Run during prediction

    def execute(self, step_info, dataset, context, runtime_context, **kwargs):
        # Transform dataset; return (context, StepOutput)
        pass
```

### Common Tasks

#### Generate Synthetic Data

```python
import nirs4all


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GBeurier/nirs4all](https://github.com/GBeurier/nirs4all) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
