---
trigger: always_on
description: description: jaxgarden tutorial on GenerationMixin, providing autoregressive text generation with sampling for JAX models.
---

---
description: jaxgarden tutorial on GenerationMixin, providing autoregressive text generation with sampling for JAX models.
globs: jaxgarden/models/generation_utils.py
alwaysApply: false
---
# Chapter 5: GenerationMixin

In the [previous chapter](llamaforcausallm.mdc), we explored `LlamaForCausalLM`, a causal language model built using `jaxgarden` components. We saw that it inherits text generation capabilities. This chapter focuses on the `GenerationMixin` class, the source of that functionality.

**Motivation:** Implementing autoregressive text generation involves complex logic: managing the token-by-token loop efficiently, handling various sampling strategies (temperature, top-k, top-p, min-p), managing padding and end-of-sequence tokens, and dealing with JAX's PRNG key management and JIT compilation nuances. Encapsulating this logic in a reusable mixin avoids code duplication across different causal language models and provides a standardized generation interface. `GenerationMixin` solves this by providing a robust `generate` method that can be easily added to any compatible causal LM inheriting from [BaseModel](basemodel.mdc).

**Central Use Case:** Adding the `generate` method to a custom causal language model (or using it via an existing model like `LlamaForCausalLM`). This allows the model instance to perform text generation based on a prompt, controlling the output's creativity and coherence through sampling parameters, and leveraging JAX optimizations like `lax.scan` and JIT compilation for performance on accelerators.

## Key Concepts

1.  **Mixin Pattern:** `GenerationMixin` is not meant to be used standalone. It's designed to be inherited *alongside* a primary base class (like `BaseModel` or a specific model class like `LlamaForCausalLM`). It "mixes in" the `generate` method and its helpers into the inheriting class.
2.  **Autoregressive Loop (`jax.lax.scan`):** Text generation is sequential. The model predicts the next token based on the previously generated tokens. `GenerationMixin` implements this loop using `jax.lax.scan`, which is highly efficient for iterative computations on JAX accelerators (GPU/TPU) as it unrolls the loop within the compiled computation graph.
3.  **Sampling Strategies:** Controls how the next token is chosen from the model's output probability distribution (logits).
    *   `temperature`: Scales logits before sampling. Higher values -> more randomness; lower values -> more determinism.
    *   `top_k`: Restricts sampling to the `k` most likely tokens.
    *   `top_p` (Nucleus Sampling): Restricts sampling to the smallest set of tokens whose cumulative probability exceeds `p`.
    *   `min_p`: Restricts sampling to tokens with probability `p * max_probability` or higher.
    *   `do_sample`: Boolean flag to enable/disable sampling (if `False`, uses greedy decoding - picks the most likely token).
4.  **Helper Functions:** Sampling strategies are implemented via standalone helper functions (`temperature_scale`, `top_k_logits`, `top_p_logits`, `min_p_logits`, `sample_logits`) for clarity and testability.
5.  **State Management:** The generation loop manages the sequence length, detects the End-of-Sequence (EOS) token to stop generation for specific sequences in a batch, handles padding (`pad_token_id`), and correctly splits and passes the JAX PRNG key (`rng`) at each step if sampling is enabled.
6.  **JIT Compilation (`use_jit`):** The `generate` method offers a `use_jit` flag. If `True`, it calls a pre-compiled version of the core generation loop (`_generate_compiled`). This requires specifying `static_argnames` (like `max_length`, `temperature`, `top_k`, etc., and crucially `self`) to `jax.jit`, as these values influence the computation graph structure and cannot be dynamic JAX tracers during compilation.

## Using `GenerationMixin`

You typically don't interact with `GenerationMixin` directly. Instead, you call the `generate` method on a model class that inherits from it, like `LlamaForCausalLM`.

```python
import jax
import jax.numpy as jnp
from flax import nnx
# Assume LlamaForCausalLM and Tokenizer are imported and initialized
# from jaxgarden.models.llama import LlamaConfig, LlamaForCausalLM
# from jaxgarden.tokenization import Tokenizer

# --- Setup (Conceptual) ---
# model_config = LlamaConfig(...) # Load appropriate config
# tokenizer = Tokenizer.from_pretrained(...) # Load matching tokenizer
# rngs = nnx.Rngs(0)
# model = LlamaForCausalLM(model_config, rngs=rngs)
# model.from_hf(...) # Optional: Load pretrained weights

# --- Generation Call ---
# prompt = "The definition of JAX is"
# inputs = tokenizer.encode(prompt, return_tensors="jax")
# input_ids = inputs['input_ids']
# attention_mask = inputs['attention_mask'] # Important for initial prompt

# Set generation parameters
# max_new_tokens = 50
# target_max_length = input_ids.shape[1] + max_new_tokens
# generation_rng = jax.random.PRNGKey(42)
# pad_token_id = tokenizer.pad_token_id
# eos_token_id = tokenizer.eos_token_id

# print(f"Generating text with max_length={target_max_length}, temperature=0.8, top_k=50")

# --- The core call to the generate method ---

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-gde/jaxgarden](https://github.com/ml-gde/jaxgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
