---
trigger: always_on
description: Guidance for AI agents working in this repository, covering both using the toolkit as a library (Usage guide) and
---

# AGENTS.md

Guidance for AI agents working in this repository, covering both using the toolkit as a library (Usage guide) and
extending it (Developer guide). The Invariants section lists rules that apply to every task. When this file and the
code disagree, the code is authoritative; verify against the source before acting on a claim here.

## Overview

AISteer360 is a toolkit for steering large language models (Hugging Face causal LMs). It provides steering methods
("controls") across four model control surfaces, a `SteeringPipeline` that composes controls from any categories into
one operation on a model, and an evaluation stack (use cases, metrics, benchmarks) for comparing steering pipelines.

Pipelines execute on one configurable backend: the in-process Hugging Face backend (default), the offline vLLM
engine (`kind="vllm"`), or a vLLM server (`kind="vllm-serve"`). Support is binary per control configuration and
backend for the generate and score phases; `pipeline.check()` reports unsupported combinations with a verdict naming
the gap and the fix, and unsupported operations raise before any work happens. The steer phase produces no verdicts:
each control declares its steer step's model access on the `ModelAccess` ladder (`facts` < `rollouts` < `capture` <
`module`), and `check()` additionally returns a deterministic steer plan stating where each step and fit will run
(see Execution backends below).

The four control categories, defined by what a method touches:

- **input**: manipulates the prompt only; generations follow `y ~ p_theta(sigma(x))` for a prompt adapter `sigma`.
- **structural**: persistently modifies weights or architecture (fine-tuning, DPO, merging); `y ~ p_theta'(x)`.
- **state**: edits internal activations at runtime via forward hooks, without changing weights.
- **output**: shapes the decoding process (logits processing, stopping, re-ranking, custom decode loops).

Vocabulary used throughout the codebase:

- **control**: one steering method, subclassing the base class of its category.
- **generic**: a dedicated recipe control class (`activation_adapter`, `value_guidance`, `search_decoding`, ...) that
  exposes common component slots through flat, sweepable `Args`, so a method from the literature is a configuration
  rather than a new class; named methods are siblings of generics, not children.
- **common library**: the per-category building blocks in `common/` (transforms, gating, drivers, selectors,
  formatters, ...) from which generics and named methods alike are assembled.
- **probe**: a calibrated linear readout over hidden states used for detection (reads, never edits); gating and
  routing consume its decisions.

## Repository map

```
aisteer360/
├── algorithms/
│   ├── core/                    # SteeringPipeline, registry, ControlSpec, BaseArgs, shared types
│   │   ├── execution/           # backend seam: spec, contracts, payloads, backend/session/registry, params, fanout
│   │   ├── internals/           # activation capture, pooling, stats; probes/ (detection)
│   │   └── utils/               # control merging, generation helpers, auxiliary_pass
│   ├── input_control/           # each category: base.py + one folder per method (triplet layout below)
│   │   └── common/             # building blocks: memory, formatters, proposers, scorers, selectors
│   ├── state_control/
│   │   └── common/             # building blocks: transforms, estimators, gating, selectors, hook runtime
│   ├── output_control/          # methods incl. routed_decoding/ (control, routing.py, actions.py)
│   │   └── common/             # building blocks: drivers, processors, scorers, values, criteria
│   └── structural_control/
│       └── wrappers/            # trl/ (sft, dpo, ppo, grpo, apo) and mergekit/
├── backends/                    # huggingface/ (HFBackend, ExclusiveSession); vllm/ (VLLMBackend, VLLMServeBackend)
├── evaluation/
│   ├── benchmark.py             # Benchmark runner (trials, sweeps, checkpoint/resume)
│   ├── metrics/                 # base.py, base_judge.py; generic/ and custom/<use_case>/
│   ├── use_cases/               # base.py; one folder per use case (use_case.py)
│   └── utils/                   # data_utils, generation_utils, metric_utils, viz_utils
└── utils/                       # tokenization, rendering, optional-dependency guard

docs/                            # MkDocs site: home/, concepts/, tutorials/, reference/, .nav.yml
examples/                        # notebooks/{algorithms,generics,benchmarks,recipes}/ + index.md
tests/                           # controls/, core/, internals/, evaluation/, utils/; conftest.py
```

## Setup and commands

Python 3.11+ with `uv` as the package manager:

```bash
uv venv --python 3.11 && uv pip install -e ".[dev]"
source .venv/bin/activate
```

On Windows, run the two chained commands separately. Optional extras: `merging` (MergeKit), `cpo` (econml), `plots`
(matplotlib/seaborn), `vllm` (the vLLM backends plus the `vllm_hook_plugins` core, git-pinned until its PyPI
release), `guided` (xgrammar, for in-process constrained decoding), `all` (all features except `vllm` and `guided`), `dev`
(`all` plus the plugin core, pytest, pre-commit, notebook), `docs` (site tooling).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [generative-computing/steerability](https://github.com/generative-computing/steerability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
