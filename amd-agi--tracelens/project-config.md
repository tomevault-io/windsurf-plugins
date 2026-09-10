---
trigger: always_on
description: Copyright (c) 2025 - 2026 Advanced Micro Devices, Inc. All rights reserved.
---

<!--
Copyright (c) 2025 - 2026 Advanced Micro Devices, Inc. All rights reserved.

See LICENSE for license information.
-->


# Generate optimization recommendations with the TraceLens Agent
```{meta}
:description: Learn how to run the TraceLens Agent, an agentic workflow that turns a GPU trace into a prioritized, stakeholder-facing performance optimization report.
:keywords: TraceLens, TraceLens Agent, agentic analysis, performance optimization, roofline, kernel fusion, GEMM, ROCm, AMD Instinct, optimization report
```

This topic shows how to run the *TraceLens Agent*, an agentic performance-analysis
workflow that reads a GPU trace and produces a single stakeholder-facing report,
`analysis.md`, organized as a prioritized bottleneck list. Findings are ranked and
grouped into three tiers: compute kernel optimizations, kernel fusion
opportunities, and system-level optimizations. Each finding carries the supporting
evidence, the reasoning behind the call-out, and a concrete resolution.

The agent combines a structured, skill-driven workflow with codified TraceLens
analysis for repeatable, reliable results.

## Analysis modes

The agent runs in one of two modes:

- **Standalone**: Single-trace roofline analysis. Use this when you have one trace
  and want to find where performance falls short of hardware limits, find system bottlenecks or fusion opportunities. This is the recommended default.
- **Comparative**: Two-trace gap analysis. The agent compares a primary trace
  against a reference trace, for example a different platform or a tuned config,
  and identifies inefficiencies in the primary trace relative to the reference.
  Comparative analysis works best when both traces come from the same framework.
  Cross-framework comparisons can produce misleading gap estimates because of
  structural differences in operation call stacks.

Support depends on the execution mode of the traced workload:

| Execution mode | Standalone | Comparative |
|---|---|---|
| Eager | Supported | Supported |
| Graph + capture | Supported | Supported |
| Graph | Not supported | Not supported |

## Before you begin

Complete the following setup steps before running the agent.

### Install TraceLens

Install locally, or into your container or virtual environment (see
[Install TraceLens](../install/install.md)):

```bash
pip install git+https://github.com/AMD-AGI/TraceLens.git
```

### Collect a trace

The orchestrator runs against a single `torch.profiler` trace (`.json` or
`.json.gz`). Collection is workload-specific:

- **Generic Eager Traces**: Instrument your loop with
  `torch.profiler.profile(...)`, enabling CPU-side call-stack and shape capture
  (`with_stack=True`, `record_shapes=True`). Profile a representative steady-state
  window of a handful of post-warmup steps, then log the trace with
  `prof.export_chrome_trace(...)`. A single rank's trace is enough for per-rank
  analysis.
- **Inference Traces with Graph Capture**: Collection has framework-specific
  requirements. Follow
  [Generate a PyTorch inference performance report](./generate-perf-report-pytorch-inference.md).
  The Profiling Skill automates
  vLLM, SGLang, and ATOM benchmarking and PyTorch profiler trace collection using
  Magpie, producing analysis-ready traces. For
  graph-mode workloads you produce two artifacts: a graph-replay trace and a
  graph-capture folder. In inference mode with execution mode
  `graph replay + capture`, TraceLens merges call-stack and shape information from
  the capture folder into the replay tree before analysis.

### Establish a hardware baseline

Roofline analysis compares each measured kernel against your GPU's max-achievable
TFLOPS and HBM bandwidth, so it needs a `<platform>.json` arch file for your
hardware. Bundled arch files ship with the package. If your platform isn't
included, or you want stack-specific measured values instead of published specs,
generate benchmark-derived peak TFLOPS and HBM bandwidth with the GPU
microbenchmarking suite. It writes the arch JSON in the shape the roofline
expects.

## Run the agent from a chat

Invoke the agent from any chat session with a capable model using one of the following prompts.

```{note}
The orchestrator skills are portable and work with agentic runners that support skill-file discovery.
```

In a chat with a capable model, invoke one of:

- Standalone (single trace):

  ```text
  Follow the analysis orchestrator installed with TraceLens and run the full
  agentic analysis workflow on <path_to_trace.json>
  ```

- Comparative (two traces)

  ```text
  Follow the analysis orchestrator installed with TraceLens and run the full
  agentic analysis workflow on <path_to_trace1.json> and <path_to_trace2.json>
  ```

If prompted, provide the trace file path, the platform of the first trace, the
analysis mode (`default` for training and eager inference outside vLLM, SGLang, and ATOM,
or `inference` for vLLM, SGLang, or ATOM), the execution mode and capture-folder path for
inference, environment details (node, container, or virtual environment), and an
optional output directory.

## Read the results

Only `analysis.md` is intended for end-user review. Everything else under
`analysis_output/` is agent internal: intermediates the orchestrator and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AMD-AGI/TraceLens](https://github.com/AMD-AGI/TraceLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
