---
trigger: always_on
description: This is the entry point for AI guidance in Apache Fory. Read this file first, then load only the `.agents/*.md` files that match the runtimes or areas you touch.
---

# AGENTS.md

This is the entry point for AI guidance in Apache Fory. Read this file first, then load only the `.agents/*.md` files that match the runtimes or areas you touch.

## Load Additional Guidance On Demand

- `.agents/README.md`: routing table for selective loading.
- `.agents/repo-reference.md`: repo layout, architecture, compiler notes, and key directories.
- `.agents/docs-and-formatting.md`: documentation, specification, and markdown rules.
- `.agents/ci-and-pr.md`: code review workflow, CI triage, PR expectations, and commit conventions.
- `.agents/testing/integration-tests.md`: `integration_tests/` prerequisites, regeneration rules, and commands.
- `docs/security/index.md`: security model index.
- `docs/security/threat-model.md`: project-level trust boundaries, non-goals,
  and downstream responsibilities.
- `docs/security/deserialization.md`: security boundaries for untrusted deserialization classification.
- `.agents/languages/java.md`
- `.agents/languages/csharp.md`
- `.agents/languages/cpp.md`
- `.agents/languages/python.md`
- `.agents/languages/go.md`
- `.agents/languages/rust.md`
- `.agents/languages/swift.md`
- `.agents/languages/javascript.md`
- `.agents/languages/dart.md`
- `.agents/languages/kotlin.md`
- `.agents/languages/scala.md`
- For protocol or xlang changes, load the relevant language files plus `.agents/docs-and-formatting.md` and `.agents/testing/integration-tests.md`.

## Agent Operating Rules

- Preserve architecture. Do not introduce new layers, parallel flows, or public APIs unless explicitly requested; prefer local repair in the existing owner over shared-infra expansion, and stop if a fix conflicts with an ADR, spec, or invariant.
- Respect ownership. Keep logic, state, and helpers in their natural owner, and do not move serializer-local, context-local, runtime-type-local, or protocol-local problems into global utilities.
- Check the spec before implementation. For wire behavior and xlang mapping, use the specs as the source of truth and never copy one runtime's bug into another runtime just to make tests pass.
- Do not make assumptions about runtime behavior, ownership, registration, metadata construction, protocol semantics, or test coverage. Read the current code, owning docs/specs, and relevant tests before making a design judgment or implementation decision. If the evidence is incomplete, inspect more or state the uncertainty explicitly instead of filling gaps from memory or analogy with another runtime.
- For untrusted deserialization, read `docs/security/deserialization.md` before changing allocation, stream filling, skip, reference, metadata, or policy validation behavior. Variable-length deserialization must not allocate or reserve backing/output capacity from attacker-declared lengths or counts before the byte owner has proven proportional readable bytes with `checkReadableBytes` or the runtime equivalent. Root graph memory reservation is accounting only and may happen before that byte check, but it must not replace the byte check.
- Root deserialization graph memory budgets are approximate gates for materialized graph owners,
  not exact heap accounting, input byte accounting, or raw element counts. `maxGraphMemoryBytes`
  defaults to fixed `128 MiB`; positive values override the default; explicit non-positive values
  are invalid and must be rejected at config/Fory creation. Do not add a disabled-budget sentinel
  path, derive this budget from root input size, or split known-length and stream root behavior.
  Read context/read state owns only raw byte reservation with `reserveGraphMemory(bytes)`; it must
  not expose counted arithmetic helpers or collection, map, array, struct, or object semantic
  reservation APIs. Do not add any non-memory-budget read-context/read-state API for this feature,
  including ref-publication controls, temporary-owner controls, serializer-owner controls,
  conversion helpers, or APIs that encode what kind of value is being materialized. Root facades may
  set/reset the per-operation budget, but they must not pre-reserve root type, root self bytes, or
  root value storage. Because the budget is fixed per root, read state must not mirror the
  configured max into a second active-limit field; use the existing config or one configured max
  field plus the mutable remaining budget. Concrete serializers and generated serializers own
  counted formulas, overflow checks, allocation-owner decisions, and reference publication timing
  for their allocation path. Reserve self storage exactly once at the owner that stores or allocates
  the value: reference/object runtimes reserve parent owner self cost plus reference storage and
  every referenced heap owner reserves its own shallow self cost when materialized; inline/value
  runtimes reserve value storage only in the holder or materializer that actually owns that storage,
  such as collection, map, set, array, smart-pointer, box, dynamic boxing, or reference-object field
  storage owners. Value serializers do not reserve their own self storage, including root and
  generated struct/product read paths. Collection, map, set, and reference-array owners reserve

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/fory](https://github.com/apache/fory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
