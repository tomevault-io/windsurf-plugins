---
trigger: always_on
description: This file provides repository-specific guidance for working on Transformer
---

# Transformer Engine Agent Guide

This file provides repository-specific guidance for working on Transformer
Engine. `README.rst`, `CONTRIBUTING.rst`, `docs/`, `qa/` remain the authoritative sources for
user documentation, contribution policy, and executable CI behavior.

## Project Scope

Transformer Engine provides optimized building blocks for Transformer models
on NVIDIA GPUs. It is not a complete model-training system. Higher-level
frameworks and toolkits compose its operations and modules into models and own
model-level orchestration.

Most functionality is at or below the level of an individual Transformer
layer. Tensor, sequence, expert, and context parallel techniques may be in
scope when they affect Transformer Engine operations. Pipeline parallelism,
distributed optimizer orchestration, and complete training loops generally
belong to higher-level systems such as Megatron-LM.

Do not copy version tables into this file. Current hardware, CUDA, cuDNN,
compiler, Python, and framework requirements are maintained in `README.rst`
and `docs/installation.rst`.

## External Integration References

Transformer Engine must remain understandable and testable from this
repository. Some integration-facing APIs are used primarily by downstream
projects, especially Megatron-LM. When the purpose or expected usage of such an
API is unclear, inspect the downstream integration and the corresponding
launchers under `qa/` to understand its real usage. Treat that usage as
supporting evidence rather than the authoritative definition of Transformer
Engine behavior.

Preserve or add standalone tests in this repository for the relevant
Transformer Engine contract whenever practical. Downstream integration tests
supplement those tests; they do not replace them.

For external library dependencies such as cuDNN Frontend, consult the
documentation matching the version supported by Transformer Engine rather than
assuming behavior from the dependency's latest branch.

## Repository Map

- `transformer_engine/common/`: framework-independent C++, CUDA, C API,
  shared Python definitions, kernel dispatch, and CUDA-library integrations.
- `transformer_engine/pytorch/`: PyTorch public API, modules, quantized tensor
  types, recipes, attention, optimizers, state, and private C++ bindings.
- `transformer_engine/jax/`: JAX and Flax public APIs, primitives, custom calls,
  quantization, state, and private C++ bindings.
- `transformer_engine/debug/`: numerical debugging and inspection features.
  Verify current framework support before extending or documenting them.
- `tests/cpp/` and `tests/cpp_distributed/`: common-library tests.
- `tests/pytorch/` and `tests/jax/`: framework-specific unit, numerical,
  integration, attention, quantization, and distributed tests.
- `qa/`: launchers for Transformer Engine's authoritative CI test suites. The
  GitHub Actions workflows expose only a subset of the full CI; use the tests
  and launchers under `qa/` to determine the coverage expected by the project's
  internal CI. `L0_*` contains the basic contribution checks, while higher
  levels contain distributed, integration, and specialized coverage.
- `benchmarks/`: performance benchmarks and profiling utilities, separate
  from correctness tests.
- `docs/`: Sphinx sources, API references, tutorials, and notebooks.
- `examples/`: runnable PyTorch and JAX examples.
- `build_tools/`, `setup.py`, `pyproject.toml`, and `MANIFEST.in`: build,
  packaging, version, and wheel infrastructure.
- `3rdparty/`: Git submodules pinned by the parent repository.

## Architecture and Ownership

Transformer Engine has a framework-independent common layer and
framework-aware PyTorch and JAX layers.

The common layer owns reusable computation, the public C API, low-level
dispatch, CUDA kernels, calls to CUDA libraries, runtime-compiled kernels, and
shared concepts used by both frontends. The framework layers adapt tensors and
execution models, manage framework-visible state, interpret recipes and
options, and compose focused operations into user-facing modules.

Place framework-independent computation in `transformer_engine/common/`. When
a frontend needs compiled common functionality that is not already readily
available in both supported frameworks, expose it through the C API or a
Python-based DSL such as Triton and adapt it in that frontend's private binding.
Prefer designs that do not re-enter Python on every steady-state invocation.
CuTeDSL kernels dispatched from common C++ may be JIT-compiled and registered
through TVM FFI. Python may participate in initial registration and compilation,
but steady-state dispatch should use the cached C++/TVM FFI function.
Framework-specific tensor handling, autograd or transformation semantics, state,
and user-facing composition belong in the corresponding frontend.

### Working architecture rules

These rules describe the intended architecture. Check them against the
subsystem being changed and document necessary exceptions.

- The common layer must not depend on PyTorch or JAX.
- GPU kernels belong in the common layer; private framework bindings translate
  framework tensor and execution conventions into common API calls.
- User-visible policy and multi-operation composition belong in a frontend;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/TransformerEngine](https://github.com/NVIDIA/TransformerEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
