---
trigger: always_on
description: version-date: 26-03-15
---

# AGENTS.md

author: heyao
version-date: 26-03-15

## 1. Project Overview

This repository implements a CacheRoute-style scheduling system for LLM serving experiments.

The system currently revolves around the following roles:

- **Scheduler**: the central control plane that selects target Proxy/KDN resources and builds requests.
- **Proxy**: executes or forwards requests, maintains runtime load information, and interacts with the scheduler.
- **KDN**: manages knowledge-related assets such as text chunks, KVCache, embeddings, and resource status.
- **Client / Perf Client**: generates workloads for testing and benchmarking.

The project is actively evolving. The current focus is not “general cleanup”, but **supporting research-driven scheduling strategies and controlled experiments**.

Codex should optimize for:
- correctness,
- incremental change,
- observability,
- reproducible experiments,
- and preserving existing behavior unless explicitly asked to change it.

### 1.2 Architecture Overview

Typical task flow:

client
  ↓
scheduler
  ↓
proxy
  ↓
instance
  ↓
vLLM engine

KDN provides knowledge resources used during scheduling
and request preparation.

Scheduler maintains resource pools for:
- proxies
- KDN nodes

### 1.3 CacheRoute Strategy Development Area

Scheduling strategies should be implemented under:

scheduler/strategy/
proxy/strategy/

Strategies must not modify core scheduler logic.
They should operate through the existing selection interfaces.

Example strategies include:
- round_robin
- load_based
- knowledge_aware
- hybrid_injection_policy

---

## 2. Current Research Direction

This repository is being extended for **strategy development under different task injection modes**.

The main task modes currently relevant are:

- **text**: send text directly for normal computation / recomputation.
- **kvcache**: inject KVCache-related information into the workflow.
- **hybrid**: mixed mode, e.g. alternating or ratio-based combinations of text and kvcache tasks.

The near-term research goal is to support **decision-making between KVCache injection and text-recomputation dynamically**, based on:
- queue state,
- network cost,
- compute cost,
- and resource availability.

When modifying code, prefer solutions that help future policy design, especially:
- measurable timing breakdowns,
- explicit request metadata,
- stable task typing,
- and resource-state visibility.

---

## 3. Engineering Priorities

When making changes, follow these priorities in order:

1. **Do not break the existing experiment flow.**
2. **Prefer incremental modifications over broad refactors.**
3. **Preserve existing public behavior unless the task explicitly requires changes.**
4. **Expose useful logs / debug status for validation.**
5. **Keep code easy to reason about for later paper-oriented modeling.**

Do **not** do the following unless explicitly requested:
- rewrite large modules,
- rename core concepts casually,
- move many files at once,
- introduce new abstractions without clear payoff,
- silently change request field semantics,
- remove debug outputs that are useful for experiments.

---

## 4. Expected Coding Style

### General
- Use **small, local, incremental patches**.
- Keep the original project structure unless there is a strong reason not to.
- Prefer **clear active-voice comments** over decorative comments.
- Avoid clever but opaque designs.

### Python style
- Preserve the repository’s existing style where possible.
- Add type hints when they improve clarity, but do not over-engineer typing.
- Keep functions focused and readable.
- Avoid introducing unnecessary framework-level indirection.

### Logging / Observability
- Add logs only when they help validate runtime behavior.
- Prefer logs that answer questions like:
  - which task was selected,
  - which injection type was used,
  - how long each phase took,
  - which proxy/KDN was chosen,
  - what resource/load snapshot was used.

Avoid noisy per-step logs unless explicitly requested. Favor **task-level summary logs**.

---

## 5. Change Strategy for Codex

When implementing a request, Codex should usually follow this workflow:

1. **Read the relevant files first.**
2. **Infer the minimum viable change set.**
3. **Modify only the files needed for that change.**
4. **Explain the patch in a file-by-file manner.**
5. **State how to verify the change.**
6. **Call out assumptions explicitly if code context is incomplete.**

For non-trivial changes, the preferred response format is:

- **What to change**
- **Why this is the minimal correct change**
- **Exact code patch**
- **How to validate**
- **What may break / edge cases**

---

## 6. Validation Expectations

Every meaningful code change should be easy to validate.

Codex should prefer adding or preserving validation hooks such as:
- debug endpoints,
- CLI-visible status,
- task-level timing output,
- resource pool snapshots,
- counters for success/failure events,
- explicit request field dumps in controlled logs.

When proposing changes, always include:
- where to run,
- what to send,
- what output is expected,
- what would indicate failure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AstraNetLab/CacheRoute](https://github.com/AstraNetLab/CacheRoute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
