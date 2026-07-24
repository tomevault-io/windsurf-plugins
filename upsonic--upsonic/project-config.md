---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Upsonic is a reliability-focused AI agent framework for building production-ready AI agents and digital workers. The framework provides advanced reliability features, MCP (Model Context Protocol) integration, and supports 20+ AI providers (OpenAI, Anthropic, Google, Azure, AWS Bedrock, Cohere, Mistral, Groq, and more — see "Model Providers and Configuration" below).

## AI Operational Guides

Consult the relevant guide before doing the work — these are operational, not optional.

- **`documents/ai/guides/feature.md`** — Use when adding a feature or non-trivial enhancement: a new public API, a new provider (Vector/Model/Storage/Tool/Embedding/OCR/Loader), a new agent type or prebuilt agent, a new policy in `safety_engine/`, a new RAG component, or a new public method/config knob on an existing class. Defines the four mandatory phases (Understand → Design → Implement → Verify), the eight cross-cutting aspect checklists, the hard gates, and the common anti-patterns.
- **`documents/ai/guides/refactor.md`** — Use when changing internal structure without changing observable behaviour: renaming, extracting, splitting an oversized module, removing dead code, mechanical migrations. Defines the four mandatory phases (Motivate & Scope → Characterize → Transform → Verify Behaviour Preserved), the per-phase hard gates, and anti-patterns specific to refactors. Cross-references `feature.md §4` for shared aspects.
- **`documents/ai/guides/bug-fix.md`** — Use when fixing a reported bug, failing test, traceback, or behaviour that contradicts a stated contract. Defines the four mandatory phases (Reproduce → Diagnose → Fix → Verify), with hard rules around root-cause discipline, regression tests, and minimal-diff scope. Cross-references `feature.md §4` for shared aspects.
- **`documents/ai/guides/testing.md`** — Use when deriving, writing, reviewing, or locking tests for any feature / refactor / bug-fix. Defines the four mandatory phases (Derive Scenarios → Generate RED Tests → Manual Review → Lock & Iterate via Code), with hard rules around user-driven scenario seeding, Serena + memory consultation, the manual review gate, and the immutability of locked tests. Complements `feature.md §4.5` (placement and types).
- **`documents/ai/guides/coding-standards.md`** — Always-on. How code in this repo is named, typed, structured, formatted, tested, and reviewed. Pure Python coding standard; framework-agnostic.
- **`documents/ai/guides/serena.md`** — Always-on. When and how to use Serena MCP for symbol and reference lookups, the read-only constraint on source code, the surface-what-you-found rule, and the optional Serena memory layer (currently unused; gitignored by default).
- **`documents/ai/guides/memory.md`** — Always-on. How Claude Code's auto memory works in this repo, where it lives, what auto-loads vs lazy-loads, when to consult it (recurring corrections, past test mistakes, similar prior requests), the surface-what-you-found rule, and the end-of-workflow MUST save reflection.
- **`documents/ai/guides/subagents.md`** — Always-on. When to dispatch subagents (heavy reads, separable investigations, long tasks, planning/review), when not to, and how to brief them.
- **`documents/ai/guides/new_prebuilt_agent_adding.md`** — Use when shipping a new prebuilt autonomous agent under `src/upsonic/prebuilt/<your_agent>/`. Canonical reference for the runtime/agent-class/template layering, `AGENT_REPO`/`AGENT_FOLDER` wiring to `PrebuiltAutonomousAgentBase`, and the `new_<X>(...)` high-level API conventions.
- **`documents/ai/guides/commit.md`** — Use before any `git commit`, `git push`, or history-rewriting command. Hard rule: never commit without explicit user approval.

### Default Pre-Work Consultation

Before any non-trivial task, the always-on guides compose into a single pre-work pass — Claude Code memory + Serena code lookup (and Serena memory if active). Surface findings together at the start of the reply so the user can see what informed the response, e.g.:

> *"From memory: prior feedback don't mock the DB. From Serena: existing similar handler at `src/upsonic/X.py:42`."*

Trivial work (single-line typo, comment edit) skips this and says so explicitly: *"Skipping memory / Serena lookup — single-line cosmetic edit."*

### Keep `documents/ai/explanation/` in sync with code

The files under `documents/ai/explanation/<subsystem>/<subsystem>.md` are treated as the authoritative description of each subsystem's behaviour contract. Whenever a code change alters an *observable* contract that an explanation doc asserts, resync the matching file in the same commit (or an immediate follow-up `docs: sync explanation/…` commit). The `documents/ai/guides/` files are Claude-operational process docs and only change when the *process* changes, not when a single contract does.

Triggers that almost always require a doc update:

- A public method / constructor signature changed (added, removed, renamed parameter; new default; new keyword-only requirement).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Upsonic/Upsonic](https://github.com/Upsonic/Upsonic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
