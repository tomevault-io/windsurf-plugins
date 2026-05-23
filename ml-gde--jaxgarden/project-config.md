---
trigger: always_on
description: description: JAXgarden tutorial chapter detailing BaseConfig, the base dataclass for managing model hyperparameters and configurations.
---

---
description: JAXgarden tutorial chapter detailing BaseConfig, the base dataclass for managing model hyperparameters and configurations.
globs: 
alwaysApply: false
---
# Chapter 3: BaseConfig

In the [previous chapter](basemodel.mdc), we explored `BaseModel`, the foundational class for models in `jaxgarden`. We saw that each `BaseModel` instance is initialized with a configuration object. This chapter dives into the base class for those configuration objects: `BaseConfig`.

**Motivation:** Neural models are complex systems with numerous hyperparameters (like hidden size, number of layers, dropout rates) and settings (like data types, precision). Managing these configurations consistently across different models is crucial for reproducibility, maintainability, and ease of use. Using simple dictionaries or ad-hoc parameter passing can quickly become messy and error-prone. `BaseConfig` provides a standardized, structured way to define, manage, and serialize these configurations.

**Central Use Case:** Defining the set of hyperparameters for a new model implementation (e.g., creating a `MyTransformerConfig` that inherits from `BaseConfig`) or inspecting and modifying the configuration of an existing `jaxgarden` model like [LlamaForCausalLM](llamaforcausallm.mdc), which uses its own `LlamaConfig` subclass derived from `BaseConfig`.

## Key Concepts

`BaseConfig` leverages Python's `dataclasses` to provide a simple yet powerful configuration system:

1.  **Dataclass Structure:** It's defined using the `@dataclass` decorator, offering type hints, default values, and automatic `__init__` generation.
2.  **Inheritance:** Model-specific configurations (e.g., `LlamaConfig`, `ModernBERTConfig`) inherit from `BaseConfig`, adding their unique parameters while retaining the common base attributes.
3.  **Common Attributes:** Provides essential base attributes applicable to most models, such as `seed` for reproducibility and `log_level` for controlling verbosity.
4.  **Extensibility:** An `extra` dictionary allows storing arbitrary additional parameters not explicitly defined in the dataclass fields, offering flexibility.
5.  **Serialization:** The `to_dict()` method converts the configuration object into a standard Python dictionary, useful for logging, saving, or inspection.
6.  **Programmatic Updates:** The `update()` method allows modifying configuration attributes after instantiation using keyword arguments.

## Using `BaseConfig`

You typically interact with subclasses of `BaseConfig` tailored to specific models.

### Defining a Custom Configuration

To define a configuration for a new model, create a class inheriting from `BaseConfig` and use the `@dataclass` decorator. Add fields for your model's specific hyperparameters.

```python
from dataclasses import dataclass, field
from jaxgarden.models.base import BaseConfig
from typing import Any

@dataclass
class MyModelConfig(BaseConfig):
    """Configuration for MyCustomModel."""
    hidden_size: int = 256
    num_layers: int = 4
    dropout_rate: float = 0.1
    activation_fn: str = "relu"
    # Override a base attribute default if needed
    seed: int = 123

# Instantiate the configuration
my_config = MyModelConfig(hidden_size=512) # Override default hidden_size

print(f"MyModelConfig instance: {my_config}")
# Output: MyModelConfig instance: MyModelConfig(seed=123, log_level='info', extra={}, hidden_size=512, num_layers=4, dropout_rate=0.1, activation_fn='relu')

print(f"Hidden size: {my_config.hidden_size}")
# Output: Hidden size: 512
print(f"Seed (overridden): {my_config.seed}")
# Output: Seed (overridden): 123
print(f"Log Level (from base): {my_config.log_level}")
# Output: Log Level (from base): info
```

**Explanation:**
- We define `MyModelConfig` inheriting from `BaseConfig`.
- `@dataclass` automatically generates methods like `__init__`.
- We add model-specific fields like `hidden_size` and `num_layers` with type hints and default values.
- We can override defaults during instantiation (`hidden_size=512`).
- Base attributes like `seed` and `log_level` are inherited.

### Serialization (`to_dict`)

Convert the configuration object to a dictionary for easy inspection or storage.

```python
config_dict = my_config.to_dict()
print(f"Configuration as dictionary: {config_dict}")
# Output: Configuration as dictionary: {'seed': 123, 'log_level': 'info', 'extra': {}, 'hidden_size': 512, 'num_layers': 4, 'dropout_rate': 0.1, 'activation_fn': 'relu'}

import json
print(f"Configuration as JSON: {json.dumps(config_dict, indent=2)}")
# Output:
# Configuration as JSON: {
#  "seed": 123,
#  "log_level": "info",
#  "extra": {},
#  "hidden_size": 512,
#  "num_layers": 4,
#  "dropout_rate": 0.1,
#  "activation_fn": "relu"
# }
```

**Explanation:** The `to_dict()` method simply returns the internal `__dict__` of the dataclass instance, making it compatible with standard serialization libraries like `json`.

### Programmatic Updates (`update`)

Modify configuration values after the object has been created.

```python
print(f"Original dropout rate: {my_config.dropout_rate}")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-gde/jaxgarden](https://github.com/ml-gde/jaxgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
