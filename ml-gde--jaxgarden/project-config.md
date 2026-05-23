---
trigger: always_on
description: description: jaxgarden tutorial chapter on Attention Mechanism MultiHeadAttention module using customizable dot_product_attention backend (XLA/cuDNN Flash).
---

---
description: jaxgarden tutorial chapter on Attention Mechanism MultiHeadAttention module using customizable dot_product_attention backend (XLA/cuDNN Flash).
globs: 
alwaysApply: false
---
# Chapter 7: Attention Mechanism (MultiHeadAttention / dot_product_attention)

In the previous chapters, we explored end-to-end models like [LlamaForCausalLM](llamaforcausallm.mdc) and [ModernBERTForMaskedLM](modernbertformaskedlm.mdc). We saw that a key component within their transformer layers is the attention mechanism (e.g., `LlamaAttention`, `ModernBertAttention`). This chapter delves into the core implementation of attention within `jaxgarden`, specifically the `MultiHeadAttention` module and the underlying `dot_product_attention` function.

**Motivation:** Attention mechanisms, particularly scaled dot-product attention and its multi-head variant, are the computational heart of transformer models. Implementing this efficiently is critical for performance. Furthermore, modern hardware and libraries offer highly optimized kernels like Flash Attention (accessible via cuDNN in JAX) that can significantly speed up computation and reduce memory usage, especially for long sequences. `jaxgarden` aims to provide a flexible attention implementation that leverages these optimizations while integrating smoothly with the Flax NNX framework.

**Central Use Case:** Building or utilizing a transformer layer within `jaxgarden` that requires efficient multi-head self-attention. This involves instantiating `jaxgarden.attention.MultiHeadAttention` (often within a higher-level layer like `LlamaAttention`) and potentially configuring it to use an optimized backend like `"cudnn"` to enable Flash Attention on compatible hardware, thereby accelerating model training and inference.

## Key Concepts

1.  **Scaled Dot-Product Attention:** The fundamental operation defined as `Attention(Q, K, V) = softmax(QK^T / sqrt(d_k))V`. It calculates attention scores by taking the dot product between queries (Q) and keys (K), scaling them, applying a softmax function to get probabilities, and then using these probabilities to compute a weighted sum of the values (V).
2.  **Multi-Head Attention (MHA):** Instead of performing a single attention calculation, MHA projects the Q, K, and V inputs into multiple lower-dimensional "heads", performs scaled dot-product attention independently for each head in parallel, and then concatenates the results and projects them back to the original dimension. This allows the model to jointly attend to information from different representation subspaces at different positions.
3.  **`dot_product_attention` (Functional Interface):** Located in `jaxgarden.functional.attention`, this function provides the low-level implementation of scaled dot-product attention. It's a wrapper around `jax.nn.dot_product_attention`.
    *   **Core Logic:** Handles the matrix multiplications, scaling, and softmax.
    *   **Masking/Bias:** Accepts an optional `mask` or `bias` argument. It automatically converts a boolean mask (where `True` means keep) into an additive bias (0.0 for keep, -inf for mask out) suitable for the underlying JAX function.
    *   **Backend Selection:** Crucially, it accepts an `implementation` argument (`"xla"`, `"cudnn"`, `"flash"` or `None`). This allows specifying which JAX backend implementation to use for the attention computation. `"cudnn"` (aliased as `"flash"`) attempts to use optimized kernels like Flash Attention if supported by the hardware (NVIDIA GPU) and cuDNN version. `None` lets JAX choose the best available option.
4.  **`MultiHeadAttention` (NNX Module):** Located in `jaxgarden.attention.multi_head_attention`, this module provides a standard Flax NNX interface for multi-head attention.
    *   **Inheritance:** It subclasses the standard `flax.nnx.MultiHeadAttention`.
    *   **Integration:** It overrides the default `attention_fn` used by the parent Flax MHA. Instead of using Flax's default attention, it uses a customized function that internally calls `jaxgarden.functional.dot_product_attention`, passing along the desired `implementation` backend specified during the `jaxgarden.MultiHeadAttention` initialization.
    *   **Responsibilities:** Manages the multiple heads, input/output linear projections (for Q, K, V and the final output), dropout, and precision settings, encapsulating the full MHA logic.

## Using the Attention Mechanism

While `dot_product_attention` is a functional component usually called internally, `MultiHeadAttention` is the module you would typically interact with if building custom transformer layers. Higher-level models like `LlamaForCausalLM` use specialized attention modules (`LlamaAttention`) which themselves *might* use `jaxgarden.MultiHeadAttention` or implement similar logic incorporating `dot_product_attention`.

Here's how you could instantiate and use `jaxgarden.MultiHeadAttention`:

```python
import jax
import jax.numpy as jnp
import flax.nnx as nnx
from jaxgarden.attention import MultiHeadAttention # This is jaxgarden's MHA

# --- Configuration ---
batch_size = 2
seq_len = 128
hidden_dim = 256 # Also in_features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-gde/jaxgarden](https://github.com/ml-gde/jaxgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
