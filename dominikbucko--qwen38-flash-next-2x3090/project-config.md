---
trigger: always_on
description: Read this before changing the checkpoint, serving flags, scheduler, or benchmark
---

# Field notes for agents

Read this before changing the checkpoint, serving flags, scheduler, or benchmark
scripts. The README is for users. This file records the engineering logic that
is easy to lose when looking only at the final configuration.

## Ground truth

The validated target is one Qwen3.8-Flash-Next request spread across two RTX
3090 24 GB cards, backed by 128 GB of system memory. The two GPUs do not each
hold an independent 256K context. Tensor parallelism splits one model instance
and one 262,144-token sequence across both cards.

The published checkpoint is:

```text
albucino/Qwen3.8-Flash-Next-W4A16-FP8PLE
canonical tensor revision: ef554143369a706525336f6b42a09094835dc077
```

`repro.lock.json` is authoritative for upstream revisions, the container digest,
tensor counts, and runtime versions. Do not replace a full revision or digest
with `main`, `latest`, or a floating image tag.

When a verified result changes, update the machine-readable benchmark JSON, the
generated graph, this file, and any public claim in the same commit. Do not let
an old explanation survive after its underlying flag or workload changes.

Keep these benchmark shapes separate:

| Claim | Request shape | Result |
|---|---:|---:|
| Peak prefill | 65,536 input tokens | 1,402 prompt tok/s |
| Balanced full-context prefill | 262,016 input + 128 output | 1,275.583 prompt tok/s |
| Decode after full context | 262,016 input + 128 output | 54.485 output tok/s |
| Matched short-decode endpoint | 128 input + 256 output, 10 requests | 80.08 output tok/s |
| Peak warmed long decode | 128 input + 4,096 output, one request | 135.21 output tok/s |

The 1,402 prefill and 135.21 decode figures are not from the same request. The
80.08 and 135.21 decode figures are not directly comparable either. The longer
decode amortizes one-time request overhead and gives MTP much more time to pay
for itself. Preserve this distinction in every chart, report, and regression
test.

## The model is large for a reason

Calling the checkpoint “INT4” is incomplete. The target backbone uses Intel's
AutoRound W4A16 packing, but Qwen3.8-Flash-Next also has a 51.2B-parameter PLE
n-gram table. That table was 102.4 GB in BF16. The release uses the published
RadixArk FP8 E4M3FN table and scale, which cuts it roughly in half without
inventing a new local quantization scheme.

The release payload is therefore expected to remain large:

- target checkpoint: 116.183 GiB, 222,716 tensors in 25 safetensors files;
- compact draft: 3.855 GiB, 4,639 tensors in two safetensors files;
- target backbone: Intel AutoRound W4A16, symmetric INT4 group-128;
- target layers excluded by Intel's quantization policy: original BF16;
- PLE table: published FP8 E4M3FN plus its global scale;
- MTP routed experts: local symmetric INT4 group-32;
- remaining MTP tensors: copied in their source dtype;
- KV cache: BF16.

Do not “fix” the size by quantizing everything to four bits. That would change
the quality contract. In particular, do not requantize Intel's target tensors,
discard the PLE scale, quantize the sensitive BF16 tensors, or change KV dtype
without a quality evaluation that can detect the loss.

## Why vLLM became the target

The early GGUF route was useful for proving that the model could prefill at a
reasonable rate, but it was the wrong place to optimize this architecture. The
hard parts are not a conventional dense INT4 backbone. They are PLE lookup,
hybrid recurrent state, sparse QSA, hundreds of routed experts, and MTP. The
vLLM Qwen3.8 implementation exposed the scheduler, cache lifecycle, MoE backend,
and speculative path needed to work on all of those together.

The final runtime is not stock upstream vLLM. It starts from the digest-pinned
day-zero Qwen3.8 image and installs the exact overlay under
`runtime/vllm-overlay`. `runtime/vllm-overlay/SHA256SUMS.json` prevents a silent
partial overlay. If the base image changes, assume every copied file needs a
three-way review; do not copy the old overlay onto a new vLLM revision and call
it compatible.

## Checkpoint assembly decisions

### Keep Intel packing intact

`scripts/build_intel_fp8ple_hybrid.py` reads the upstream indexes and builds a
new weight map. It does not decode and repack Intel's GPTQ/AutoRound weights.
Unchanged shards are hard-linked by default. The builder:

1. omits Intel shard 16, which contains only the BF16 PLE table;
2. omits Intel's bundled BF16 MTP tensor file;
3. inserts 128 RadixArk FP8 PLE shards plus the published scale tensor;
4. changes `ple_embedding_dtype` to `float8_e4m3fn`;
5. writes `hybrid_sources.json` and a new auditable safetensors index.

Do not infer a broken download from gaps in shard numbering. The index is the
source of truth. `scripts/validate_hybrid.py` checks every indexed tensor against
the actual safetensors headers, rejects duplicates and unindexed tensors, and
verifies that the target has PLE tensors but no bundled MTP tensors.

### Make MTP separate and small

The target does not need its bundled BF16 MTP copy. A standalone draft makes it
possible to compress draft experts aggressively while leaving target decisions
unchanged.

`scripts/build_mtp_int4.py` quantizes only the 512 routed draft experts. It

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DominikBucko/qwen38-flash-next-2x3090](https://github.com/DominikBucko/qwen38-flash-next-2x3090) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
