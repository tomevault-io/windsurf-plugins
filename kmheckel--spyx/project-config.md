---
trigger: always_on
description: This document provides a concise overview of the Spyx project structure for AI coding agents and developers.
---

# AGENTS.md

This document provides a concise overview of the Spyx project structure for AI coding agents and developers.

## Project Overview

Spyx is a spiking neural network (SNN) library built on JAX and Flax NNX. It provides a compact, high-performance framework for training SNNs via surrogate gradient descent and neuroevolution, and — increasingly — a research vehicle for efficient sequence modeling (SNNs, state-space models, phasor/complex networks, and quantization under one roof).

## Stable core vs. `spyx.experimental`

Spyx has two tiers, and agents should respect the boundary when advising users:

- **Stable core** — the supported, API-stable surface: `nn`, `ssm`, `phasor`,
  `nir`, `bench`, `quant`, `data`, `optimize`, `fn`, `axn`. Rely on these for
  anything a user will depend on.
- **`spyx.experimental`** — research-stage building blocks whose **API may change
  without a deprecation cycle**: `PSU_LIF`, `ResonateFire`, `raven` (RavenRSM +
  SpikingSlotMemory), `compress` (packed-bit activations), `stochastic` (SPSN,
  stochastic-associative neurons), `hybrid` (surrogate + orthogonal-ES error
  correction), `zoo` (runnable recipes by application × method × architecture),
  `onnx` (ONNX export of the spiking step or the whole `nn.run` loop),
  `matfree` (matmul-free layers: ternary/BitNet + shift-add, and a MatMulFreeBlock).
  Always import these from `spyx.experimental`
  (e.g. `from spyx.experimental import PSU_LIF, RavenRSM`), never from a top-level
  module, so usage signals the stability contract. `PSU_LIF`/`ResonateFire` are
  physically defined in `nn`/`phasor` for code locality but are *surfaced* here.

When something in `experimental` matures (stable API, tests, docs, a real use
case), it graduates into the core namespace in a minor release.

## Technology Stack

- **Core Framework**: JAX (JIT compilation, automatic differentiation)
- **Neural Network Library**: Flax NNX (module system)
- **Optimization**: Optax (gradient-based optimization)
- **Data Loading**: Google Grain (data pipelines)
- **Package Management**: uv
- **Testing**: pytest
- **Linting**: Ruff
- **Documentation**: MkDocs with Material theme

## Directory Structure

```
spyx/
├── src/spyx/           # Main library code
│   ├── __init__.py     # Package initialization, public API exports
│   ├── axn.py          # Surrogate gradient functions (activation/axon)
│   ├── bench.py        # Benchmark harness (latency, throughput, MFU, spike-rate)
│   ├── data.py         # Data loading utilities and Grain transforms
│   ├── fn.py           # Functional utilities (losses, metrics, regularizers)
│   ├── nir.py          # NIR (Neuromorphic Intermediate Representation) support
│   ├── nn.py           # Neuron models (LIF, ALIF, CuBaLIF, etc.)
│   ├── optimize.py     # High-level training loop (fit, make_train/eval_step)
│   ├── phasor.py       # Complex-valued phasor & spiking-phasor networks
│   ├── quant.py        # int8/int4/BitNet quantization (qwix wrapper, optional)
│   ├── ssm.py          # State-space layers (LRU, S5Diag, Mamba, ChunkedSSM)
│   ├── experimental/   # Research-stage, UNSTABLE API (see below)
│   │   ├── __init__.py #   PSU_LIF, ResonateFire (re-exported), + the modules:
│   │   ├── raven.py    #   RavenRSM routing-slot memory + SpikingSlotMemory
│   │   ├── compress.py #   bit-packed activation storage for BPTT memory
│   │   ├── stochastic.py #  SPSN, StochasticAssociative*, sigmoid_bernoulli
│   │   ├── hybrid.py   #   0+1 trainer: surrogate grad + orthogonal-ES correction
│   │   ├── zoo/        #   reference recipes (control/classification/language)
│   │   └── onnx.py     #   ONNX export (per-step or full nn.run loop)
│   └── _version.py     # Version information
├── tests/              # Test suite (conftest.py pins JAX to CPU + seeds fixtures)
├── docs/               # MkDocs docs, organized by Diátaxis
│   ├── tutorials/      # Learning-oriented lessons
│   ├── how-to/         # Goal-oriented guides
│   ├── reference/      # API reference (mkdocstrings)
│   ├── explanation/    # Background & design
│   └── examples/       # Tutorial notebooks
├── research/           # Research experiments and benchmarks
├── scripts/            # Smoke tests, demos, release automation
├── .claude/            # Agent skills, SessionStart hook, settings
├── .github/workflows/  # CI (ci.yml) + PyPI publish (python-publish.yml)
├── pyproject.toml      # Project configuration and dependencies
└── README.md           # User-facing documentation
```

## Core Modules

### `axn.py` - Surrogate Gradients
Defines surrogate gradient functions for backpropagation through spiking neurons. Each is a factory that returns a JIT-compiled `jax.custom_gradient` function:
- `custom(bwd, fwd)`: Build an activation with arbitrary forward / surrogate-gradient functions
- `heaviside()`: The forward spiking nonlinearity
- `superspike(k=25)`: SuperSpike surrogate gradient (Zenke & Ganguli, 2018)
- `arctan(k=2)`: Arctangent surrogate gradient
- `triangular(k=2)`: Triangular surrogate gradient
- `boxcar(width=2, height=0.5)`: Boxcar surrogate gradient
- `tanh(k=1)`: Hyperbolic-tangent surrogate gradient

### `nn.py` - Neuron Models
Spiking neuron implementations using Flax NNX:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kmheckel/spyx](https://github.com/kmheckel/spyx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
