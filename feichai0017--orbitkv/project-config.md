---
trigger: always_on
description: This file provides guidance for agents working in the OrbitKV repository.
---

# OrbitKV Agent Guide

This file provides guidance for agents working in the OrbitKV repository.

## Current execution and review

Use [the completion plan](docs/completion-plan.md) for the next delivery sequence.
It is the only execution queue; architecture documents explain contracts,
not independent completion claims. Do not recreate a root TODO or parallel roadmap. Check the current code before reviving an old
unchecked item. Mark implemented-but-unqualified work separately from missing code.

For the current handoff, the implementation agent delivers one stage or coherent
substage at a time; the reviewing agent independently inspects its diff, reruns
the relevant gates and checks the evidence before accepting dependent work.
Return the commit, changed contracts, tests, external artifact locations and
remaining limits. A missing hardware gate stays unqualified, not passed.

## Documentation, layout and experiment evidence

- Follow [LMCache's project organization](https://github.com/LMCache/LMCache) and
  [documentation structure](https://docs.lmcache.ai/) for the README, quickstart,
  deployment recipes, compatibility matrix, benchmark methodology and contributor
  navigation. Adapt these to OrbitKV's Rust ownership boundaries; do not copy
  LMCache's implementation layers or claim its deployment support as our own.
- Preserve OrbitKV's website typography, colors, components and visual identity.
  Maintain technical content in `docs/` and render it on the website. Diagrams
  must distinguish implemented paths, experimental paths and future work.
- Keep experiment results outside the source checkout and Git history: raw
  JSON/CSV, logs, traces, run manifests, generated plots and per-run reports belong
  in an explicitly selected external artifact directory or CI/release artifacts.
  Do not force-add ignored output. Keep failed runs and controls with their evidence.
- Keep benchmark programs, reusable workload definitions, small deterministic
  test fixtures and reproduction instructions in the repo. Public README/docs
  may contain concise reviewed conclusions and stable evidence links; generated
  result collections and duplicate report tables are not source code. Publish
  measured charts from versioned external artifacts. Record revisions, hardware,
  budgets, correctness and uncertainty before claiming a performance advantage.
- Migrate existing tracked result collections only after verifying their archive
  and updating every documentation, website and reproduction reference. Do not
  delete the sole copy of evidence or rewrite Git history to hide old results.

## Repository Codex skills

Project skills live in `.agents/skills/` and are versioned with the code. Launch
Codex inside this repository so repository-scoped discovery applies. Read the
relevant `SKILL.md` for Python bindings, Manager operations, version changes or
released-engine integration and lifecycle diagnosis; resolve repository paths from the Git root. Keep one
maintained skill per workflow and update it when the consumed API changes.

## Project Overview

OrbitKV is a framework-neutral state cache with compiled recovery requirements
for LLM inference; general physical planning remains future work. Engine pins are vLLM `0.30.0`
and SGLang `0.5.20`; see S5.1 for the 0.30.0 qualification scope and historical
0.29.0 evidence. The SGLang UnifiedRadixCache linker is implemented, while
public lifecycle integration and broader deployment qualification remain open.

- Single-node KV cache offloading between GPU and host memory
- Cross-node KV cache sharing via Mooncake Transfer Engine (RDMA/TCP)
- Prefix cache reuse for repeated requests
- Python bindings and connectors for inference frameworks
- Framework-neutral state identity and recovery contracts

## Repository Layout

```text
orbitkv/
├── crates/
│   ├── orbitkv-state/           # Framework-neutral state and recovery contracts
│   ├── orbitkv-channel/         # iceoryx2/UDS process transport
│   ├── orbitkv-common/           # Process logging and peer-connection defaults
│   ├── orbitkv-core/             # Storage owners, planning, costs and execution
│   ├── orbitkv-proto/            # Protobuf and gRPC definitions
│   ├── orbitkv-server/           # Cache Manager orchestration and protocol adapters
│   ├── orbitkv-catalog/          # Complete local global index
│   ├── orbitkv-mooncake-sys/     # Pinned native build and dynamic C ABI
│   └── orbitkv-transfer/         # Mooncake transfer wrapper
├── python/                       # PyO3 package and framework adapters
├── third-party/                  # Pinned Mooncake, vLLM, and SGLang sources
├── examples/                     # Runnable usage examples
├── benches/                       # Benchmark code, workloads and reproduction
├── docs/                         # Architecture and completion plan
├── scripts/                      # Project helper scripts
├── .agents/skills/                # Repository-scoped Codex workflows
└── prek.toml                     # Local check configuration
```

## Where To Change Code

| Target | Location |
|--------|----------|
| State identity and recovery contracts | `crates/orbitkv-state/` |
| Inference-to-Cache-Manager process channel | `crates/orbitkv-channel/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feichai0017/orbitkv](https://github.com/feichai0017/orbitkv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
