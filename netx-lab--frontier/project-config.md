---
trigger: always_on
description: - Current public branch supports `co-location` and sequential PDD / `pd-disaggregation`.
---

# Frontier: LLM Inference Simulator (Co-location + PDD Release)

## Release Status: `pre-release-v0.2`

- Current public branch supports `co-location` and sequential PDD / `pd-disaggregation`.
- AFD / `pd-af-disaggregation` remains outside this public release surface.
- The co-location example suite uses `--cc_backend_config_type analytical` for one-click smoke runs without the optional network simulator; direct CLI experiments may still select `astra_sim_analytical` explicitly.
- The PDD example suite also uses `--cc_backend_config_type analytical` and `--no-enable_parallel_clusters` for one-click sequential smoke runs.
- collective_sim is optional. Initialize and build its submodule only when you explicitly select `--cc_backend_config_type collective_sim`.

Frontier is a modular **discrete-event simulator (DES)** for large language model (LLM) inference. This `pre-release-v0.2` branch supports the **co-location** architecture, where prefill and decode run in a single monolithic cluster, and sequential **PDD / `pd-disaggregation`**, where prefill and decode run in separate clusters with KV cache transfer between them.

The supported PDD path requires sequential cluster execution. If a user selects `pd-disaggregation` with parallel clusters enabled, Frontier fails fast with message.

If a user selects `pd-af-disaggregation` or other unsupported disaggregated research surfaces, Frontier still fails fast with the guarded disaggregation release error.

This AGENTS.md release guide is intended to be the **authoritative entry point** for users and developers. Older documents in the repo may contain deeper narrative explanations but may lag behind the current code.

## Contents

- [What Frontier Simulates](#what-frontier-simulates)
- [Key Features](#key-features)
- [Supported System Architectures & Mode Compatibility](#supported-system-architectures--mode-compatibility)
- [Repository Layout](#repository-layout)
- [User Guides](#user-guides)
- [Install / Environment](#install--environment)
- [Docker Environment](#docker-environment)
- [Quick Start: Run a Simulation](#quick-start-run-a-simulation)
- [Examples](#examples)
- [Configuration Model](#configuration-model)
- [System Architecture](#system-architecture)
- [Metrics & Outputs](#metrics--outputs)
- [Canonical TTFT Contract for Frontier vs vLLM V1 Online Alignment](#canonical-ttft-contract-for-frontier-vs-vllm-v1-online-alignment)
- [Training (Execution-Time & Network Models)](#training-execution-time--network-models)
- [Profiling Utilities](#profiling-utilities)
- [Development Gates](#development-gates)
- [Tests](#tests)
- [Contributing](#contributing)
- [License](#license)
- [Other Documentation](#other-documentation)

## What Frontier Simulates

Frontier models an LLM serving system as a set of clusters and replicas processing incoming requests over time. It uses a DES event loop to represent:

- **Request arrival** and workload generation (synthetic or trace-based)
- **Hierarchical scheduling** (global → cluster → replica → pipeline stage)
- **Execution time prediction** for model operations (ML-driven or “dummy mode”)
- **MoE execution modeling**, including Expert Parallelism (EP) synchronization and token imbalance
- **Speculative decoding / MTP runtime modeling** for supported `spec_decode` methods
- **Prefix caching** with block-hash-based KV reuse on supported schedulers

## Key Features

- `co-location` and sequential `pd-disaggregation` system architectures for this release
- Runtime guard for `pd-af-disaggregation`, parallel PDD clusters, and unsupported disaggregated research surfaces
- MoE support (EP synchronization, routing and imbalance modeling)
- Speculative decoding support via `frontier/spec_decode/` and `ReplicaConfig.speculative_decoding_config`
- Prefix caching for supported replica schedulers (`vllm_v1`, `sglang`)
- Pluggable **communication-cost backend**:
  - ASTRA-Sim-inspired analytical backend (default for public examples and direct CLI defaults)
  - Collective-sim topology-aware backend (optional; requires explicit `--cc_backend_config_type collective_sim`)
  - Analytical backend
  - Vidur (sklearn-based) backend trained on profiling data
- Detailed metrics collection + optional plots
- Optional per-cluster **event logging** for debugging

## Supported System Architectures & Mode Compatibility

This release supports two runtime architectures:

- `co-location`: Monolithic mode with a single cluster.
- `pd-disaggregation`: Sequential PDD mode with separate `PREFILL` and unified `DECODE` clusters. Public examples use `--no-enable_parallel_clusters`.

The CLI/config parser still accepts the historical `sys_arch` choices so existing parameter parsing structures remain stable. Runtime behavior is stricter:

- `offline + co-location` is supported.
- `online + co-location` is supported where the selected scheduler/runtime path supports online mode.
- `offline + pd-disaggregation` is supported for the public PDD scripts.
- `online + pd-disaggregation` is supported for the public PDD scripts.
- `pd-af-disaggregation` aborts during `SimulationConfig.__post_init__()` with the guarded disaggregation release error.
- `pd-disaggregation` aborts unless `--no-enable_parallel_clusters` is provided.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NetX-lab/Frontier](https://github.com/NetX-lab/Frontier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
