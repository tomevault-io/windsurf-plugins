---
trigger: always_on
description: Cairn is a graph-based architecture map for codebases: systems, containers, modules, and actors as nodes joined by dependency edges, each node carrying code targets, contracts, artefacts (decisions, todos, research), and temporal history. The graph is the source of truth for what exists, how it connects, and why it is shaped that way.
---

# Cairn: Agent Orientation

Cairn is a graph-based architecture map for codebases: systems, containers, modules, and actors as nodes joined by dependency edges, each node carrying code targets, contracts, artefacts (decisions, todos, research), and temporal history. The graph is the source of truth for what exists, how it connects, and why it is shaped that way.

Two chains meet at a hinge: the **provenance chain** (evidence flowing in: Source → Research → Decision) and the **authority chain** (rules flowing out: Decision → Blueprint → Contract → Code). Describe the architecture through this two-chain topology, never as a flat stack of layers.

## Start here

Start at `.claude/skills/cairn-dev/SKILL.md`. It is a short router: it names the target-authority precedence, the first orientation query, and the gate, then points at the one reference your task needs. Load the reference the router sends you to, not all of them.

If your task has a change directory (`meta/changes/<change-id>/`), work from its `proposal.md` (why), `design.md` (how), `tasks.md` (what), and `specs/` (acceptance criteria).

## Where things live

| Path | What |
|---|---|
| `docs/conventions.md` | Rust conventions (error codes, module size, state versioning, testing, docs). Authoritative. Section 10 gives artefact placement; section 11 routes new prose to its owning layer, so check it before adding prose anywhere. |
| `docs/registries/` | `declared-items.md`, `error-codes.md`. Check when adding public items or error codes to avoid collisions. |
| `archive/openspec/changes-archive/<phase>/specs/` | Other phases' acceptance criteria. Relevant only when your design.md references them. |
| `archive/openspec/specs/<area>/spec.md` | Consolidated per-area specs, distinct from the per-phase criteria above. |
| `docs/spec.md` | Narrative model and history. Fallback for what the graph cannot answer; never bulk-loaded for routine work (`dec.spec-authority-retirement`). |
| `docs/design-system/` | Canonical tokens, components, fonts, and live reference for any UI work. |
| `docs/` | Marketing landing page (GitHub Pages target); pulls from the design system like any UI surface. |
| `cairn.blueprint` | Root blueprint: cairn describing itself (dogfood). The graph's source of truth. |
| `tests/fixtures/cairn-bootstrap/` | Bootstrap fixture for tests; may lag the root blueprint, gate-asserted to scan clean (`tests/examples_gate.rs`). |

When implementing a feature phase with a paired `phase-<N>.0-tests` change, remove the matching `#[cairn_planned(phase = <N>)]` attribute as the feature lands rather than rewriting those tests from scratch. The attribute is structured (proc-macro), not a comment; do not parse the `#[ignore]` reason string.

## Terminology

CAIRN spec is v0.8. Use `blueprint`/`.blueprint` (not `DSL`/`.dsl`) and `map`/`map.md` (not `ontology`/`index.md`) in all new prose, code identifiers, and spec drafts; the phase 2.6 rename is applied and archived (merge commit `3f15946`). `DSL`/`.dsl` string literals in `src/cli/mod.rs` and `src/blueprint/parser.rs` are intentional legacy-file detection; leave them.

Preserve these distinctions; the taxonomy encodes them deliberately, so do not propose flattening it:

- `reconciler` (pluggable interface), `scanner` (engine), `scan` (verb/CLI): three distinct concepts.
- `artefact`: typed-schema kernel primitive (umbrella kept; direct types are contract, decision, todo, research, review, source).
- `rationale tension`: advisory non-blocking finding class, distinct from `interface contradiction` (blocking).
- `change` / `changes/`: carries delta semantics (ADDED/MODIFIED/REMOVED/RENAMED); `proposal.md` lives inside it.
- `neighbourhood`: graph-theoretic query primitive.
- `provenance chain` / `authority chain`: spec §3 spine (see above).
- `interface hash`, `ghost`/`synced`/`orphaned`, `drift`, `divergence`, `verified`/`external`/`unverified`, `hinge`: preserve these distinctions.

## Project state and artefacts

For project status, outstanding work, or the reasoning behind a decision, query cairn directly: `cairn status` and `cairn context` to orient, `cairn change list` and `cairn frontier` for what is next, `cairn get` / `cairn neighbourhood` / `cairn decisions` for a node, `cairn lint --json` for structured findings. The graph is the source of truth, never markdown files, strongholds, or memory; anything under `docs/` or `archive/` is secondary context. The router's command reference lists every command.

`cairn scan --strict` is the verification gate (non-zero on Error or Warning), and a clean `cairn scan` (zero findings) is the target state: every new source file falls under a node `path` in `cairn.blueprint`, else extend a module or declare a new one. Record friction with `cairn feedback "<msg>"`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cairn-framework/cairn](https://github.com/cairn-framework/cairn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
