---
trigger: always_on
description: This file defines project-level guardrails for `turso_ex`.
---

# TursoEx Project Instructions

This file defines project-level guardrails for `turso_ex`.

These rules are here to protect the end-state shape of the library:

- a tiny public API
- clean package boundaries
- honest documentation
- no Ecto-driven contamination of the core package

## Working Style

This project uses a deliberate AI collaboration model.

How to work with me on this project:

- lead with understanding, not code. When I ask for a feature, start with the domain and constraints before writing implementation.
- ask sharp questions when edge cases or domain semantics are unclear in a high-impact way. Do not silently choose domain behavior that should be my decision.
- push back if a proposed design complects separate concerns. Name what is being tangled.
- build small. Give me a foundational piece I can validate in IEx before building the rest.
- prefer a small module plus example calls over a large speculative patch.
- when we are in novel domain logic (not boilerplate), prompt me to make the key decisions. Explain trade-offs and alternatives you considered.
- do not introduce unfamiliar patterns without explaining them first, unless they are already established in the repo.

## When To Ask vs Decide

- ask when the choice affects public API, domain semantics, failure semantics, or package boundaries
- decide when the choice is low-risk, internal, and reversible
- when deciding, note the assumption briefly and keep the change easy to revise

## Interpretation Rules

Use this file as a decision aid, not a bag of vibes.

When multiple instructions seem to overlap, resolve them in this order:

1. existing code and tests, unless they are clearly scaffolding or stale relative to the active contract
2. the active contract document in `plans/` (currently `plans/01-contract.md`)
3. hard package and API boundary rules in this file
4. documentation and roadmap files
5. style and taste guidance

Specific rules:

- if code, tests, and docs disagree, trust code and tests first, then fix the docs
- if the active contract doc and a later roadmap file disagree, treat the active contract as the source of truth
- if a decision affects the public API, package boundaries, or failure semantics, do not improvise silently
- if a decision is low-risk and reversible, choose the simpler path and note it
- do not invent a third behavior when two docs disagree, pick the source of truth and align the other file

## Decision Order

When making trade-offs in this project, prefer this order:

1. correctness and truthfulness
2. stable public API
3. clean package boundaries
4. simple Elixir data and functions
5. observability and debuggability
6. SDK parity
7. convenience sugar

If a choice improves parity but makes the Elixir API worse, parity loses.

## Product Bar

Build `turso_ex` as an Elixir library first, and a NIF wrapper second.

If a change makes the public API feel like renamed FFI, it is probably the wrong change.

The public API should stay centered on:

- `TursoEx.open/1`
- `TursoEx.query/3`
- `TursoEx.execute/3`
- `TursoEx.one/3`
- `TursoEx.transaction/2` later, when it is real

The public data model should stay centered on:

- `%TursoEx.Conn{}`
- `%TursoEx.Result{}`
- `%TursoEx.Error{}`

Prefer a small, boring, stable surface over broad early parity.

## Package Boundaries

Keep this repository as a mono-repo with separate packages, not an umbrella.

Rules:

- the repo root is a workspace, not a Mix app
- the core package lives in `packages/turso_ex`
- the adapter package lives in `packages/ecto_turso_ex`
- the Rust NIF crate lives in `packages/turso_ex/native/turso_nif`
- `packages/turso_ex` must not depend on `Ecto`, `EctoSQL`, or `DBConnection`
- do not restructure this repo into an umbrella unless there is a deployment reason, not just development convenience

The Ecto adapter exists to depend on the core package, not to dictate the shape of the core package.

## Layering Rules

Prefer this layering:

- `TursoEx`, canonical public API
- `TursoEx.Native`, thin NIF boundary
- `TursoEx.Driver`, only if later parity work clearly justifies it

Rules:

- do not expose `TursoEx.Native` as the primary user-facing API
- do not add `TursoEx.Driver` preemptively
- do not add a new layer unless it removes real complexity
- keep the public API shaped around user intent, not upstream method names

## API Taste

Write APIs the way a good Elixir library should feel:

- one obvious happy path
- stable structs instead of ad hoc maps and tuples everywhere
- minimal ceremony
- advanced control without infecting the beginner path
- no hidden SQL routing tricks in the core public contract

Specific rules:

- `open/1` returns a ready-to-use connection capability
- `%TursoEx.Conn{}` should stay opaque and boring
- `%TursoEx.Conn{}` retains both database and connection handles internally so sync support does not force a public struct change
- `%TursoEx.Result{}` should use positional rows in the base API
- `%TursoEx.Error{}` should stay machine-meaningful and pleasant in `iex`
- avoid adding convenience APIs unless they clearly earn their keep

## Documentation Rules

Documentation must tell the truth, especially around limitations.

Rules:

- distinguish engine compatibility, Rust SDK parity, and Elixir API support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bytebottom/turso_ex](https://github.com/bytebottom/turso_ex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
