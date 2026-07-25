---
trigger: always_on
description: Guidance for Claude Code (and other AI assistants) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repository.

EdgeRunner is a from-scratch LLM inference engine for Apple Silicon, written in Swift 6.2 with custom Metal compute kernels. It targets fast on-device decode of quantized GGUF models (primarily Qwen3-0.6B Q8_0). See `AGENTS.md` for the autoresearch optimization loop — that file is the source of truth for performance-tuning work and takes precedence over this file when they disagree on benchmarking.

## Repository Layout

```
Package.swift              # SwiftPM manifest — do NOT modify dependencies
README.md                  # Public-facing overview and quick start
AGENTS.md                  # Autoresearch / optimization loop rules (PRECEDENCE for perf work)
TROUBLESHOOTING.md         # Common build / load / runtime issues
CLAUDE.md                  # This file
docs/
  ROADMAP.md               # Phased roadmap and perf context
  EdgeRunner-Framework-Deep-Documentation.md
  arch/                    # Architecture references (pipeline, kernels, API, memory)
Sources/
  EdgeRunnerSharedTypes/   # C header shims shared with Metal shaders
  EdgeRunnerMetal/         # Metal kernels + wrappers (GPU hot path)
    Shaders/               # *.metal source files (compiled as package resource)
  EdgeRunnerIO/            # Model loading: GGUF, SafeTensors, NPY/NPZ, dequant kernels
    GGUF/                  # GGUF parser + memory-mapped file
    Protocols/             # LoadableModel protocol
  EdgeRunnerCore/          # Tensors, sampling, tokenization, graph, structured generation
    Sampling/              # Greedy / temperature / top-k / top-p / min-p / rep-penalty
    Tokenizer/             # BPE, SentencePiece, chat templates, pre-tokenizer
    Graph/                 # ComputeGraph + FusionEngine + TensorOp
    Generation/            # SpeculativeDecoder
    StructuredGeneration/  # JSON schema / grammar-constrained decoding
  EdgeRunner/              # Public façade + high-level API
    Models/                # LlamaLanguageModel (primary), GPT2*
    Transformer/           # Generic transformer block scaffolding
    Module/                # nn-module-style wrappers (Linear, Sequential, TensorBox)
    Backends/              # Backend factory, local backend, Foundation Models backend
    Streaming/             # TokenStream + GenerationSession
    ToolCalling/           # Tool protocol, parser, executor, tool choice
    Chat/                  # ChatMessage, ChatViewModelState, ModelInfo
    Metrics/               # Perplexity
    Documentation.docc/    # DocC catalog
  ANEInteropIO/            # C code for ANE / IOSurface interop
  EspressoEdgeRunner/      # Experimental ANE/Espresso backend (weight conversion, RoPE bridge)
Tests/
  EdgeRunnerMetalTests/    # Kernel-level tests + KV cache / memory benchmarks
  EdgeRunnerIOTests/       # Loader / dequant tests
  EdgeRunnerCoreTests/     # Sampling, tokenizer, graph, structured generation
  EdgeRunnerTests/         # Integration, parity, publishable/framework benchmarks
  EspressoEdgeRunnerTests/
Examples/
  EdgeRunnerChat/          # SwiftUI sample chat app
benchmarks/
  pinned_qwen3_0.6b_q8_0.json  # Canonical contract — source of truth
  experiment_log.md             # Append-only experiment history
  baseline.json
  run_long_prompt_framework_benchmark.py
```

## Module Dependency Graph

Layered from bottom up (each depends only on layers below):

```
EdgeRunnerSharedTypes  (C headers — scalar type defs shared with Metal)
        │
EdgeRunnerMetal        (Metal kernels, buffer cache, residency, KV cache)
        │
EdgeRunnerIO           (GGUF/SafeTensors loaders, dequantization kernels)
        │
EdgeRunnerCore         (Tensors, sampling, tokenizers, graph, structured gen)
        │
EdgeRunner             (Public façade: EdgeRunner actor, LlamaLanguageModel, streaming)
```

`EspressoEdgeRunner` is a parallel experimental target layered on `EdgeRunnerIO + EdgeRunnerMetal + ANEInteropIO`. `ANEInteropIO` is C-only with `IOSurface` linkage.

## Platform & Toolchain

- Swift tools 6.2, platforms: `.iOS(.v26)`, `.macOS(.v26)`
- Requires Apple Silicon (M1+), Xcode 26 beta or newer
- `@_exported import` surface lives in `Sources/EdgeRunner/EdgeRunner.swift` (re-exports Core, IO, Metal, SharedTypes)

## Key Entry Points

| Type | File | Purpose |
|---|---|---|
| `EdgeRunner` (actor) | `Sources/EdgeRunner/EdgeRunnerFacade.swift` | High-level public API — `init(modelPath:)`, `stream(_:)`, `generate(_:)` |
| `EdgeRunnerLanguageModel` (protocol) | `Sources/EdgeRunner/EdgeRunnerLanguageModel.swift` | Contract for model implementations |
| `LlamaLanguageModel` | `Sources/EdgeRunner/Models/LlamaLanguageModel.swift` | **Primary inference engine** — hot path for all optimization work |
| `ModelLoader` | `Sources/EdgeRunner/ModelLoader.swift` | Dispatches GGUF/SafeTensors loading |
| `GGUFLoader` / `GGUFParser` | `Sources/EdgeRunnerIO/GGUF/` | Memory-mapped GGUF parsing |
| `MetalBackend` | `Sources/EdgeRunnerMetal/MetalBackend.swift` | Device / command queue / shader library |
| `KVCache` | `Sources/EdgeRunnerMetal/KVCache.swift` | GPU-resident K/V cache for incremental decode |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christopherkarani/EdgeRunner](https://github.com/christopherkarani/EdgeRunner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
