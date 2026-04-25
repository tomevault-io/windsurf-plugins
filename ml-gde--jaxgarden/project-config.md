---
trigger: always_on
description: description: Details the jaxgarden LlamaForCausalLM model for causal language modeling, covering architecture, components, and HF weight conversion.
---

---
description: Details the jaxgarden LlamaForCausalLM model for causal language modeling, covering architecture, components, and HF weight conversion.
globs: jaxgarden/models/llama.py
alwaysApply: false
---
# Chapter 4: LlamaForCausalLM

In the [previous chapter](baseconfig.mdc), we examined `BaseConfig`, the foundation for configuring models in `jaxgarden`. We saw how model-specific configurations like `LlamaConfig` inherit from it to define hyperparameters. Now, we dive into the complete model implementation that uses this configuration: `LlamaForCausalLM`.

**Motivation:** Large language models like Meta's Llama have demonstrated remarkable capabilities in natural language understanding and generation. Implementing such complex architectures efficiently within the JAX ecosystem requires careful integration of various components (attention mechanisms, normalization layers, embeddings) and adherence to best practices for performance and state management. `LlamaForCausalLM` provides a faithful and optimized implementation of the Llama architecture, ready for training and inference using JAX and Flax NNX.

**Central Use Case:** Loading pretrained Llama weights (e.g., from Hugging Face Hub) into a `jaxgarden.LlamaForCausalLM` instance and using it for autoregressive text generation. This involves initializing the model with the correct `LlamaConfig`, leveraging the `from_hf` method inherited from [BaseModel](basemodel.mdc) for weight conversion, and then using the `generate` method from the [GenerationMixin](generationmixin.mdc) for text synthesis.

## Key Concepts

`LlamaForCausalLM` integrates several advanced transformer components into a cohesive causal language model:

1.  **Core Structure:** It inherits from [BaseModel](basemodel.mdc) for configuration, state management, and HF integration capabilities, and from [GenerationMixin](generationmixin.mdc) for text generation methods.
2.  **Token Embeddings (`nnx.Embed`):** Maps input token IDs from a vocabulary to dense vector representations (hidden states).
3.  **`LlamaTransformerBlock`:** The main building block, repeated multiple times (`n_layers` specified in `LlamaConfig`). Each block contains:
    *   **`LlamaRMSNorm` (Pre-Normalization):** Applied before the attention and MLP layers for improved training stability. RMSNorm is a simpler and often faster alternative to LayerNorm.
    *   **`LlamaAttention`:** Implements multi-head self-attention. It incorporates [Rotary Position Embeddings (RoPE)](rotary_position_embeddings__rope_.mdc) via `LlamaRotaryEmbedding` to inject positional information dynamically. It also supports Grouped Query Attention (GQA) where the number of key/value heads (`n_kv_heads`) can be smaller than the number of query heads (`n_heads`) for reduced computational cost and memory footprint, particularly during inference.
    *   **`LlamaMLP`:** A feed-forward network using the SwiGLU activation function (`silu(gate(x)) * up(x)`), which has shown strong performance in recent models.
4.  **Final Normalization & LM Head:** After the last `LlamaTransformerBlock`, a final `LlamaRMSNorm` is applied. The output is then passed through a linear layer (`lm_head`) that projects the final hidden state back to the vocabulary size, producing logits for the next token prediction.
5.  **Weight Tying:** The weights of the `lm_head` are typically tied to the weights of the `token_embed` layer. This reduces the total number of parameters and can improve performance. This tying is handled during weight initialization and conversion.
6.  **HF Weight Conversion (`convert_weights_from_hf`):** Implements the logic to map parameter names and shapes from Hugging Face Llama checkpoints (stored in Safetensors format) to the specific structure and naming convention of the `jaxgarden` `LlamaForCausalLM` state.
7.  **Text Generation:** Inherits the `generate` method from [GenerationMixin](generationmixin.mdc), enabling autoregressive text generation with sampling strategies like temperature, top-k, and top-p.

## Using `LlamaForCausalLM`

Let's see how to initialize, load weights, and use the model.

### Initialization

First, define the configuration (`LlamaConfig`) and instantiate the model using `nnx.Rngs`.

```python
import jax
import jax.numpy as jnp
from flax import nnx
from jaxgarden.models.llama import LlamaConfig, LlamaForCausalLM

# Example: Configuration for a small Llama model
config = LlamaConfig(
    dim=512,
    n_layers=4,
    n_heads=8,
    n_kv_heads=4, # GQA enabled (n_kv_heads < n_heads)
    head_dim=64,
    intermediate_size=1024,
    vocab_size=10000, # Example vocabulary size
    norm_eps=1e-5,
    rope_theta=10000.0
)

# Initialize PRNG keys
rngs = nnx.Rngs(params=0) # Or use more sophisticated key management

# Instantiate the model
model = LlamaForCausalLM(config, rngs=rngs, param_dtype=jnp.bfloat16)

print(f"Initialized LlamaForCausalLM with {config.n_layers} layers.")
# Output: Initialized LlamaForCausalLM with 4 layers.
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ml-gde) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
