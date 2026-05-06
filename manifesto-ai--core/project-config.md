---
trigger: always_on
description: > **Scope:** Guide for developers using Claude Code agents to work with Manifesto.
---

# Manifesto Agent Guide

> **Scope:** Guide for developers using Claude Code agents to work with Manifesto.

**Version:** 1.0
**Status:** Operational Guide
**Audience:** Human developers using Claude Code agents

---

## Document Purpose

This guide is for developers using Claude Code agents to work with the Manifesto codebase. It differs from `CLAUDE.md` in the following ways:

| Document | Audience | Purpose | Tone |
|----------|----------|---------|------|
| **CLAUDE.md** | LLM agents writing code | Constitutional constraints (binding) | Normative, restrictive |
| **AGENTS.md** | Developers using agents | Practical agent usage guide (advisory) | Educational, practical |

**When to read CLAUDE.md:** If you are an LLM agent modifying Manifesto code (this is automatically loaded).

**When to read AGENTS.md:** If you are a developer using Claude Code agents to work with Manifesto.

## Tooling Setup

If you want Codex to load Manifesto-specific guidance in another project:

1. Install `@manifesto-ai/skills` as a dev dependency
2. Run `npm exec manifesto-skills install-codex` or `pnpm exec manifesto-skills install-codex`
3. Restart Codex

This setup is explicit. `@manifesto-ai/skills` does not auto-register itself from `postinstall`.
For the full walkthrough, see the external `@manifesto-ai/skills` package README.

**Current contract note:** The canonical Snapshot block below reflects the current Core v4.0.0 contract. Accumulated `system.errors` and `appendErrors` are no longer part of the current Snapshot/SystemDelta surface.

---

## Quick Reference: Constitutional Constraints

For developers using agents, here's a condensed reference to the key constraints from `CLAUDE.md`. Agents will automatically follow these rules.

**Version:** 1.0 (based on CLAUDE.md)

### 0. Document Identity

The Constitution (`CLAUDE.md`) is a **binding operational constitution** for any LLM that interacts with the Manifesto codebase.

This is NOT documentation. This is NOT a tutorial. This is a **constraint specification**.

**Who it applies to:**
- Any LLM writing new code
- Any LLM refactoring existing code
- Any LLM adding features
- Any LLM modifying architecture

**Why violating it invalidates changes:**
Changes that violate this constitution produce systems that are NOT Manifesto-compliant. Partial compliance is not recognized. A system violating any single axiom, sovereignty rule, or forbidden pattern is NOT Manifesto.

**Normative hierarchy:**
1. Constitution (highest authority)
2. SPEC documents
3. FDR documents
4. Code
5. README (lowest authority)

When documents conflict, prefer higher-ranked sources.

**Commit and PR discipline:**
- If an agent creates or rewrites commits, each commit subject must use Conventional Commit format: `type(scope): summary` or `type: summary`.
- If an agent opens or updates a pull request, the pull request title must also use Conventional Commit format: `type(scope): summary` or `type: summary`.
- Allowed types are the repository-enforced set: `build`, `chore`, `ci`, `deps`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`.
- Treat non-conforming commit subjects as invalid output, not as a style preference, because CI rejects them and release automation relies on them.
- Treat non-conforming pull request titles as invalid output, not as a style preference, because CI rejects them at the pull request gate.

---

### 1. Core Engineering Axiom

**Manifesto computes what the Snapshot should become; Host makes declared work real.**

The fundamental equation is:

```
compute(schema, snapshot, intent) -> (snapshot', requirements, trace)
```

This equation is:
- **Pure**: Same inputs MUST always produce same outputs
- **Total**: MUST always return a result (never throws)
- **Traceable**: Every step MUST be recorded
- **Complete**: Snapshot MUST be the whole truth

---

### 2. Engineering Priorities (Ordered)

When priorities conflict, higher-ranked priorities MUST prevail.

1. **Determinism** — Same input MUST produce same output, always
2. **Accountability** — Every state change MUST be traceable to Actor + Authority + Intent
3. **Explainability** — Every value MUST answer "why?"
4. **Separation of Concerns** — Core computes, Host executes, SDK exposes runtime, Lineage records continuity, Governance authorizes legitimacy
5. **Immutability** — Snapshots and Lineage Worlds MUST NOT mutate after creation
6. **Schema-first** — All semantics MUST be expressible as JSON-serializable data
7. **Type safety** — Zero string paths in user-facing APIs
8. **Simplicity** — Minimum complexity for current requirements only

**Never trade a higher priority for a lower one.** Convenience, performance optimization, and developer preference are NOT valid reasons to violate determinism or accountability.

---

### 3. Package Boundary Rules

#### @manifesto-ai/core

**IS responsible for:**
- Pure semantic computation
- Expression evaluation
- Flow interpretation
- Patch generation
- Trace generation
- Schema validation

**MUST NOT:**
- Perform IO (network, filesystem, database)
- Access wall-clock time (`Date.now()` is forbidden)
- Execute effects
- Have mutable state
- Know about Host, SDK runtime assembly, Lineage, or Governance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manifesto-ai/core](https://github.com/manifesto-ai/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
