---
trigger: always_on
description: Manages models and their datasets. Responsible for lazy-loading models from HuggingFace, local paths, or model definitions.
---

# FusionBench AI Coding Agent Instructions

## Project Overview

FusionBench is a comprehensive benchmark and toolkit for deep model fusion—merging, ensembling, or mixing multiple neural networks into unified models. The project supports various fusion techniques across computer vision (CLIP, ResNet, etc.) and NLP (GPT-2, Llama, Flan-T5) tasks.

**Core Purpose**: Enable researchers to experiment with model fusion algorithms through a unified, configuration-driven CLI and programmatic API.

## Architecture: The Three-Component System

FusionBench follows a strict three-component architecture that you MUST understand:

### 1. **Method** (`fusion_bench/method/`)
The fusion algorithm implementation. Each algorithm inherits from `BaseAlgorithm`.

```python
# Example: fusion_bench/method/simple_average/simple_average.py
class SimpleAverageAlgorithm(BaseAlgorithm):
    def run(self, modelpool: BaseModelPool):
        # Implement fusion logic here
        return merged_model
```

**Key Conventions**:
- File path: `fusion_bench/method/{method_name}/{variant}.py`
- Must implement `run(self, modelpool)` abstract method
- Use `_config_mapping` dict to map attributes to YAML config keys (or use `@auto_register_config` decorator)
- Lifecycle hooks: `on_run_start()` and `on_run_end()`
- Algorithm examples: `simple_average`, `task_arithmetic`, `adamerging`, `regmean`, `dare`, `ties_merging`, `model_stock`, `slerp`

### 2. **ModelPool** (`fusion_bench/modelpool/`)
Manages models and their datasets. Responsible for lazy-loading models from HuggingFace, local paths, or model definitions.

```python
# Example: fusion_bench/modelpool/clip_vision.py
class CLIPVisionModelPool(BaseModelPool):
    def load_model(self, model_name: str) -> nn.Module:
        # Load and return model
```

**Key Conventions**:
- Inherits from `BaseModelPool`
- Supports special model names: `_pretrained_` (base/pretrained model)
- Configuration specifies models as dict with model names as keys
- Can include `train_datasets`, `val_datasets`, `test_datasets` in config
- Property `model_names` returns only regular models (excludes special ones like `_pretrained_`)

### 3. **TaskPool** (`fusion_bench/taskpool/`)
Evaluates fused models on specific tasks/datasets.

```python
# Example: fusion_bench/taskpool/clip_vision.py
class CLIPVisionModelTaskPool(BaseTaskPool):
    def evaluate(self, model: nn.Module) -> Dict[str, Any]:
        # Run evaluation and return metrics
```

**Key Conventions**:
- Inherits from `BaseTaskPool`
- Implements `evaluate(model)` to return metrics dict
- Test datasets configured in YAML under `test_datasets`

## Configuration System: Hydra-Based YAML

FusionBench uses [Hydra](https://hydra.cc/) for hierarchical configuration. Understanding this is CRITICAL.

### Configuration Structure
```
config/
├── fabric_model_fusion.yaml     # Main entry point
├── method/                      # Fusion algorithms
│   ├── simple_average.yaml
│   └── task_arithmetic.yaml
├── modelpool/                   # Model pool definitions
│   └── CLIPVisionModelPool/
│       └── clip-vit-base-patch32_TA8.yaml
└── taskpool/                    # Task definitions
    └── CLIPVisionModelTaskPool/
        └── clip-vit-classification_TA8.yaml
```

### YAML Configuration Pattern
Every component config MUST include `_target_` pointing to the Python class:

```yaml
# config/method/simple_average.yaml
_target_: fusion_bench.method.SimpleAverageAlgorithm
# algorithm hyperparameters here

# config/modelpool/my_pool.yaml
_target_: fusion_bench.modelpool.CLIPVisionModelPool
models:
  _pretrained_: openai/clip-vit-base-patch32
  task1: path/to/finetuned/model
  task2: path/to/another/model
```

### `_config_mapping` Pattern
To serialize class attributes to YAML config, you have two options:

**Option 1: Manual mapping with `_config_mapping`**:
```python
class MyAlgorithm(BaseAlgorithm):
    _config_mapping = BaseAlgorithm._config_mapping | {
        "learning_rate": "lr",      # self.learning_rate -> config.lr
        "num_epochs": "epochs",     # self.num_epochs -> config.epochs
    }
```

**Option 2: Automatic registration with `@auto_register_config` decorator**:
```python
from fusion_bench.mixins import auto_register_config

@auto_register_config
class MyAlgorithm(BaseAlgorithm):
    def __init__(self, learning_rate: float, num_epochs: int):
        # Parameters are automatically registered and set as attributes
        super().__init__()
```

The `@auto_register_config` decorator automatically:
- Maps all `__init__` parameters to `_config_mapping`
- Sets instance attributes from constructor arguments
- Applies default values when parameters are not provided

## Running FusionBench

### CLI Usage (Primary Interface)
```bash
# Basic command structure
fusion_bench \
  method=simple_average \
  modelpool=CLIPVisionModelPool/clip-vit-base-patch32_TA8 \
  taskpool=clip_vision \
  method.some_param=value  # Override config values

# The CLI entry point is fusion_bench/scripts/cli.py
# It uses @hydra.main decorator with config_path pointing to config/
# The default config name is "fabric_model_fusion.yaml"

# There's also a web UI available:
fusion_bench_webui
```

### Programmatic Usage
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanganke/fusion_bench](https://github.com/tanganke/fusion_bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
