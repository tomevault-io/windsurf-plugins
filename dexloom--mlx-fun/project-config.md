---
trigger: always_on
description: MLX-FUN implements REAP (Routing-based Expert Activation Pruning) for MoE models on Apple Silicon via MLX. It prunes routed experts from MoE language models based on calibration saliency data. It also supports safety-critical expert analysis (SAFEx), inference-time expert steering (SteerMoE), refusal direction removal (abliteration), domain-specific expert identification, and permanent gate amplification.
---

# CLAUDE.md — MLX-FUN Project Guide

## Project Overview

MLX-FUN implements REAP (Routing-based Expert Activation Pruning) for MoE models on Apple Silicon via MLX. It prunes routed experts from MoE language models based on calibration saliency data. It also supports safety-critical expert analysis (SAFEx), inference-time expert steering (SteerMoE), refusal direction removal (abliteration), domain-specific expert identification, and permanent gate amplification.

## Quick Reference

```bash
# Activate venv
source .venv/bin/activate

# Install (editable)
uv pip install -e ".[dev]"

# Run tests
pytest tests/ -v

# CLI entry point
mlx-fun --help
mlx-fun collect --help
mlx-fun prune --help
mlx-fun merge --help
mlx-fun smoke-test --help
mlx-fun safety-scan --help
mlx-fun steer --help
mlx-fun abliterate --help
mlx-fun domain-scan --help
mlx-fun amplify --help
mlx-fun convert-nvfp4 --help
mlx-fun ui --help
```

## Project Structure

```
src/mlx_fun/
├── adapters/              # Model-specific MoE access (BaseAdapter ABC)
│   ├── minimax.py         # MiniMax/MiniMax-M2: all layers MoE, block_sparse_moe
│   ├── glm4_moe.py       # GLM4: MoE layers >= first_k_dense_replace, mlp
│   ├── glm4_moe_lite.py  # GLM4-Lite: adds moe_layer_freq stride
│   ├── glm_moe_dsa.py    # GLM-5/DeepSeek V3.2: DeepseekV32MoE with MoEGate
│   ├── qwen3_moe.py      # Qwen3/Qwen3-Next: sparse layers by decoder_sparse_step, mlp
│   └── nemotron_h.py      # Nemotron-H: hybrid Mamba-2/Attn/MoE via hybrid_override_pattern
├── observer.py            # Hooks via __class__ swap (not MethodType — special methods)
├── ream_hooks.py          # REAM hooks: capture MoE inputs + full gate logits
├── saliency.py            # numpy float64 accumulator with np.add.at() scatter-add
├── pruner.py              # Expert selection (bottom + strided) + mx.take() tensor slicing
├── merger.py              # REAM: expert grouping, permutation alignment, weight merging
├── safety.py              # SAFEx: differential accumulator, safety report, expert classification
├── steering.py            # SteerMoE: gate logit bias injection for expert (de)activation
├── abliterate.py          # Abliteration: residual hooks, refusal direction orthogonalization
├── convert_nvfp4.py       # NVIDIA NVFP4 (modelopt) -> MLX NVFP4 checkpoint converter
├── domain.py              # Domain expert identification, amplification bias computation, gate modification
├── frontend.py            # Gradio web dashboard: chat, heatmaps, steering controls, server management
├── data.py                # JSONL + directory dataset loading with random subsampling
├── save.py                # mlx_lm.utils.save_model + reap/ream/abliteration/amplification metadata
└── cli.py                 # Click CLI: collect, prune, merge, smoke-test, serve, ui, safety-scan, steer, abliterate, domain-scan, amplify
```

## Key Design Decisions

- **Observer hooks use `__class__` swapping**, not `types.MethodType`. Python resolves `__call__` on the type, not the instance. The observer creates a dynamic subclass with the hooked `__call__` and swaps `block.__class__`.

- **Saliency uses numpy float64**, not MLX arrays. This avoids lazy evaluation issues and provides numerical stability for accumulation across many batches.

- **After capturing hook data, `mx.eval()` is called immediately** followed by `_to_numpy()` which casts bf16 to float32 before `np.array(..., copy=False)`. This materializes lazy MLX arrays before they can be garbage collected.

- **Tensor slicing uses `mx.take(tensor, keep_indices, axis=0)`** on the expert dimension for SwitchLinear weights, scales, biases, gate weights, and correction biases. Both `nn.Linear` and `nn.QuantizedLinear` gates are handled via `_slice_linear()`.

- **Dataset loading reads all samples first, then randomly subsamples** if `--max-samples` is set and the source has more. Use `--seed` for reproducibility.

- **Two pruning strategies**: `bottom` (remove lowest-scoring, standard REAP) and `strided` (split into important/unimportant groups, prune at regular intervals from both for better diversity).

- **REAM merging** (`merger.py`): Instead of pruning, merges experts around centroids using saliency-weighted averaging with neuron permutation alignment. Processes layers sequentially so merged weights feed into the next layer's similarity computation. Uses gated similarity (gate_logit * expert_output cosine similarity) for grouping.

- **Steering hooks inject bias into gate logits** before top-k selection. A pre-computed `mx.array` of shape `(num_experts,)` is added to raw gate logits — `mask_value` (-1e9) for deactivation, `boost_value` (1e4) for activation. When bias is None, the if-branch is skipped (negligible overhead).

- **Compound hooks** in `server.py` combine counting + steering in a single `__call__` to avoid hook composition issues (two `__class__` swaps on the same block would conflict).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dexloom/mlx_fun](https://github.com/dexloom/mlx_fun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
