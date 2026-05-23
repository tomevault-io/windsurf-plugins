---
trigger: always_on
description: description: JAXgarden tutorial chapter detailing BaseModel, the abstract base class for models, covering config, state management, I/O, and Hugging Face integration.
---

---
description: JAXgarden tutorial chapter detailing BaseModel, the abstract base class for models, covering config, state management, I/O, and Hugging Face integration.
globs: 
alwaysApply: false
---
# Chapter 2: BaseModel

In the [previous chapter](tokenizer.mdc), we learned how the `jaxgarden.Tokenizer` prepares text data into JAX arrays suitable for neural network processing. Now, we turn our attention to the core building block for the models themselves: `BaseModel`.

**Motivation:** Building diverse neural network models (like Llama, BERT, etc.) requires a consistent structure. Without a common base, each model implementation might handle configuration, parameter management (state), saving/loading checkpoints, and interacting with external resources (like the Hugging Face Hub) differently. This leads to code duplication, inconsistencies, and makes it harder to develop, maintain, and reuse components. `BaseModel` addresses this by providing an abstract base class that defines a standard interface and implements shared functionalities.

**Central Use Case:** Defining a new neural network model within the `jaxgarden` ecosystem. For example, when implementing a model like [LlamaForCausalLM](llamaforcausallm.mdc), subclassing `BaseModel` ensures it correctly handles its configuration ([BaseConfig](baseconfig.mdc)), manages its learnable parameters and mutable states using Flax NNX conventions, can be saved and loaded consistently using Orbax, and provides a standardized way to import weights from equivalent Hugging Face models.

## Key Concepts

`BaseModel` establishes several core responsibilities and provides associated features:

1.  **Configuration Management:** Each `BaseModel` instance holds a configuration object, typically a subclass of [BaseConfig](baseconfig.mdc), which stores hyperparameters and architectural details (e.g., number of layers, hidden size).
2.  **Flax NNX Foundation:** It inherits from `flax.nnx.Module`, making every `jaxgarden` model an NNX module. This enables the powerful state management capabilities of NNX.
3.  **State Handling:**
    *   `state` property: Provides easy access to the model's `nnx.State` (learnable parameters and mutable states), separated from the static graph definition.
    *   `state_dict` property: Returns the state as a nested dictionary of JAX arrays, suitable for serialization frameworks like Orbax.
4.  **Checkpointing (Save/Load):** Offers `save` and `load` methods that use `orbax-checkpoint` (`ocp`) to serialize and deserialize the model's `state_dict` to/from disk, ensuring consistent checkpointing across all models.
5.  **Hugging Face (HF) Integration Interface:** Defines a standard workflow for interacting with the Hugging Face Hub:
    *   `download_from_hf`: Static method to download model artifacts (like config and weights) from a repository.
    *   `iter_safetensors`: Static method to efficiently iterate over tensors stored in `.safetensors` files.
    *   `convert_weights_from_hf`: An *abstract* method that subclasses *must* implement to translate HF weights into the `jaxgarden` model's state structure.
    *   `from_hf`: Orchestrates the download, iteration, and conversion process to initialize a `jaxgarden` model instance with weights from the Hub.

## Using `BaseModel`

`BaseModel` is an abstract class, so you typically interact with its *subclasses* (like `LlamaForCausalLM`). However, understanding its interface is crucial for both using and implementing models.

### Initialization (Subclass Implementation)

When creating a new model (e.g., `MyCustomModel`), you must subclass `BaseModel` and call its `__init__` method using `super().__init__(...)`.

```python
import jax
import jax.numpy as jnp
from flax import nnx
from jaxgarden.models.base import BaseModel, BaseConfig
from dataclasses import dataclass

@dataclass
class MyConfig(BaseConfig):
    hidden_size: int = 128
    # ... other params

class MyCustomModel(BaseModel):
    def __init__(self, config: MyConfig, *, rngs: nnx.Rngs):
        # Call the parent BaseModel constructor FIRST
        super().__init__(config, rngs=rngs, dtype=jnp.float32) # Specify dtype, etc.

        # Initialize model layers (e.g., using nnx.Linear)
        self.dense = nnx.Linear(config.hidden_size, config.hidden_size, rngs=rngs)
        # ... other layers

    def __call__(self, x: jnp.ndarray) -> jnp.ndarray:
        # Define the forward pass
        return self.dense(x)

# --- Usage ---
config = MyConfig()
rngs = nnx.Rngs(0) # Or more sophisticated key splitting
model = MyCustomModel(config=config, rngs=rngs)
print("Model initialized.")
```

**Explanation:** The subclass `__init__` first calls `BaseModel.__init__`, passing the configuration object (`config`) and NNX random number generators (`rngs`). It also specifies other base parameters like `dtype`. Then, it proceeds to define its own layers (like `self.dense`).

### Accessing State

You can easily access the model's parameters and mutable states.

```python
# Assuming 'model' is an instance of a BaseModel subclass
# Get the nnx.State object

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-gde/jaxgarden](https://github.com/ml-gde/jaxgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
