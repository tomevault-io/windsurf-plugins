---
trigger: always_on
description: This document captures patterns, conventions, and preferences observed from maintainer reviews of recently merged pull requests in `prometheus/client_golang` as well as common standards shared across the Prometheus organization. Use this guide to align your contributions with what maintainers expect.
---

# Agents Guide for Prometheus client_golang

This document captures patterns, conventions, and preferences observed from maintainer reviews of recently merged pull requests in `prometheus/client_golang` as well as common standards shared across the Prometheus organization. Use this guide to align your contributions with what maintainers expect.

---

## Strict Backwards Compatibility & No Breaking Changes

`client_golang` is a foundational Go library (v1) relied upon by thousands of production services across the Go ecosystem.

- **No Breaking Changes:** Breaking changes to exported APIs, interfaces, struct definitions, or metric registration behaviors in stable packages (`prometheus`, `promhttp`, `api/prometheus/...`) are **strictly prohibited**.
- Do not remove, rename, or alter the signatures of exported functions, methods, or structs.
- Even subtle behavioral breaking changes (such as how labels are validated, how partial matches behave in `DeletePartialMatch`, or how panics are handled across wrapped collectors) must be avoided or safeguarded to preserve existing contracts.
- If an API change or new capability is needed, ensure it is completely backward-compatible or introduce it inside the `exp/` module.

---

## Experimental Features & Decoupling (`exp/` Module)

- **Use the `exp/` Module for Experimentation:** When introducing new capabilities, evolving APIs, unproven performance optimizations (such as sharded per-P counters, TTL eviction mechanisms, or alternative compression techniques), place them in the `exp/` module (`github.com/prometheus/client_golang/exp/...`).
- **Decoupling from Stable v1:** Building inside `exp/` decouples experimental development from the strict compatibility guarantees of the core `prometheus` package. It enables rapid iteration and gathering real-world user feedback without risking breaking changes to stable v1 contracts.
- Note that `exp/` has its own `go.mod` file and dependency management; ensure dependency updates in `exp/` are handled cleanly without impacting the root go module.

---

## API Design & Constructors

- **Avoid Proliferating `<constructor>WithXYZ` Variants:** Avoid introducing new constructor functions following the `<constructor>WithXYZ` or `<constructor>WithOptions` naming patterns (e.g., `NewCollectorWithClientAndTimeout(...)`). This pattern does not scale when new configuration options are introduced over time.
- **Use Config Structs or Functional Options:** Prefer configuration options structs (e.g., `Opts`, `Config`, `HandlerOpts`) passed into standard constructors, functional options, or setting fields where appropriate.
- When adding new parameters or behavior toggles to an existing object, extend its existing options struct (with sensible zero-value defaults) rather than multiplying constructor variants.

---

## Code Quality & Maintainability

- **Avoid Code Duplication:** Keep code DRY (Don't Repeat Yourself) while maintaining clarity and readability. When handling common logic across metric vectors, collector wrappers, or HTTP handlers (`promhttp`), extract clean, reusable internal utilities rather than copy-pasting logic across files.
- **Maintainable Code:** Prioritize long-term maintainability, simplicity, and clean encapsulation over clever or overly complex abstractions. Code should be easy to read, debug, and test for future contributors.
- **Memory & Map Ownership:** Pay strict attention to map and slice aliasing/ownership. When passing `prometheus.Labels` maps or slices into methods (such as `MetricVec.With(labels)` or `GetMetricWith(labels)`), explicitly state and enforce whether the underlying map or slice is copied or retained. Document caller ownership requirements at the interface/method boundary.
- **Collector Concurrency & Safety:** `Collect` and `Gather` operations must be safe for concurrent execution and resilient against unexpected panics (e.g., recovering panics cleanly from wrapped collectors).

---

## PR Title Format

Titles must follow `area: short description`, using a prefix that identifies the subsystem. Examples from merged PRs:

```
api/prometheus: clamp out-of-range formatted timestamps
promhttp: fix grammar in exemplar option doc comments
testutil: add native histogram assertion helpers
fix(prometheus): stabilize label maps in MetricVec.GetMetricWith
chores: remove example Dockerfile and container_description.yaml
build(deps): bump github.com/klauspost/compress from 1.18.7 to 1.19.0 in /exp
```

Common area prefixes in `client_golang`:
- `prometheus`: Core metric primitives, vectors, registry, and collectors.
- `promhttp`: HTTP instrumentation and metrics exposition handlers.
- `api/prometheus`: Client HTTP API packages (`api/prometheus/v1`).
- `exp`: Experimental features, metrics, and sync primitives.
- `testutil`: Test utilities and assertion helpers.
- `tutorials/<name>`: Tutorials and example modules.
- `docs`, `ci`, `build`, `chore`.

For performance-focused pull requests, append `[PERF]` to the area segment or use the `perf(area):` convention.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prometheus/client_golang](https://github.com/prometheus/client_golang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
