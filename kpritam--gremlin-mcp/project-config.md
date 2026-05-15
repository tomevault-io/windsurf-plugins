---
trigger: always_on
description: Authoritative operational standard for all AI coding agents (Copilot, Claude, GPT, others) contributing to this repository. Defines mandatory engineering rules, workflow protocol, architectural constraints, and decision heuristics to ensure consistent, safe, high‑quality, effect‑based TypeScript contributions.
---

# AI AGENTS GUIDELINES

## 1. Title & Purpose

Authoritative operational standard for all AI coding agents (Copilot, Claude, GPT, others) contributing to this repository. Defines mandatory engineering rules, workflow protocol, architectural constraints, and decision heuristics to ensure consistent, safe, high‑quality, effect‑based TypeScript contributions.

## 2. Scope (What this file governs / What it does not)

Governs: Code generation, refactors, tests, docs authored by AI; architectural changes; effect modeling; schema & Gremlin interactions; error & data modeling; PR structure; automation hygiene.
Does NOT govern: Human editorial style guides, product roadmap, infrastructure ops, non-code repository assets. If a conflict exists with LICENSE or SECURITY.md, those take precedence.

## 3. Core Principles (short list)

1. KISS – minimal viable abstraction.
2. YAGNI – add complexity only with a concrete second use.
3. DRY – no duplicated logic; prefer reuse/composition.
4. Purity – isolate side effects at explicit boundaries.
5. Explicitness – types, errors, contracts are declared, not inferred from context.
6. Determinism – schema generation & transformations must be reproducible.
7. Composability – build small effectful units combined via functional operators.
8. Safety – no unchecked external data, no silent failures.
9. Observability of intent – names reveal purpose; no hidden coupling.
10. Focus – each change addresses one coherent concern.

## 4. Operational Rules (MUST / MUST NOT)

### Architecture & Design

MUST:

- Preserve existing layered separation (config / gremlin services / schema / handlers / utils / models).
- Keep modules single‑purpose; split when a file exceeds clearly separated responsibilities (heuristic: > ~300 lines with 2+ conceptual domains).
- Encapsulate traversal construction behind utilities/services; expose intention, not query strings.
- Represent cross‑cutting concerns (config, schema cache, connection) via injectable services.
- Maintain deterministic schema assembly (stable ordering, idempotent recomputation, cache invalidation explicit).
- Introduce an abstraction only after a second concrete call site or clear complexity reduction.
- Ensure new layers or services declare explicit dependency requirements (no implicit global access).

MUST NOT:

- Mix query building logic with schema generation or result parsing.
- Create circular dependencies between modules.
- Leak internal traversal fragments to handlers or tests.
- Add global singletons outside controlled Layer/service patterns.
- Hide IO inside ostensibly pure helpers.

### TypeScript & Types

MUST:

- Use strict, explicit public function signatures (return + parameter types).
- Prefer `type` for unions/intersections/discriminated unions; `interface` for extensible object contracts.
- Use discriminated unions for state variants instead of boolean flags.
- Prefer readonly & immutable data structures unless a justified hotspot.
- Replace magic literals with named `const` declarations.
- Use `unknown` + type guards instead of `any` for external/unvalidated data.
- Export only stable public surface from barrel indexes; keep internals file‑scoped.

MUST NOT:

- Use `any` (except in a boundary shim with justification comment).
- Silence compiler warnings via assertions (`as`, `!`) unless after explicit validation.
- Re‑declare types already available through inference if local, unless clarity improves public API.
- Overuse generics where a concrete type is simpler.

### Functional & Effects

MUST:

- Model all side effects as typed Effects (or typed result constructs) rather than raw Promises.
- Keep effect execution (unsafe run / actual IO) at boundary layers (server entrypoints, handler adapters, CLI bootstrap).
- Compose behavior with `pipe`, `map`, `flatMap`, `mapError`, avoiding nested callbacks.
- Separate pure data transformation from effect orchestration.
- Provide explicit effect return type annotations for exported functions.

MUST NOT:

- Interleave imperative mutation with effect composition.
- Block or busy‑wait inside effects; use async composition.
- Partially execute effects during module top‑level evaluation.

### Error Handling

MUST:

- Represent failures using typed error/result constructs (Either/Option/Result style or domain error classes) – never raw throwing in core logic.
- Include structured context fields (e.g. code, operation, parameters) in error values.
- Distinguish operational (retryable) vs. domain (validation) vs. programmer errors via type shape or discriminant.
- Fail fast on invalid schema definitions; do not continue with partial state.
- Convert external/library exceptions into domain error representations at the boundary.

MUST NOT:

- Throw raw errors inside domain, parsing, schema, or transformation layers.
- Swallow or log‑only errors without returning a typed failure.
- Return opaque string errors.

### Data & Models

MUST:

- Validate external/untrusted data immediately (type guards or schema parsing) before internal use.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kpritam/gremlin-mcp](https://github.com/kpritam/gremlin-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
