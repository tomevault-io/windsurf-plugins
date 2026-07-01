---
trigger: always_on
description: Guidance for AI agents implementing unworklet v1.0.0.
---

# AGENTS.md

Guidance for AI agents implementing unworklet v1.0.0.

> **Using unworklet (not contributing to it)?** This file is the _contributor /
> implementation_ contract. If you are an AI agent helping someone _build with_
> unworklet, the consumer-facing entry points are:
>
> - [`llms.txt`](./llms.txt) — install-time entry point + DSL quick reference.
> - Package READMEs with the exact call forms:
>   [`packages/core/README.md`](./packages/core/README.md),
>   [`packages/unplugin/README.md`](./packages/unplugin/README.md),
>   [`packages/lang/README.md`](./packages/lang/README.md),
>   [`packages/offline/README.md`](./packages/offline/README.md),
>   [`packages/test/README.md`](./packages/test/README.md).
> - [`.claude/skills/unworklet/SKILL.md`](./.claude/skills/unworklet/SKILL.md) — the same surface as a skill.

## What this repository is

A from-scratch implementation of `unworklet` — a TypeScript-first framework for declarative Audio Worklet DSP, compiled to WebAssembly. The v1.0.0 implementation is being driven by AI agents working in parallel against the specifications in `docs/`.

## Source of truth

- **`docs/`** is the authoritative specification. Read `docs/README.md` first to learn the read order and which component you own. Every component doc links to `docs/decisions-log.md` for the "why" behind any decision.
- If your work touches an area that is not yet covered in `docs/`, **stop and surface the gap** to the human reviewer rather than inventing the missing decision.

## Canonical examples integrity rule (HARD CONTRACT)

`docs/12-canonical-examples.md` is the **integrity anchor** for the entire spec. It is a curated set of self-contained, end-to-end plugin examples that exercise the full surface of unworklet (every primitive, every declaration, every main-side method). Any change to any other file in `docs/` — primitive shapes, declaration shapes, surface listings, decisions, transport contracts — must be cross-checked against the examples there before the change is accepted.

**Required process when modifying any `docs/*.md` (other than `12-canonical-examples.md` itself)**:

1. Identify which examples in `12-canonical-examples.md` exercise the surface you are changing (the `## Coverage` table at the top of that doc maps concept → example numbers).
2. Apply the proposed change to those examples (mentally or as a draft) and verify they still:
   - Compile under the changed surface (no broken signatures, types, or references).
   - Make sense for the realistic use case the example was designed for (no awkward workarounds, no apologetic comments).
   - Preserve the user's mental-model simplicity — the change should not force a production-grade audio plugin author to re-learn a concept they already understood.
3. **If any example breaks or becomes awkward, the proposed change is rejected** until either (a) the change is revised to preserve the example, or (b) the example is updated together with the change as a single coherent revision (and the resulting UX cost is made visible to the human reviewer in the same diff).

This rule is non-negotiable. Spec changes that pass review without an accompanying check against `12-canonical-examples.md` are **out of process**.

The same rule applies in the reverse direction: changes to `12-canonical-examples.md` itself trigger a re-read of the affected component docs to ensure the new example shape matches the spec — examples cannot drift from the spec, the spec cannot drift from the examples.

The purpose of this rule is to keep one question answerable at any time during spec evolution: **"is the user experience still simple, coherent, and production-ready?"** If the canonical examples no longer read that way, the spec change is the wrong shape regardless of how clean it looks in isolation.

## Implementation invariant (HARD CONTRACT)

unworklet v1.0.0 is built incrementally as 14 vertical slices (see `docs/10-roadmap.md` §2). **Minimal / vertical-slice implementations within a phase are by design acceptable**, but the following are **absolutely prohibited**:

1. **Ad hoc implementations that diverge from the spec** — the public API surface (public types, argument shapes, return value shapes, as defined in `09-repo-structure.md` §2.1 + §2.2 and each component doc) must match the spec exactly. Shipping a temporary shape with the intention of fixing it later within a phase is not allowed.
2. **Implementations that are not forward-compatible** — designs that conflict with surfaces added in later phases (e.g. adding declaration kinds, new primitives, main-side methods, or messaging surface extensions) are not allowed. Whatever is implemented within a phase must be a strict **subset** of the final architecture, shaped so that subsequent phases can expand it to a **superset** without rework.

At the start of each phase, consult every doc that touches the surfaces you are working on (`00-foundations.md`, `01-dsl.md`, the relevant component docs, and the applicable questions in `decisions-log.md`), and implement as a subset of the final target shape. "Minimal" means the smallest correct subset — it does not mean "whatever works now, rewrite later."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuichkun/unworklet](https://github.com/yuichkun/unworklet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
