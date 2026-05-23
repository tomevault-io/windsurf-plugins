---
trigger: always_on
description: description: Details the jaxgarden ModernBERTForMaskedLM model, a bidirectional encoder with RoPE, pre-LN, and mixed attention for MLM tasks.
---

---
description: Details the jaxgarden ModernBERTForMaskedLM model, a bidirectional encoder with RoPE, pre-LN, and mixed attention for MLM tasks.
globs: jaxgarden/models/modernbert.py
alwaysApply: false
---
# Chapter 6: ModernBERTForMaskedLM

In the [previous chapter](generationmixin.mdc), we explored `GenerationMixin`, focusing on autoregressive text generation for causal language models. Now, we shift our focus to a different type of transformer architecture: a bidirectional encoder designed specifically for Masked Language Modeling (MLM) tasks, incorporating modern optimizations. Welcome to `ModernBERTForMaskedLM`.

**Motivation:** While causal models excel at text generation, many NLP tasks benefit from understanding context bidirectionally (both left and right). Traditional BERT achieved this but often suffered from computational inefficiency, especially with long sequences. The ModernBERT architecture, as proposed by Answer.AI, aims to create a "Smarter, Better, Faster, Longer" bidirectional encoder by incorporating techniques like Rotary Position Embeddings (RoPE), pre-Layer Normalization, and efficient attention mechanisms, making it suitable for fast, memory-efficient training and inference, particularly for long contexts. `ModernBERTForMaskedLM` implements this architecture within `jaxgarden` for the MLM pre-training objective.

**Central Use Case:** Pre-training or fine-tuning a language model using the masked language modeling objective, where the model predicts randomly masked tokens in the input sequence. This model can also serve as a powerful feature extractor for downstream NLP tasks requiring bidirectional context understanding, potentially after loading pre-trained weights using the framework provided by [BaseModel](basemodel.mdc).

## Key Concepts

`ModernBERTForMaskedLM` integrates several components and modern architectural choices:

1.  **Inheritance:** It inherits from [BaseModel](basemodel.mdc), gaining standardized configuration management, state handling (Flax NNX), checkpointing (`save`/`load`), and the interface for Hugging Face weight conversion (`from_hf`).
2.  **`ModernBertEmbeddings`:** Handles the initial input processing. It converts token IDs into dense vectors using an embedding layer and applies Layer Normalization. Crucially, unlike traditional BERT, it does *not* include explicit learned position embeddings; positional information is injected later via RoPE.
3.  **`ModernBERTEncoder`:** The core of the model, consisting of a stack of `ModernBertLayer` instances (`num_hidden_layers` defined in `ModernBERTConfig`).
4.  **`ModernBertLayer`:** Implements a single transformer block using the **Pre-LayerNorm** architecture (LayerNorm applied *before* the attention and MLP sub-layers, followed by residual connections). This structure often leads to more stable training compared to the original Post-LayerNorm BERT. Each layer contains:
    *   `ModernBertAttention`: Performs multi-head self-attention.
    *   `ModernBertMLP`: A feed-forward network applied after attention.
5.  **`ModernBertAttention`:**
    *   Calculates query, key, and value projections from the input.
    *   Applies **[Rotary Position Embeddings (RoPE)](rotary_position_embeddings__rope_.mdc)** directly to the query and key vectors before computing attention scores. `RoPEPositionalEmbedding` generates the necessary sinusoidal encodings.
    *   Supports **Mixed Global and Local Attention:** Can operate in standard global attention mode (all tokens attend to all others) or use a sliding window (local attention) where tokens only attend to nearby tokens (`local_attention` parameter). This can be configured to happen only on certain layers (`global_attn_every_n_layers`) to balance global context understanding with computational efficiency.
6.  **`ModernBertMLP`:** Uses GELU activation function within the feed-forward network.
7.  **`ModernBERTMLMHead`:** Added on top of the `ModernBERTEncoder`. It takes the final hidden states, applies Layer Normalization, a dense layer with GELU activation, and finally projects the result to the vocabulary size to produce logits for predicting the original masked tokens. The final projection layer (decoder) is often tied to the token embedding weights.

## Using `ModernBERTForMaskedLM`

Let's see how to instantiate and use the model.

### Initialization

Define a `ModernBERTConfig` and initialize the model. Key parameters in the config control the architecture, including attention behavior.

```python
import jax
import jax.numpy as jnp
from flax import nnx
from jaxgarden.models.modernbert import ModernBERTConfig, ModernBERTForMaskedLM

# Configuration for a smaller ModernBERT model
config = ModernBERTConfig(
    vocab_size=30522, # Example BERT vocab size
    hidden_size=256,
    num_hidden_layers=4,
    num_attention_heads=8,
    intermediate_size=512,
    max_position_embeddings=512, # Max sequence length for RoPE cache
    attention_dropout=0.1,
    hidden_dropout=0.1,
    # Use local attention (window size 128 left, 128 right)
    local_attention=(128, 128),
    # Apply global attention every 2 layers (layers 0, 2)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-gde/jaxgarden](https://github.com/ml-gde/jaxgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
