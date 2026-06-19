---
trigger: always_on
description: > This file is for AI agents (and humans) working on the Abliterlitics project.
---

# AGENTS.md — Project Context & Lessons Learned

> This file is for AI agents (and humans) working on the Abliterlitics project.
> It captures hard-won knowledge about how to run evaluations correctly.

## Project Overview

Abliterlitics compares **base LLMs vs their abliterated/uncensored variants** across multiple axes:
- **Weight forensics**: Layer-by-layer weight diffs, SVD, cosine similarity
- **KL divergence**: Distribution shifts between base and modified models
- **Benchmark evaluation**: lm-evaluation-harness on 8 standard tasks
- **HarmBench**: Safety benchmark via vLLM + classifier
- **Reports**: Technical reports published to HuggingFace, condensed versions to Reddit

### Current Comparison: Qwen3.6-27B Family

6 models, all `Qwen3_5ForConditionalGeneration` architecture (~51-52GB bf16):

| Short Name | Model Path | Type |
|---|---|---|
| `base` | `models/Qwen3.6-27B/` | Base (vanilla) |
| `aeon` | `models/Qwen3.6-27B-AEON-Ultimate-Uncensored-BF16/` | Uncensored |
| `huihui` | `models/Huihui-Qwen3.6-27B-abliterated/` | Abliterated (Chinese community) |
| `heretic` | `models/Qwen3.6-27B-uncensored-heretic-v2/` | Uncensored v2 |
| `hauhau` | `models/Qwen3.6-27B-HauhauCS-Q8KP-recovered/` | Recovered variant |
| `abliterix` | `models/Qwen3.6-27B-abliterated-v2/` | Abliterated v2 |

---

## LM-Eval: The Proven Working Approach

### Architecture: vLLM OpenAI Server + `local-completions` Backend

**Why this approach?** The `vllm_causallms` direct backend OOMs with 27B BNB4 on a single 32GB GPU. Running vLLM as an OpenAI-compatible server and using lm-eval's `local-completions` backend avoids this because the server manages memory more efficiently.

**Both vLLM server AND lm-eval must run inside the SAME Docker container** — if lm-eval runs outside, the `local-completions` model init tries to download the tokenizer from HF Hub using the model name `/model` and hangs.

### Docker Images

- `abliterlitics-lmeval:1.0.0` — vLLM 0.19.0, lm-eval, proven working
- `abliterlitics-forensics:1.0.0` — Weight analysis, KL divergence

### Working Docker Command Template

```bash
docker run --rm --runtime=nvidia --shm-size=16g --ipc=host \
  --name <MODEL>-lmeval \
  -e "NVIDIA_VISIBLE_DEVICES=0" -e "CUDA_VISIBLE_DEVICES=0" \
  -v "<model_path>:/model:ro" \
  -v "models/Qwen3.6-27B:/tokenizer:ro" \
  -v "results/lm_eval:/results" \
  -v "results/lm_eval_logs:/logs" \
  -v ".cache/hf:/root/.cache/huggingface" \
  -v ".cache/vllm:/root/.cache/vllm" \
  -e HF_HOME=/root/.cache/huggingface \
  abliterlitics-lmeval:1.0.0 \
  bash -c '
    # Start vLLM server
    python3 -m vllm.entrypoints.openai.api_server \
      --model /model --dtype bfloat16 --quantization bitsandbytes \
      --load-format bitsandbytes --trust-remote-code --max-model-len 8192 \
      --gpu-memory-utilization 0.90 --enforce-eager \
      --reasoning-parser qwen3 \
      --reasoning-config '\''{"reasoning_start_str": "<think}", "reasoning_end_str": "</think"}'\'' \
      --port 8080 --host 127.0.0.1 > /logs/<model>_vllm_server.log 2>&1 &
    SERVER_PID=$!

    # Wait for ready
    for i in $(seq 1 120); do
      if curl -s http://127.0.0.1:8080/v1/models | grep -q model; then break; fi
      sleep 5
    done

    # Phase 1: Loglikelihood + truthfulqa (fast, 2048 gen tokens)
    python3 -m lm_eval --model local-completions \
      --model_args "base_url=http://127.0.0.1:8080/v1/completions,model=/model,tokenizer=/tokenizer,tokenizer_backend=huggingface,num_concurrent=4,max_length=8192" \
      --tasks "mmlu,hellaswag,arc_challenge,winogrande,truthfulqa,piqa,lambada_openai" \
      --batch_size 4 --gen_kwargs max_gen_toks=2048 \
      --output_path /results/ --log_samples 2>&1 | tee /logs/<model>_phase1.log

    # Phase 2: GSM8K only (needs high gen tokens for reasoning models)
    python3 -m lm_eval --model local-completions \
      --model_args "base_url=http://127.0.0.1:8080/v1/completions,model=/model,tokenizer=/tokenizer,tokenizer_backend=huggingface,num_concurrent=4,max_length=8192" \
      --tasks "gsm8k" \
      --batch_size 4 --gen_kwargs max_gen_toks=7168 \
      --output_path /results/ --log_samples 2>&1 | tee /logs/<model>_gsm8k.log

    kill $SERVER_PID 2>/dev/null
  '
```

---

## Critical Lessons Learned

### 1. `max_gen_toks` includes thinking tokens — plan accordingly

**The mistake:** Set `max_gen_toks=2048` thinking that was the "response budget". But for reasoning models (Qwen3.5 with `<think/>` tags), `max_gen_toks` is the TOTAL generation budget including thinking tokens. The model would think for 1900 tokens, get cut off, and never produce an answer.

**The fix:** GSM8K needs `max_gen_toks=7168` to give the model room for extended reasoning (~5000 thinking tokens) + answer (~2048 response tokens). But this is ONLY needed for generative tasks (gsm8k). Loglikelihood tasks (mmlu, hellaswag, arc, winogrande, piqa, lambada) ignore `max_gen_toks` entirely.

**Never set `max_gen_toks` higher than `max_model_len - max_prompt_length`.** With `max_model_len=8192` and prompts up to ~1024 tokens, `max_gen_toks=7168` is the safe maximum. Setting 8092 caused context overflow crashes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dreamfast/abliterlitics](https://github.com/dreamfast/abliterlitics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
