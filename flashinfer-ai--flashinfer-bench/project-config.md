---
trigger: always_on
description: This document gives agents repo-level context for working in `flashinfer-bench`.
---

# FlashInfer-Bench Repo Guide

This document gives agents repo-level context for working in `flashinfer-bench`.
Use it for repository structure, trace dataset conventions, and source-of-truth guidance.
Task-specific procedures live in `.claude/skills/`.

## Project Overview

FlashInfer-Bench is a GPU kernel optimization benchmarking framework for:
- Standardizing FlashInfer Trace format
- Real-time workload tracing and collection
- Automated kernel optimization and replacement
- Performance leaderboards and tracking

### Core Concepts

1. **Definition**: Specifies an operation's interface (inputs/outputs, axes, reference implementation)
2. **Solution**: Concrete implementation of a Definition (Python/Triton/CUDA)
3. **Workload**: Specific input configuration and test case
4. **Trace**: Execution record containing correctness and performance data
5. **Model**: Hierarchical module structure mapping model components to Definitions

## Repository Structure

```
flashinfer-bench/
├── flashinfer_bench/           # Main Python package
│   ├── data/                   #   Definition, Solution, Workload, Trace data classes
│   ├── bench/                  #   Benchmarking engine + evaluators
│   ├── compile/                #   Build system (Python/Triton/CUDA)
│   ├── apply/                  #   Kernel auto-replacement API
│   ├── serve/                  #   Benchmark orchestration service (NOT inference)
│   ├── integration/            #   FlashInfer integration
│   ├── tracing/                #   Workload tracing utilities
│   └── agents/                 #   Agent orchestration tools
├── tests/                      # Pytest test suite
├── scripts/                    # Standalone scripts (workload collection, sanitization)
├── tools/                      # Developer tools (GPU locking, etc.)
├── docs/                       # Documentation (model coverage, op_type schemas)
├── web/                        # Web UI for visualization
├── examples/                   # Example code and benchmarks
├── .claude/skills/             # Agent skill definitions (see below)
└── tmp/                        # Cloned external repos, including the
                                #   flashinfer-trace HF dataset clone
                                #   (SGLang, FlashInfer, sgl-cookbook, flashinfer-trace)
```

## Trace Dataset

### Single source of truth: HuggingFace

The canonical (and only) trace dataset lives at
[`flashinfer-ai/flashinfer-trace`](https://huggingface.co/datasets/flashinfer-ai/flashinfer-trace)
on HuggingFace. It contains definitions, reference tests, baseline solutions, workloads,
blobs, and evaluation traces.

There is **no** in-repo `flashinfer_trace/` directory in `flashinfer-bench`. The earlier
"internal trace layer" was removed in the trace-dataset refactor (PR #418); definitions,
reference tests, and workloads no longer live here. Skills that need to read or edit trace
content do so through a local clone of the HF dataset at `tmp/flashinfer-trace/`:

```
tmp/flashinfer-trace/                  # local clone of the HuggingFace dataset
├── definitions/{op_type}/{definition_name}.json
├── tests/references/test_{definition_name}.py
├── solutions/baseline/{op_type}/{definition_name}/...
├── workloads/{op_type}/{definition_name}.jsonl
├── blob/workloads/{op_type}/{definition_name}/*.safetensors
└── traces/{op_type}/{definition_name}.jsonl
```

Browse `tmp/flashinfer-trace/definitions/` to see the current set of supported op_types
once `/clone-repos` has been run.

### Lifecycle

1. Run `/clone-repos` to ensure `tmp/flashinfer-trace/` is checked out and up to date.
2. Generate or update content under `tmp/flashinfer-trace/` and commit on a feature branch.
3. Open a PR against the HuggingFace dataset repo (PR 2 in the onboard-model flow).
4. Open a companion PR against `flashinfer-bench` that updates **only** `docs/model_coverage.mdx`
   to reflect the new coverage (PR 1 in the onboard-model flow).

The HuggingFace dataset is the primary edit surface — flashinfer-bench owns code, docs,
and the coverage doc, not the trace data itself.

### Definition JSON Structure

Each definition JSON follows a common structure:

```json
{
  "name": "...",
  "description": "...",
  "op_type": "...",
  "tags": ["stage:decode", "status:verified", "model:...", "fi_api:...", "tp:N"],
  "axes": { "batch_size": {"type": "var"}, "num_heads": {"type": "const", "value": 16} },
  "constraints": ["len_indptr == batch_size + 1"],
  "inputs": { "tensor_name": {"shape": ["axis1", "axis2"], "dtype": "bfloat16"} },
  "outputs": { "output": {"shape": ["axis1", "axis2"], "dtype": "bfloat16"} },
  "reference": "import torch\n\ndef run(...):\n    ..."
}
```

Key conventions:
- **Axes**: `type: "var"` for runtime dimensions (batch_size, seq_len); `type: "const"` with
  `value` for model-specific constants (num_heads, hidden_size)
- **Tags**: `stage:`, `status:`, `model:`, `fi_api:`, `tp:`, `ep:`, `quantization:` prefixes
- **Reference**: Plain PyTorch `run()` function serving as ground truth
- **TP/EP**: Some kernel types (attention, MoE) produce separate definitions per tensor/expert
  parallelism setting because parallelism changes constant axis values (e.g., head counts,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flashinfer-ai/flashinfer-bench](https://github.com/flashinfer-ai/flashinfer-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
