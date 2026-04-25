---
trigger: always_on
description: <!-- [PROJECT CUSTOMIZATION] Replace this section with your repository's purpose -->
---

# Copilot Instructions

## Repository Purpose

<!-- [PROJECT CUSTOMIZATION] Replace this section with your repository's purpose -->
<!-- Example: This repository is a standalone implementation of [product/system]. -->
<!-- Example: Reference material under `ref/` and archived docs are evidence inputs, not runtime dependencies. -->

## Active Working Set

1. `docs/design/` is the live technical design source of truth.
2. `docs/plans/` contains current implementation baselines and approved sequencing.
3. `docs/decisions/` contains accepted constraints.
4. `docs/status/` contains active closure items.
5. `docs/reference/` contains long-lived technical evidence.
6. `docs/archive/` is historical only unless the user explicitly asks for historical research.
7. All active documentation must be written in English.

## Code And Architecture Boundaries

<!-- [PROJECT CUSTOMIZATION] Replace the rules below with your project-specific architecture constraints -->

1. Keep service boundaries aligned with `docs/design/`.
2. Keep runtime behavior aligned with documented design patterns.
3. Treat source directories as separate concerns; do not collapse unrelated responsibilities together.
4. Keep tenant routing, authorization, auditability, and error semantics explicit for every new endpoint or workflow.

## Design Documents and Traceability

1. Treat `docs/design/` as the coding-ready contract source for DTOs, validation, authorization, tenant scope, error behavior, and service-specific closure items.
2. Every new endpoint or write workflow must be traceable to at least one data source and one behavior source; active design conclusions must be written into `docs/design/` before coding starts.
3. If traceability is incomplete, record a closure item in the relevant design document; route it to `docs/status/outstanding-items.md` only when repository-wide or cross-service.
4. Consolidate temporary review, validation, and discovery packs into active docs and archive them once their conclusions are absorbed.

## Document Rules

1. Use stable filenames without date prefixes in `docs/design/**`, `docs/status/**`, and `docs/reference/**`.
2. Use `YYYYMMDD_` prefixes in `docs/decisions/**`, `docs/plans/**`, and `docs/archive/**`.
3. Prefer templates in `docs/templates/` when creating or rewriting formal artifacts.
4. After renaming, moving, or archiving documents, update README entries and in-body references.
5. Do not treat `docs/archive/**` as the default working set.
6. Keep root `README.md` stable unless the user explicitly asks for repository-overview changes.
7. Keep root `AGENTS.md` and `CLAUDE.md` limited to static repository facts; behavioral instructions belong in `.github/copilot-instructions.md`.

## Implementation Expectations

1. Design before code for medium or large changes. Update the relevant document under `docs/design/` before or alongside code changes.
2. Service or module contracts must define request and response shape, validation, authorization, tenant scope, error behavior, and test expectations before implementation starts.
3. Prefer small, reviewable increments grounded in one domain or one capability family at a time.
4. When an implementation plan is expressed as delivery slices, require one completed slice per git commit unless the user explicitly asks for a different commit strategy.

## Verification And Debugging Expectations

1. Do not claim a change is complete, fixed, or passing without fresh verification evidence from the current run.
2. When verification fails or behavior is unexpected, stop speculative fixes and use root-cause-first debugging before changing more code.
3. Prefer worktree-first or other isolated branch setup for larger implementation or plan-execution runs so the main workspace stays stable.
4. Parallelize only when tasks are genuinely independent and do not share files, runtime state, or ordered slice dependencies.
5. For code-focused feature or bugfix work with a practical automated harness, prefer test-first changes and make the verification path explicit.

---

## Working Method Overview

| Asset | Location | Purpose | When to Use |
| --- | --- | --- | --- |
| **Prompts** | `.github/prompts/*.prompt.md` | One-shot focused entry points for narrow tasks | Task is clear, output is a single deliverable |
| **Skills** | `.github/skills/*/SKILL.md` | Multi-step repeatable workflows with staged reasoning | Task needs sequenced stages, bundled references, or subagent coordination |
| **Agents** | `.github/agents/*.agent.md` | Parent-coordinated worker subagents | A skill or prompt needs to delegate a bounded worker step |
| **Templates** | `docs/templates/*.md` | Standard document structures for formal artifacts | Creating or rewriting design, plan, decision, or evidence documents |

> **Agents are never user-facing starting points.** They are always delegated to by a parent skill or prompt. When the right entry point is unclear, use `.github/workflow-decision-tree.md`.

---

## Delivery Lifecycle

This repository follows a **12-stage** staged delivery lifecycle. Full stage detail, cross-cutting capabilities, subagent reference, and template reference: see [`.github/workflow-lifecycle.md`](.github/workflow-lifecycle.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LazyWorkshopCreate/agent-based-workflow](https://github.com/LazyWorkshopCreate/agent-based-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
