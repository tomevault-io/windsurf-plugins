---
trigger: always_on
description: This is the canonical root instruction surface for coding agents in this repo. Keep this file and `STYLE.md` short, stable, and authoritative. Put extended, example-heavy standards in [`.agents/standards/`](.agents/standards/README.md). If those standards disagree with this file or `STYLE.md`, the root files win.
---

# Banksia coding agent contract

Status: Reference

This is the canonical root instruction surface for coding agents in this repo. Keep this file and `STYLE.md` short, stable, and authoritative. Put extended, example-heavy standards in [`.agents/standards/`](.agents/standards/README.md). If those standards disagree with this file or `STYLE.md`, the root files win.

## Product purpose

Banksia is an accountable agent-team runtime for complex work that must remain auditable, reproducible, trackable, and operationally recoverable.

- controller-owned runtime truth stays separate from provider behavior
- explicit routing and boundaries beat hidden conversational continuity
- Task-member, Operator, product, and support/audit lanes stay distinct
- prompts, Work Plans, loose files, reviews, and observability stay explicit enough to validate and recover

## Design philosophy

- start from the owning product contract and verify the shipped code against it
- change one ownership seam at a time while preserving controller invariants
- keep recovery, routing, and ownership rules teachable
- treat docs, prompts, examples, and gates as implementation inputs
- keep support observability subordinate to controller truth

## Principles

- do not assume agents know the product concepts, nouns, or rules unless the prompt or docs restate them
- do not assume hidden transcript memory is sufficient for correctness
- do not assume cross-system context sharing is robust, cheap, or lossless
- do not assume filesystem state is canonical runtime truth unless canon says so
- do not assume repo-local YAML or packaged definition files stay canonical after a controller-owned definition registry exists
- do not assume validation preview is equivalent to publish-, start-, commit-, or runtime-time legality
- treat Banksia as one-process and local-tool-first until canon explicitly adds distributed delivery
- do not assume retries are safe to replay across queued or distributed delivery
- do not assume support-state files are authoritative controller truth
- do not introduce compatibility aliases without an explicit product contract
- do not assume provider terminal success implies assignment success
- do not assume missing contract details can be reconstructed safely from nearby code shape
- keep exact inline-versus-after-return timing and sync/async ownership with the owning subject page

## Authority split

- `AGENTS.md` owns shared repo policy, routing, verification expectations, and delegation rules
- `STYLE.md` owns measurable coding standards and refactor triggers
- `.agents/standards/*` owns long-form structural, readability, test, docs, and boundary guidance
- [Naming standard](.agents/standards/code/naming.md) owns long-form symbol, module, and package naming guidance
- [Source layout standard](.agents/standards/structure/source-layout.md) owns long-form monorepo, package-root, domain-first runtime, transport-thinness, and test-layout guidance
- public product docs, public reference/internals docs, and internal canon docs should remain distinct methodology layers
- `docs-internal/README.md` routes factual internal product and implementation owners under `architecture/`, `interfaces/`, `operations/`, and `verification/`
- durable accepted decisions live under `docs-internal/adr/**`
- public reference owns the external Workflow schema; internal interface and verification owners keep exhaustive prompt, operation, payload, and proof detail

## Instruction layering

- read this file first
- read `STYLE.md` second
- start from `docs-internal/README.md`, then read the smallest relevant internal owner pages before implementation work
- use `.agents/standards/*` for extended cleanup and layout guidance after the root surfaces
- if a closer subtree `AGENTS.md` is added later, treat it as local routing for that subtree, not a silent replacement for root canon

## Docs layout rule

The documentation layout is:

- public docs under `docs/**`
- maintained examples under `examples/**`
- factual internal owners under `docs-internal/{architecture,interfaces,operations}/`
- durable accepted decisions under `docs-internal/adr/**`

Rules:

- keep public docs versionless by default
- do not create version-era, current-versus-target, archive, or migration authority lanes
- do not recreate deleted execution or archive trees just to satisfy stale references

## Source of truth rule

- subject pages routed from `docs-internal/README.md` are the internal product and implementation source of truth
- public reference owns external schemas and maintained examples; generated readbacks remain subordinate to their named internal owner
- frontend work consumes the product/interface owner and generated controller contracts as its data boundaries
- external design repos, ignored source clones, screenshots, and static HTML handoffs are visual, state, and interaction references only; they do not override controller-owned routes, fields, states, or legality

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ringlochid/banksia](https://github.com/ringlochid/banksia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
