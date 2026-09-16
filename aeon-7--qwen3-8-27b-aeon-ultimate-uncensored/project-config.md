---
trigger: always_on
description: **Read this file before changing anything in this repo or its container.**
---

# AGENTS.md - Operator's Manual for AI Agents (Qwen3.8 MIXED)

**Read this file before changing anything in this repo or its container.**

You are an AI coding agent working with this repository, its container images, or the model it serves. Public docs, blog posts, and even Qwen3.6 recipes in sibling repos are **stale for this stack**. Qwen3.8 **NVFP4-MIXED** has different quantization, attention, speculative, and util locks than Qwen3.6 compressed-tensors / flash_attn recipes.

This file is authoritative for **Qwen3.8-27B-AEON-ULTIMATE-UNCENSORED-NVFP4-MIXED**. If public docs contradict it, **trust this file**. Canonical docker blocks also live on the [HF MIXED card](https://huggingface.co/AEON-7/Qwen3.8-27B-AEON-ULTIMATE-UNCENSORED-NVFP4-MIXED).

---

## ⚠️ Hardware scope

**Root compose + most of this file target DGX Spark / GB10 / sm_121a (UMA).** RTX seats live under `other-hardware/` with different images and speculative paths.

| You're on | Recipe location | Why Spark rules don't apply wholesale |
|---|---|---|
| **1x DGX Spark** (GB10, sm_121a) | [`docker-compose.yml`](docker-compose.yml) | This file's defaults |
| **2x Spark TP=2** | [`docker-compose.tp2-rank1.yml`](docker-compose.tp2-rank1.yml) -> [`docker-compose.tp2-rank0.yml`](docker-compose.tp2-rank0.yml) | Fixed DFlash2 n=7 + YaRN; MRv2 **off**; B0 allreduce knobs |
| **RTX 5090** (sm_120, 32 GB) | [`other-hardware/rtx5090/`](other-hardware/rtx5090/) | RTX image; **MTP n=3**; no DFlash; util 0.92 |
| **RTX PRO 6000** (sm_120, 96 GB) | [`other-hardware/rtx6000pro/`](other-hardware/rtx6000pro/) | Same RTX image; MTP n=3; util 0.80; **validated for Qwen3.8** |
| **BF16 / H200 teacher** | HF BF16 card only | Not day-to-day serve - no compose in this repo |

Do **not** apply Spark util/DFlash/image advice to RTX, or RTX MTP advice to Spark.

---

## TL;DR for agents (60 seconds)

| Thing | Value | Don't second-guess |
|---|---|---|
| **Body** | `AEON-7/Qwen3.8-27B-AEON-ULTIMATE-UNCENSORED-NVFP4-MIXED` (~23.8G) | Not the Qwen3.6 tree; not compressed-tensors-only dumps |
| **Spark image** | `ghcr.io/aeon-7/aeon-vllm-ultimate:2026-09-11-v0.29.0-omni` (digest `sha256:2421bb...`) | Pin dated until `:latest` matches; rollback `:2026-09-07-reasoning-eos`. ENTRYPOINT is bash -> compose sets `entrypoint: vllm` |
| **RTX image** | `ghcr.io/aeon-7/aeon-vllm-ultimate-rtx:latest` | Do **not** cross Spark ↔ RTX images |
| **Quantization** | **Leave `--quantization` UNSET** | `hf_quant_config.json` -> `modelopt_mixed`. Never `compressed-tensors` / `nvfp4` / `modelopt` / `modelopt_fp4` on this tree |
| **Attention** | **`--attention-backend TRITON_ATTN`** | Never `flash_attn` on these MIXED recipes |
| **Prefix cache** | **`--no-enable-prefix-caching`** | Always off on published MIXED seats |
| **Spark spec** | **Dynamic DFlash lattice** + `z-lab/Qwen3.8-27B-DFlash2` | Exact map below; peak **237.67** tok/s Coding@c16 |
| **TP=2 spec** | Fixed **DFlash2 n=7** | Not the lattice |
| **RTX spec** | **MTP n=3** | No DFlash on 5090 |
| **Spark util** | **0.80** default | Downshift if sidecars; optional **0.85** dedicated-only; beyond 0.80 -> UMA OOM risk |
| **#54367** | Bind `patches/modelopt-54367.py` on Spark 0.29 | **Required** for MIXED on `2026-09-11-v0.29.0-omni` |
| **Gen defaults** | temp 0.6, top_p 0.95, top_k 20, **`repetition_penalty` 1.0** | **>1.0 breaks `/parameter` / XML tool parsers** |
| **Served name** | `aeon` (alias `aeon-ultimate` OK) | |
| **Chat** | `reasoning-parser qwen3`, `tool-call-parser qwen3_coder`, `enable-auto-tool-choice` | |

Compose files are the **source of truth**. Don't invent flags.

### Dynamic DFlash lattice (1x Spark - EXACT)

```json
{"method":"dflash","model":"/draft","num_speculative_tokens":10,"num_speculative_tokens_per_batch_size":[[1,1,10],[2,2,10],[3,4,8],[5,8,7],[9,10,6],[11,12,5],[13,14,4],[15,16,3]],"attention_backend":"TRITON_ATTN"}
```

Runtime K: c1-2->10, c3-4->8, c5-8->7, c9-10->6, c11-12->5, c13-14->4, c15-16->3.

---

## Pick-the-compose guide

| Goal | File(s) | Notes |
|---|---|---|
| Production 1x Spark (quality + throughput) | `docker-compose.yml` | Lattice, util 0.80, 262k, MRv2 on, FULL_AND_PIECEWISE |
| Dual Spark 1M quality | `docker-compose.tp2-rank1.yml` **then** `docker-compose.tp2-rank0.yml` | Rank 1 headless first; YaRN factor-4; util 0.70 |
| Dual Spark 64k speed | Same TP2 files with deltas in comments | Drop YaRN / long-len env; max-model-len 65536; util 0.60 |
| RTX 5090 chat | `other-hardware/rtx5090/docker-compose.yml` | MTP n=3; util 0.92 |
| RTX PRO 6000 | `other-hardware/rtx6000pro/docker-compose.yml` | MTP n=3; util 0.80; validated Qwen3.8 |

---

## DO NOT UNDO - critical traps (Qwen3.8 MIXED)

### 1. Don't set `--quantization`

```
# WRONG for MIXED:
--quantization compressed-tensors
--quantization nvfp4
--quantization modelopt
--quantization modelopt_fp4
```

**Why:** MIXED needs `hf_quant_config.json` to select **`modelopt_mixed`**. Passing an explicit quant flag forces the wrong loader. Leave it **unset**.

### 2. Don't use `flash_attn` - must `TRITON_ATTN`

```
# WRONG (stale Qwen3.6 Spark advice):
--attention-backend flash_attn
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AEON-7/Qwen3.8-27B-AEON-ULTIMATE-UNCENSORED](https://github.com/AEON-7/Qwen3.8-27B-AEON-ULTIMATE-UNCENSORED) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
