---
trigger: always_on
description: > **For AI agents and ML engineers reproducing or extending this work.**
---

# AGENTS.md — Intel Arc Pro B60/B70 Inference Cookbook

> **For AI agents and ML engineers reproducing or extending this work.**
> This file is the authoritative setup guide. Follow it top-to-bottom on a
> fresh B60/B70 host. Last updated: 2026-08-06.

## 1. What this repo does

Open recipes to run LLMs on **Intel Arc Pro B60/B70 (Battlemage, Xe2)** GPUs:

- **vLLM XPU native int4 + MTP** — 126 t/s decode / 7.5K t/s prefill on Qwen3.6-35B-A3B MoE (single-stream, one B70). Requires four in-container patches.
- **llama.cpp SYCL** — the production single-user engine and the only working dense path.
- **Benchmark harnesses** — apples-to-apples prefill × generation grids for both engines.
- **Power-sweep tooling** — find the MoE (150W) vs Dense (180W) sweet spots.

Headline result and full methodology:
[sergiiob.dev/posts/intel-arc-b70-vllm-vs-llamacpp-moe-dense-showdown/](https://sergiiob.dev/posts/intel-arc-b70-vllm-vs-llamacpp-moe-dense-showdown/)

## 2. Target hardware

- **Intel Arc Pro B70** (32 GB GDDR6, 608 GB/s, Xe2, ~$600) — primary test target.
- **Intel Arc Pro B60** (16 GB, same arch) — should work with smaller models / lower context. **Tested contributions welcome.**
- Ubuntu 24.04 / 26.04, x86_64.
- Reference rig: B70 + AMD Ryzen 7 5700X3D, 30 GB RAM, NVMe.

## 3. Host prerequisites (install once)

### 3.1 Drivers + oneAPI

The B70 needs the Intel GPU kernel driver + oneAPI runtime for both vLLM
(inside Docker) and llama.cpp (native SYCL).

```bash
# Intel GPU drivers (follow the official Intel guide for your distro):
#   https://dgpu-docs.intel.com/
# Verify the card is visible:
sudo apt install -y intel-level-zero-tools
sudo lszk  # or: lspci | grep -i vga

# oneAPI 2026.0 (for native llama.cpp SYCL builds):
#   Download from intel.com/content/www/us/en/developer/tools/oneapi/...
# Source it before any native SYCL work:
source /opt/intel/oneapi/setvars.sh --force
```

### 3.2 Docker (for vLLM)

```bash
# Standard Docker install, then verify GPU passthrough:
sudo docker run --rm --device /dev/dri --group-add $(stat -c "%g" /dev/dri/render* | head -n1) \
  -v /dev/dri:/dev/dri:ro --entrypoint bash intel/vllm:0.21.0-xpu-int4moe \
  -lc 'python -c "import torch; print(torch.xpu.device_count(), torch.xpu.get_device_name(0))"'
# Expect: 1 Intel Arc Pro B70
```

If that prints `0`, your driver/render-node permissions are wrong — fix before
proceeding. The user running Docker must be in the `render` group, or use
`--group-add $(stat -c "%g" /dev/dri/render*)`.

### 3.3 llama.cpp SYCL (native, for the dense + production path)

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
mkdir build-sycl && cd build-sycl
source /opt/intel/oneapi/setvars.sh --force
cmake .. -DGGML_SYCL=ON -DGGML_XMX=ON -DCMAKE_C_COMPILER=icx \
  -DCMAKE_CXX_COMPILER=icpx -DGGML_SYCL_TARGET_INTEL=ON \
  -DDETECT_ONEAPI_LICENSE=ON -DGGML_BACKEND_DL=ON -DLLAMA_CURL=ON
cmake --build . --config Release -j -- llama-server llama-bench
# Binary: ./bin/llama-server
```

The reference production build is `b10255+` (commit `071327508`). See
`docs/CAMPAIGN-LOG.md` for the exact flags that produced the headline numbers.

### 3.4 lmx (localmaxxing CLI) — optional, for leaderboard submission

```bash
curl -fsSLO https://github.com/LottoLottoLotto/localmaxxing-cli/releases/latest/download/lmx-linux-amd64.tar.gz
tar -xzf lmx-linux-amd64.tar.gz && sudo mv lmx /usr/local/bin/
lmx --version  # v0.1.30+ recommended
```

## 4. Reproducing the headline result (vLLM MTP, 133 t/s)

The full sequence — model download → patch → serve → bench.

### 4.1 Get the model

You need the **MTP-preserved** GPTQ checkpoint. Plain `Qwen3.6-35B-A3B-GPTQ-Int4`
has `mtp_num_hidden_layers: 1` in config but **zero MTP tensors** in the shards.
The preserved variant has the real `mtp.*` weights:

```bash
# Fastest: resolve the HF CDN URL, then aria2c -x16 (~86 MB/s)
CDN=$(curl -sI -L -o /dev/null -w '%{url_effective}' \
  'https://huggingface.co/llmfan46/Qwen3.6-35B-A3B-uncensored-heretic-Native-MTP-Preserved-GPTQ-Int4/resolve/main/config.json')
# Download all 6 shards + sidecars from:
#   https://huggingface.co/llmfan46/Qwen3.6-35B-A3B-uncensored-heretic-Native-MTP-Preserved-GPTQ-Int4
# (~22 GB total)
```

### 4.2 Serve with both patches applied

```bash
# From the repo root:
bash benchmarks/launch-mtp-bf16draft.sh /path/to/Qwen3.6-35B-A3B-MTP-Preserved-GPTQ-Int4 8000
```

**Critical flag — `--max-num-batched-tokens 8192`:** MTP caps
`max_num_scheduled_tokens` to 2048 by default, which **chunks prefill** on any
prompt >2048 tokens and costs 21-28% prefill throughput. Setting it to 8192
clears the cap (verified: p4k prefill 6,626 → 8,484 t/s; p8k → 8,718 t/s).
Without this flag you lose the long-prompt prefill advantage.

The launch script:
1. Sets speculative config `{"method":"mtp","num_speculative_tokens":1}`
2. Runs `patch_xpu_int4_moe_v4.py` + `patch_mtp_bf16_draft.py` in-container
3. Starts `vllm serve` with the right flags
4. Polls until `/v1/models` responds

Watch for `[B70] GDN XPU: spec decode active` in the logs — that's MTP running.
Full graph capture + startup takes ~3-4 min.

### 4.3 Benchmark

```bash
python3 benchmarks/b70-vllm-reddit-bench-v2.py \
  http://localhost:8000/v1/chat/completions \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SergiioB/intel-arc-pro-b70-inference-cookbook](https://github.com/SergiioB/intel-arc-pro-b70-inference-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
