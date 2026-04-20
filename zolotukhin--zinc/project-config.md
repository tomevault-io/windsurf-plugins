---
trigger: always_on
description: Instructions for AI coding agents working on this repository.
---

# AGENTS.md — ZINC Inference Engine

Instructions for AI coding agents working on this repository.

## Commands

```bash
# Build (shaders compile on Linux only; macOS skips GPU inference)
zig build -Doptimize=ReleaseFast

# Run inference
ZINC_DEBUG=1 ./zig-out/bin/zinc -m model.gguf --prompt "Hello" [-d device_id] [--kv-quant 3] [--debug]
./zig-out/bin/zinc --model-id qwen3-8b-q4k-m --prompt "Hello" [--chat]

# Run unit tests
zig build test

# Compile shaders manually (requires glslc / shaderc)
glslc --target-env=vulkan1.3 -O -o out.spv src/shaders/name.comp
```

### Agent harness scripts (`loops/`)

Run-to-completion and autonomous harnesses that spawn `claude` or `codex` subagents. See file headers for full flags.

```bash
# One-shot: run a single prompt to completion with pre/post benchmarks,
# auto-retry on regressions (up to 3x) before reverting.
bun loops/guided_change.ts --prompt "..."          # inline prompt
bun loops/guided_change.ts --prompt-file plan.md   # from file
echo "..." | bun loops/guided_change.ts            # from stdin

# Autonomous multi-cycle loops (run until killed):
bun loops/optimize_zinc.ts          # rsync → build → run → agent → keep/revert on RDNA4 test node
bun loops/optimize_llm_tps.ts --agent claude "..."  # iterate on llama.cpp throughput
bun loops/optimize_perf.ts --effort N               # execute MULTI_HOUR_EFFORT_N.md
bun loops/implement_metal.ts        # iteratively build out the Metal backend
```

### Managed models and cache

For local Apple Silicon work, always prefer ZINC's managed model cache over ad hoc GGUF paths.

```bash
# Inspect the managed catalog for the current GPU profile
./zig-out/bin/zinc model list
./zig-out/bin/zinc model list --all

# Pull a managed model into the default local cache
./zig-out/bin/zinc model pull qwen3-8b-q4k-m

# Run via the managed model id instead of a handwritten GGUF path
./zig-out/bin/zinc --model-id qwen3-8b-q4k-m --prompt "What is the capital of France?" --chat
```

Default local managed cache layout:

```text
~/Library/Caches/zinc/models/models/<model-id>/model.gguf
```

Agent policy:
- On local Metal, use `./zig-out/bin/zinc model pull <id>` to fetch catalog models.
- On local Metal, prefer `--model-id <id>` or the default managed cache path over arbitrary `/Users/.../*.gguf` paths.
- Do not copy large GGUFs into one-off local locations when a catalog id exists.
- If a needed local model is missing, install it into the managed cache or explicitly explain why that is not possible.
- RDNA node runs are the exception: the benchmark node still uses the canonical `/root/models/...` GGUF paths unless the repo explicitly migrates that workflow too.

## Tech Stack

- **Zig 0.15.2+** — host code, build system
- **GLSL 460** — compute shaders compiled to SPIR-V via `glslc` (Vulkan backend)
- **MSL** — Metal Shading Language compute shaders (Apple Silicon backend)
- **Vulkan 1.3** — GPU API for AMD RDNA3/RDNA4
- **Metal** — GPU API for Apple Silicon (M1–M5)
- **GGUF** — model format (parsed natively in Zig)

### Supported Architectures
- **Qwen3 / Qwen3.5** — dense and MoE variants
- **Gemma 4** — with GeGLU activation and Gemma-specific normalization
- **OpenAI GPT-OSS** — MoE with OAI SwiGLU, MXFP4 experts, attention sinks, ISWA, YaRN RoPE

## Project Structure

```
src/
├── main.zig                     # CLI entry, arg parsing, server startup, chat subcommand
├── bench_hot_decode.zig         # Hot decode microbenchmark binary
├── bench_support.zig            # Shared helpers for Metal benchmarks
├── regression_tests.zig         # Source-level regression guards
├── zig-struct-analyzer.zig      # Zig API/doc structure extraction helper
├── compute/
│   ├── forward.zig              # Vulkan inference engine — prefill + decode loop
│   ├── forward_metal.zig        # Metal inference engine — prefill + decode loop
│   ├── dmmv.zig                 # DMMV dispatch (quantized matmul-vec)
│   ├── elementwise.zig          # Fused elementwise ops (RMS norm, SwiGLU, etc.)
│   ├── attention.zig            # Flash attention dispatch
│   ├── argmax.zig               # Argmax / sampling dispatch
│   └── graph.zig                # Decode graph builder and exporter
├── model/
│   ├── tokenizer.zig            # BPE tokenizer, chat templates, thinking toggle
│   ├── catalog.zig              # Managed model catalog with thinking_stable flag
│   ├── gguf.zig                 # GGUF file parser and tensor metadata
│   ├── loader.zig               # Model loader (Vulkan — mmap + DMA to VRAM)
│   ├── loader_metal.zig         # Model loader (Metal — zero-copy mmap)
│   ├── architecture.zig         # Architecture detection (Qwen, MoE, SSM, etc.)
│   ├── config.zig               # Model configuration from GGUF metadata
│   └── managed.zig              # Managed model download, install, activation
├── server/
│   ├── routes.zig               # OpenAI-compatible API, streaming, stop detection
│   ├── chat.html                # Built-in chat UI (embedded at compile time)
│   ├── http.zig                 # HTTP server and connection handling
│   ├── model_manager.zig        # Hot model switching and catalog view
│   ├── runtime.zig              # Backend runtime dispatch (Vulkan vs Metal)
│   └── session.zig              # Chat session state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zolotukhin/zinc](https://github.com/zolotukhin/zinc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
