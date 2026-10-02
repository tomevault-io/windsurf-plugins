---
trigger: always_on
description: > **Library class: reference-grade.** Deletion requires a domain argument — wrong abstraction, subsumption by another construct, or theory-unsoundness; a usage count is inadmissible as a deletion ground (P7, 2026-08-17).
---

# gen (hub) — agent capability sheet

> **Library class: reference-grade.** Deletion requires a domain argument — wrong abstraction, subsumption by another construct, or theory-unsoundness; a usage count is inadmissible as a deletion ground (P7, 2026-08-17).

## Scope

The ecosystem hub: it owns no concern of its own. It publishes `mkGenLibs` — two-stage instantiation
of the gen library roster (stage 1 is `lib/mkGenLibs.nix`, a function of RESOLVED MEMBER VALUES that
binds the roster; stage 2 is a vestigial-argument wrapper bound in `flake.nix` that hands back that
same value) — the three **stratum buckets**
cut from that roster (`lib.substrate`, `lib.modules`, `lib.aspects`, `lib.framework`), and one
flake-parts module.
It publishes **no `mkCi`**: the CI-flake wrapper every library's `ci/` calls is
`gen-harness.lib.mkCi`, in its own repository. The harness pins no gen library, so a sibling's `ci/`
lock no longer drags the aggregator that pins that sibling — and since the hub no longer re-exports
it, there is no second route back to that edge.

## Not this library's job

The hub owns nothing; every concern belongs to a member library. Quoted text is that member's own
`flake.nix` `description` field, verbatim. Left column is the roster key under `mkGenLibs`.

<!-- gen-roster:begin -->

| Concern (roster key) | Owner                                                                                                                                                                                                                                        |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `prelude`            | `gen-prelude` — "gen-prelude: vendored, nixpkgs-lib-free pure utilities for the gen ecosystem"                                                                                                                                               |
| `algebra`            | `gen-algebra` — "gen-algebra: pure Nix algebra — records, intensional functions, either"                                                                                                                                                     |
| `types`              | `gen-types` — "gen-types: pure, nixpkgs-lib-free structural type checker for the gen ecosystem"                                                                                                                                              |
| `merge`              | `gen-merge` — "gen-merge — pure-Nix byte-mode module MERGE engine (evalModuleTree) for the pure-gen module system"                                                                                                                           |
| `schema`             | `gen-schema` — "gen-schema: typed record registry with extension points for the pure-gen module system"                                                                                                                                      |
| `aspects`            | `gen-aspects` — "gen-aspects: aspect-oriented composition types (pure-gen, re-hosted on gen-merge)"                                                                                                                                          |
| `scope`              | `gen-scope` — "gen-scope: demand-driven attribute grammar evaluator over algebraic scope graphs"                                                                                                                                             |
| `graph`              | `gen-graph` — "gen-graph: accessor-based graph query combinators"                                                                                                                                                                            |
| `select`             | `gen-select` — "gen-select: selector algebra for attributed graph positions"                                                                                                                                                                 |
| `bind`               | `gen-bind` — "gen-bind: module binding with external arguments for Nix"                                                                                                                                                                      |
| `dispatch`           | `gen-dispatch` — "gen-dispatch: relational rule dispatch over ordered groups (the dispatch STEP)"                                                                                                                                            |
| `class`              | `gen-class` — "gen-class — pure-Nix class-share mechanism (partition / contract / apply / gate) for the pure-gen module system"                                                                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sini/gen](https://github.com/sini/gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
