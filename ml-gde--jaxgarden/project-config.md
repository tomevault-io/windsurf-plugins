---
trigger: always_on
description: description: JAXgarden tutorial chapter on Rotary Position Embeddings (RoPE) for injecting relative positional information in attention.
---

---
description: JAXgarden tutorial chapter on Rotary Position Embeddings (RoPE) for injecting relative positional information in attention.
globs: 
alwaysApply: false
---
# Chapter 8: Rotary Position Embeddings (RoPE)

In the [previous chapter](attention_mechanism__multiheadattention___dot_product_attention_.mdc), we explored the core `MultiHeadAttention` mechanism and its efficient backend implementations. However, standard self-attention is permutation-invariant, meaning it doesn't inherently understand the order of tokens in a sequence. To address this, models need positional information. This chapter introduces Rotary Position Embeddings (RoPE), a clever technique for incorporating relative position information directly into the attention mechanism.

**Motivation:** Traditional methods for adding positional information include adding learned absolute position embeddings or fixed sinusoidal embeddings to the input token embeddings. While effective, these methods add extra parameters (learned embeddings) or modify the input representation before the transformer layers. RoPE offers an alternative by *rotating* the query and key vectors based on their absolute positions *before* the dot-product calculation in the attention mechanism. This rotation is designed such that the dot product between a rotated query and a rotated key naturally depends on their relative positions, effectively injecting relative positional awareness directly into the attention scores without adding separate embedding vectors.

**Central Use Case:** Applying RoPE within the attention modules of models like [LlamaForCausalLM](llamaforcausallm.mdc) (`LlamaAttention`) or [ModernBERTForMaskedLM](modernbertformaskedlm.mdc) (`ModernBertAttention`). RoPE modifies the query and key vectors just before their dot product, enabling the attention mechanism to weigh interactions based on how far apart tokens are in the sequence.

## Key Concepts

1.  **Core Idea:** RoPE operates by viewing pairs of features in the query and key vectors as complex numbers and rotating them in the complex plane. The angle of rotation depends on the token's absolute position (`m`) and the feature index (`i`). When computing the dot product between a rotated query (at position `m`) and a rotated key (at position `n`), the resulting score implicitly depends on their relative distance (`m - n`).
2.  **Mathematical Basis:** The rotation is achieved using sinusoidal functions derived from the position index (`m`) and the feature dimension (`d`). Specifically, frequencies (`theta_i`) are calculated based on the feature index `i` and a base value (`base`, often 10000 or larger). The rotation involves multiplying elements by `cos(m * theta_i)` and `sin(m * theta_i)`.
3.  **Implementation Variations:** `jaxgarden` provides two main implementations reflecting common practices:
    *   **`LlamaRotaryEmbedding`:** Used in [LlamaForCausalLM](llamaforcausallm.mdc). It calculates the `cos` and `sin` values *on-the-fly* based on the input `position_ids`. This is flexible for varying sequence lengths during generation.
    *   **`RoPEPositionalEmbedding`:** Used in [ModernBERTForMaskedLM](modernbertformaskedlm.mdc). It *pre-computes* and caches the `cos` and `sin` values for positions up to a specified `max_position_embeddings` during initialization. This can be more efficient if the maximum sequence length is known and fixed, as it replaces calculation with lookup.
4.  **Application Point:** RoPE is applied to the query and key vectors *after* their initial linear projections (`Wq`, `Wk`) but *before* the dot-product attention score calculation (`QK^T`). This modification is typically encapsulated within a helper function (e.g., `apply_rotary_pos_emb`) called by the attention module.

## Using RoPE

RoPE is generally an internal component of attention modules. You typically don't interact with `LlamaRotaryEmbedding` or `RoPEPositionalEmbedding` directly unless implementing a custom attention layer. Instead, you configure the model (via its `Config` object) which implicitly configures the RoPE within its attention layers.

**Example 1: RoPE within `LlamaAttention`**

The `LlamaAttention` module internally instantiates `LlamaRotaryEmbedding` and uses it within its `__call__` method.

```python
# Inside LlamaAttention initialization (__init__)
# config values like head_dim and rope_theta are passed
self.rotary_emb = LlamaRotaryEmbedding(
    dim=config.head_dim,
    base=config.rope_theta,
    rngs=rngs
)

# Inside LlamaAttention forward pass (__call__)
def __call__(self, x, position_ids, attention_mask):
    # ... project x to query, key, value ...
    query = self.q_proj(x).reshape(...) # [batch, n_heads, seq_len, head_dim]
    key = self.k_proj(x).reshape(...)   # [batch, n_kv_heads, seq_len, head_dim]
    value = self.v_proj(x).reshape(...) # [batch, n_kv_heads, seq_len, head_dim]

    # Calculate cos/sin on the fly (assuming batch_size=1 for simplicity here)
    cos, sin = self.rotary_emb(position_ids[0])

    # Apply RoPE rotation *before* attention calculation
    query, key = self.apply_rotary_pos_emb(query, key, cos, sin)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-gde/jaxgarden](https://github.com/ml-gde/jaxgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
