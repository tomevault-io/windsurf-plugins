---
trigger: always_on
description: OpenAI-compatible LLM serving stack for concurrent agent use on 4× AMD Radeon AI PRO R9700.
---

# llmstack — Claude context

OpenAI-compatible LLM serving stack for concurrent agent use on 4× AMD Radeon AI PRO R9700.
You are the expert agent responsible for this codebase. Every change you make should leave
the stack more reliable, faster, or better documented than you found it.

---

## Hardware

- **4× AMD Radeon AI PRO R9700** (gfx1201 / RDNA4), 32 GB VRAM each = **128 GB total**
- ROCm 7.2, Vulkan/RADV available
- Models stored at `/mnt/models/llm/` (read-only mount inside containers)
- Host OS: Linux, rootless podman, no sudo needed for containers

VRAM budget is the primary constraint on every model and config decision.

---

## Stack architecture

```
Claude Code / OpenCode / curl
        │
        ▼
  llmproxy :9000          ← optional SSE fix shim (for @ai-sdk clients)
        │
        ▼
  llama-swap :8080         ← router (host process, no container)
        │
        ├── vLLM container :9100+    (FP8 / AWQ / safetensors)
        └── llama-server container :9100+  (GGUF / Vulkan)
```

| Component | Port | Process | Managed by |
|-----------|------|---------|-----------|
| llama-swap | :8080 | host | `llmctl up/down` |
| llmproxy | :9000 | host | `llmctl proxy-up/down` |
| vLLM backends | :9100+ | podman container | llama-swap |
| llama-server | :9100+ | podman container | llama-swap |

Container naming: `llmstack-vllm-<profile-id>` / `llmstack-llama-<profile-id>`  
State files: `~/.local/share/llmstack/llama-swap.{pid,log}`, `llmproxy.{pid,log}`

---

## Key commands

```bash
llmctl up                        # start llama-swap router
llmctl swap qwen3.6-35b-code     # load a model (blocks until healthy)
llmctl status                    # show running state + VRAM
llmctl logs qwen3.6-35b-code     # tail container log (essential for debugging startup)
llmctl unload                    # free VRAM, keep router running
llmctl down                      # stop everything
llmpanel                         # TUI: inference metrics, GPU, model list, config, logs
```

After changing `config/models.yaml`, restart llama-swap for the change to take effect:
```bash
llmctl down && llmctl up
```

---

## vLLM cold start — critical knowledge

**Historical figures (vLLM ≤0.25.x, per-shape Inductor compilation):** first boot on a
fresh machine took 18–20 minutes — ~14 min Inductor torch.compile (4 TP ranks in
parallel, cached after the first run in `.vllm-cache/`) plus ~14 min (~800 s) of model
profiling/warmup (FP8 KV calibration), uncached, every start. `healthCheckTimeout: 1200`
and `llmctl swap` timeout `1260` s were sized for this.

**vLLM 0.26.0 changed this dramatically.** It replaced per-shape Inductor compilation
with a single dynamic-shape "compile range" (log: `Compiling a graph for compile range
(1, 32768) takes ~20s`), and the FP8 KV-calibration/profiling step dropped from ~800s to
~20-40s. Measured cold-start totals on this hardware (warm `.vllm-cache/`, 2026-07-27):

| Profile | Profiling/warmup step | Total swap time |
|---------|----------------------|-----------------|
| `qwen3.6-35b-128k-nomtp` (FP8, no MTP) | 20.98 s | 2m04s |
| `qwen3.6-35b-128k` (FP8, MTP) | — | 2m41s |
| `gemma4-26b-a4b` (BF16) | — | 3m07s |
| `qwen3.6-35b-awq` (AWQ Int4) | 37.17 s | 3m21s |

First boot is now **~2-3 minutes**, not 18-20. `healthCheckTimeout`/swap timeouts are kept
at a generous 2100 s ceiling (see `config/models.yaml`) as headroom for profiles not
covered by this measurement (256K/512K YaRN-scaled contexts, dense 27B) — treat that
number as a safety margin, not the expected wait.

**TTL must be 0 for these profiles.** llama-swap's TTL timer starts from container launch, not
from when the model becomes healthy. Even at the new ~2-3 min startup, a TTL below that would
unload the model the moment it becomes healthy. All FP8-35B profiles set `ttl: 0`.

The cache directories `.vllm-cache/` and `.triton-cache/` live in this repo root (gitignored).
They **must exist on the host** — created automatically by `mkdir -p .vllm-cache .triton-cache`
and mounted into every vLLM container. Without the mounts, every start is a cold compile.

To watch startup progress: `llmctl logs <profile>` — wait for "Application startup complete."
`healthCheckTimeout: 300` in models.yaml is set for warm starts. For first boot, wait manually.

---

## CRITICAL RULES — never break these

### vLLM profiles: mandatory flags

Every vLLM profile **must** have these three things:

```yaml
# 1. Nullify the AMD official image's broken ENTRYPOINT
--entrypoint=""
# ...then explicitly invoke:
vllm serve /models/...

# 2. Explicit CUDA graph sizes — without this, vLLM generates ~2000+ sizes = 18+ min startup
--cudagraph-capture-sizes 1 2 4 8 16 32

# 3. Persistent cache volumes — without these, torch.compile re-runs every start (~14 min)
-v __LLMSTACK_DIR__/.vllm-cache:/root/.cache/vllm
-v __LLMSTACK_DIR__/.triton-cache:/root/.triton/cache
```

`__LLMSTACK_DIR__` is a placeholder. Run `scripts/configure` to replace it with the
repo's absolute path before first use (llama-swap v223+ executes cmd without a shell).

If you add a new vLLM profile and omit any of these, the first startup will take 18+ minutes
or fail with `unrecognized arguments`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [x7even/llmctl](https://github.com/x7even/llmctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
