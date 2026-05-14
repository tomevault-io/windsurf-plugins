---
trigger: always_on
description: > Rules and standards for the ai-pipeline-core framework repository. Governs how framework code must be written, not what the framework provides (see `.ai-docs/` for API documentation).
---

# AI Pipeline Core — Coding Standards & Rules

> Rules and standards for the ai-pipeline-core framework repository. Governs how framework code must be written, not what the framework provides (see `.ai-docs/` for API documentation).

## Design Principles

1. **Framework Absorbs Complexity, Apps Stay Simple** — All heavy/complex logic lives in the framework. Application code built on this framework should be minimal and straightforward. Execution tracking, retries, deployment, progress tracking, storage, logging, and validation are handled automatically.

2. **Deploy First, Optimize Later** — Get working system fast. Architecture must allow future optimization without major refactoring.

3. **Distributed by Default** — Multiple processes on independent machines with centralized services (LiteLLM, ClickHouse, logging). Design to avoid race conditions and data duplication.

4. **AI-Native Development** — Designed for AI coding agents to understand, modify, and debug. One correct way to do everything. Definition-time validation catches mistakes before runtime.

5. **Single Source of Truth** — No duplicate documentation. Code defines behavior. Auto-generate documentation from code.

6. **Self-Explanatory Code** — Code must be understandable without deep-diving into documentation or framework source code. Naming, structure, and types make intent obvious.

7. **Automate Everything Possible** — If a check, validation, or transformation can be automated, it must be. Manual steps invite errors.

8. **Minimal Code** — Less code is better code. Every line must justify its existence.

9. **No Legacy Code** — No backward compatibility layers, deprecation shims, or references to previous implementations. Unused code must be removed immediately.

10. **No Unvalidatable Derivatives** — When a value is derived from a typed source (field name, class name, enum variant), it must be computed programmatically, not written as a manual string. Dict keys mirroring model fields, string identifiers mirroring class names — if the type checker can't trace it back to the source, derive it from the typed source instead. This prevents silent breakage when renaming.

11. **Sequential phases, not if/elif branches** — When logic is "try A, then fall back to B if A is insufficient," write it as two sequential blocks with a condition between them — not `if A: ... elif B: ...` which duplicates the B logic and obscures the relationship.

12. **Poka-Yoke (Mistake-Proofing)** — The framework follows the Poka-Yoke methodology: make mistakes impossible rather than relying on vigilance to avoid them. Prevention over detection, detection over correction. Every class validates its own constraints at definition/import time via `__init_subclass__`. Construction paths enforce correct provenance (four factory methods, no raw constructors). Frozen models eliminate mutation bugs structurally. Layered static analysis (ruff, basedpyright, semgrep, vulture, interrogate) catches what types alone cannot. When a bug is found, the response is to close the entire category — add a structural guard, not just a point fix (§3.2 Bug Response Protocol). Actionable error messages (§4.13) complete the loop: every failure tells the caller exactly how to fix it, enabling AI agents to self-correct without external documentation.

---

## 1. Architecture Rules

### 1.1 Async Execution

All operations must be asynchronous. No blocking I/O calls allowed.

**`async def` must contain async operations** — Functions declared with `async def` must contain at least one `await`, `async for`, or `async with` statement. Functions without async operations must not be marked `async`. Enforced via semgrep rule.

**Exceptions:**
- Protocol stubs (method signature only)
- ABC base class methods meant for override
- In-memory test implementations (e.g., MemoryDatabase)

### 1.2 Immutability & Safety

- **No mutable global state that creates inter-task dependencies.** Module-level variables fall into three permitted categories:

  **Category 1 — Constants and frozen configuration.** `settings = Settings()`, frozen mappings, `frozenset` constants, module-level type aliases. Always allowed.

  **Category 2 — Infrastructure singletons.** HTTP client pools, provider facades, rate limiters. Allowed when ALL of the following hold:

  - The variable is assigned exactly once at module scope and never reassigned.
  - Internal state (connections, caches, locks) is a private implementation detail that never leaks to callers through the public API.
  - The singleton is **caller-stateless**: calling `await provider.fetch(url)` produces the same result regardless of what other tasks, flows, or deployments have previously called on the singleton. No task writes state that another task reads through the singleton.
  - The singleton exposes an `override()` context manager for test replacement, backed by a ContextVar for per-test isolation.
  - The variable is annotated with a `# infrastructure singleton` comment at the assignment site.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [researchtech-inc/ai-pipeline-core](https://github.com/researchtech-inc/ai-pipeline-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
