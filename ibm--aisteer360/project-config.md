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

The four control categories, defined by what a method touches:

- **input**: manipulates the prompt only; generations follow `y ~ p_theta(sigma(x))` for a prompt adapter `sigma`.
- **structural**: persistently modifies weights or architecture (fine-tuning, DPO, merging); `y ~ p_theta'(x)`.
- **state**: edits internal activations at runtime via forward hooks, without changing weights.
- **output**: shapes the decoding process (logits processing, stopping, re-ranking, custom decode loops).

Vocabulary used throughout the codebase:

- **control**: one steering method, subclassing the base class of its category.
- **generic**: a reusable building block in a category's `_common/` library (transforms, gates, drivers, selectors,
  formatters, ...). Named methods are often thin presets over generics.
- **probe**: a calibrated linear readout over hidden states used for detection and routing (reads, never edits).

## Repository map

```
aisteer360/
├── algorithms/
│   ├── core/                    # SteeringPipeline, registry, ControlSpec, BaseArgs, shared types
│   │   ├── internals/           # activation capture, pooling, stats; probes/ (detection + routing rules)
│   │   └── utils/               # control merging, generation helpers, auxiliary_pass
│   ├── input_control/           # each category: base.py + one folder per method (triplet layout below)
│   │   └── _common/             # generics: memory, formatters, proposers, scorers, selectors
│   ├── state_control/
│   │   └── _common/             # generics: transforms, estimators, gates, selectors, hook runtime
│   ├── output_control/
│   │   └── _common/             # generics: drivers, processors, scorers, values, criteria
│   └── structural_control/
│       └── wrappers/            # trl/ (sft, dpo, ppo, grpo, apo) and mergekit/
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
(matplotlib/seaborn), `all` (all features), `dev` (`all` plus pytest, pre-commit, notebook), `docs` (site tooling).

Hugging Face access uses a `.env` file at the repo root containing `HUGGINGFACE_TOKEN=hf_***` (see
`.env.example`). Some models (e.g. `meta-llama/*`) are gated; the account behind the token needs access on the
model's Hub page. Never commit tokens; a detect-secrets pre-commit hook scans against `.secrets.baseline`.

Models run inside the current process. Real steering runs need GPU memory for the base checkpoint plus the method's
overhead; for smoke tests use the tiny models listed in `tests/utils/ci_models.yaml`
(e.g. `hf-internal-testing/tiny-random-LlamaForCausalLM`).

Common commands:

```bash
pytest tests/controls/                    # all control tests
pytest tests/controls/test_pasta.py       # one control
pytest tests/core/ tests/internals/       # pipeline, registry, probes
pre-commit install                        # once per clone
pre-commit run --all-files                # detect-secrets, whitespace, isort (black profile)
uv pip install -e ".[docs]" && uv run mkdocs serve   # docs at localhost:8000
```

Tests parametrize over the models in `tests/utils/ci_models.yaml` and over devices (`cpu`, `cuda`, `mps`); unavailable
devices are skipped automatically, so the suite runs on CPU-only machines. Commit messages require a DCO
`Signed-off-by:` line (see `CONTRIBUTING.md`).

## Usage guide

### Minimal steering loop

Every use of the toolkit follows the same loop: instantiate controls, wrap them in a `SteeringPipeline`, call
`steer()` once, then call `generate()` for inference.

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBM/AISteer360](https://github.com/IBM/AISteer360) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
