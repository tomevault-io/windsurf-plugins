---
trigger: always_on
description: Guidance for coding agents working in `tamer-op`.
---

# AGENTS.md

Guidance for coding agents working in `tamer-op`.

## Core intent
- Optimize for performance and clean architecture.
- Keep APIs ergonomic for new users.
- Primary user base is mathematicians (not software developers); optimize naming, defaults, and docs/comments for mathematical readability first.
- Prefer clarity over compatibility shims (project is pre-release).
- Keep `Workflow` thin orchestration; place subsystem logic in dedicated files/modules.

## Project-specific principles
- No legacy-compatibility layers unless explicitly requested.
- No alias-heavy APIs in performance-critical surfaces.
- Keep canonical mode/value contracts strict and explicit.
- Avoid hidden behavior switches; backend/mode choice should be inspectable.
- Do not keep "legacy"/"shim" naming in production paths (`legacy_*`, alias symbols, old keyword bridges).
- Favor signature cleanup over overload accretion: remove convenience arities that do not add real capability.
- Keep internal plumbing internal (`_name` + non-export) when it is not part of the intended user surface.
- Internal function policy (practical):
  - Keep internal functions only if they do at least one of:
    - remove duplication in real call paths,
    - isolate hot-path logic for performance,
    - enforce invariants/contracts centrally,
    - provide reusable plumbing used in multiple places.
  - Remove internal functions that are:
    - dead (unused),
    - one-line wrappers with no semantic boundary,
    - internal knobs not consumed by runtime behavior,
    - diagnostic-only helpers that do not support active tests or critical debugging.
  - Do not keep "switchable dead paths":
    - an internal fast path that stays off by default because it is not a consistent win should be treated as temporary tuning scaffolding, not permanent library structure,
    - either retune/narrow it until it is the canonical default in its winning regime, or delete it,
    - do not leave long-lived off-by-default branches in production code merely because they can still be toggled manually.
  - Apply the same cleanup rule to thresholded/gated paths:
    - if a gate never activates on realistic workloads, or the activated regime does not benchmark as a clear win, remove the path rather than keeping dormant branch/plumbing complexity,
    - remove associated dead workspaces/caches/tests/bench lanes when a path is deleted.

## API policy
- Breaking API changes are acceptable when they improve clarity/performance.
- Still keep convenience defaults (users should not need to pass empty `Options()` everywhere).
- Prefer one canonical public path over multiple near-duplicate wrappers.
- For mathematical subsystems, make the canonical public path task-oriented:
  - users should ask for the mathematical object/result they want,
  - not assemble it from internal storage-shaped helpers.
- Treat `Workflow` as the main notebook-facing task surface for cross-owner operations.
  - High-level tasks such as change-of-poset workflows on `EncodingResult`, common-refinement translation, cheap resolution tables, and exact 2D distance belong on `Workflow` when they materially improve discoverability.
- Default public wrappers to the cheapest mathematically meaningful output.
  - Prefer dims/summaries/lazy views by default.
  - Force explicit opt-in for heavy objects (full bases, representatives, page terms, quotient data, etc.).
- Advanced workflows must still be easy to discover and use without deep software-engineering context.
- If adding new options/fields, thread them through all relevant call paths (do not silently drop).
- Prefer a single cache keyword surface in Workflow APIs; avoid multi-keyword cache controls.
- When a Workflow wrapper becomes the curated root binding for a name that also exists on an owner module, preserve the owner-native capability on the Workflow generic.
  - Example pattern: if `betti_table` or `matching_distance_exact_2d` are rebound through `Workflow`, keep forwarding methods for resolution objects / cache objects so the root surface does not lose advanced functionality.
- Avoid adding new public wrappers unless they provide clear UX or measurable performance benefit.
- Add public wrappers when they materially improve mathematical readability, standardize a subsystem surface, or keep simple users off storage-shaped internals.
- Standardize UX across analogous ingestion families (point cloud / graph / image): same option names and semantics where possible.
- Prefer named-keyword forms for mathematically indexed queries when positional arguments are easy to misread.
  - Example pattern: `term(ss; page=2, p=a, q=b)` rather than positional page/bidegree arguments.
  - Keep positional forms for internal/advanced hot paths only when they add real value.
- Prefer one notation convention per public subsystem and use it consistently in docs and helpers.
  - If a subsystem uses cohomology degree `t`, homology degree `s`, and spectral bidegree `(p,q)`, keep that convention everywhere user-facing.
- Avoid forcing users to inspect raw struct fields for ordinary workflows.
  - Prefer typed accessors (`dimensions`, `basis`, `coordinates`, `component`, `structure_map`, etc.) and semantic object accessors over field archaeology.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erikmnovak/TamerOp.jl](https://github.com/erikmnovak/TamerOp.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
