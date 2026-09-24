---
trigger: always_on
description: Pure-Python PyTorch implementation of the Levenshtein Transformer (Gu et al.,
---

# CLAUDE.md

## Overview

Pure-Python PyTorch implementation of the Levenshtein Transformer (Gu et al.,
NeurIPS 2019). It includes strict JSON configuration/data loading, batched
dual-policy training, Hugging Face input-embedding import, checkpoints, and
iterative inference. There is no packaging or dependency file.

## Architecture

All token tensors are seq-first `(length, batch)`. Padding masks are
`(batch, length)` with `True` meaning ignored.

`LevTModel` has one `shared_embedding` of shape
`(vocab_size, embedding_dim)`. `src_embed` and `tgt_embed` are compatibility
properties returning that same module. Independent, bias-free
`encoder_input_projection` and `decoder_input_projection` layers always map
`embedding_dim -> d_model`, including when the dimensions are equal.

The encoder uses sinusoidal, RoPE, or ALiBi position handling. Sinusoidal tables
are nonpersistent runtime buffers and support odd `d_model`; RoPE cos/sin
caches are nonpersistent and refresh for the active device/dtype. The decoder is
bidirectional and returns every layer output for deletion/placeholder early
exit. RMSNorm is used throughout; optional QK normalization and headwise or
elementwise attention output gates are supported.

Packed training (multiple unrelated examples concatenated per row) uses
segment-level attention masks so bidirectional attention never leaks across
segment boundaries: encoder/decoder self-attention get a block-diagonal mask
and decoder→encoder cross-attention gets a diagonal mask. Segment boundaries
are inferred from `[EOS][BOS]` token pairs (`levt/segment_mask.py`); unpacked
training passes `None` masks and is behaviorally identical.

### Prediction heads

- `deletion_head`: `d_model -> 2`, classes keep/delete.
- `placeholder_head`: concatenated adjacent decoder states to
  `max_placeholder + 1` counts.
- Token prediction has no `nn.Linear` output module or output parameter.
  Decoder states map back through the transpose of
  `decoder_input_projection.weight`, then logits use
  `F.linear(projected, shared_embedding.weight)`.

## Configuration ownership

`config.json` is loaded by `LevTConfig.from_json` and contains model
architecture, special IDs, and the decoding initial-sequence strategy
(`initial_strategy`: `"src"` or `"bos_eos"`). Unknown keys are rejected. Boolean switches
require actual booleans, numeric settings must be finite, and `rope_base` must
be positive. Special IDs must be distinct and in range. `embedding_dim` defaults
to `d_model` for old Python callers. Legacy training constructor attributes
remain optional only so older code can instantiate `DualPolicyTrainer`; strict
model JSON rejects them.

`train_config.json` is loaded by `TrainConfig.from_json`. It is a flat strict
schema owning:

- train/validation JSONL paths and DataLoader batch size;
- Hugging Face model source, `local_files_only`, `trust_remote_code`, import
  dtype, and embedding freezing;
- dual-policy alpha/beta, random deletion, and label smoothing;
- AdamW and Muon optimizers with warmup/linear-decay schedule;
- device, AMP, accumulation, clipping, logging, validation, checkpoints,
  resume path, early stopping, and checkpoint retention.

## Data

JSONL rows use required `src` and `target`, plus optional `initial`. Missing
`initial` defaults per `config.json`'s `initial_strategy`: `"src"` uses the full
source sequence `src` (edit-task semantics: the model edits src into target),
`"bos_eos"` uses the minimal `[BOS, EOS]` (generation from scratch); there is no
`id` field. Lists must be nonempty
integers (bool invalid), in vocabulary range and configured length limits.
Raw rows cannot contain pad/placeholder tokens. Target and initial must start
with BOS, end with EOS, and have no interior boundary tokens. Unknown keys,
blank lines, and malformed data fail with file/line context.

A dataset file may begin with an optional metadata header
`{"__meta__": {"format": "levt-jsonl", "version": 1, "packed": true}}` that
declares whether rows are packed (concatenated segments, interior BOS/EOS
boundaries allowed) or regular. The header line is skipped, not validated as a
data row. Training auto-detects packed vs regular from the header; a
header-less (legacy) file falls back to the `packed` flag in `train_config.json`
(default regular). `scripts/pack_dataset.py` (packed) and
`scripts/pre_tokenize.py` (regular) write the header on their outputs;
`scripts/precompute_oracles.py` forwards an existing header.

`LevTCollator` returns seq-first padded `src_tokens`, batch-first
`src_padding_mask`, and unpadded CPU `initial`/`targets` lists.

## Hugging Face embeddings

`levt/embeddings.py` imports Transformers lazily and calls only
`AutoModel.from_pretrained(..., torch_dtype=...).get_input_embeddings()`. No tokenizer is
loaded. Vocabulary and embedding dimensions must match exactly. Construct and
randomly initialize LevT first, then call `copy_embedding_weights`, ensuring
external weights are not reinitialized. The core model has no Transformers
dependency.

## Batched dual-policy training

`DualPolicyTrainer` accepts a `PolicyConfig`, with fallback to legacy
`LevTConfig` training attributes/defaults. For each collated batch:

1. Move each unpadded initial/target to CPU and construct insertion oracle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KrisTHL181/LevT-Nekoizer](https://github.com/KrisTHL181/LevT-Nekoizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
