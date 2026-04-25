---
trigger: always_on
description: Tauri 2 + SvelteKit 2 + Svelte 5 desktop app for AI agent interaction via Agent Client Protocol.
---

# Acepe

Tauri 2 + SvelteKit 2 + Svelte 5 desktop app for AI agent interaction via Agent Client Protocol.

## Project Vision

Acepe is a **production-grade Agentic Developer Environment**: a native workspace where developers run, supervise, compare, and ship work from multiple coding agents without giving up engineering discipline.

Trend toward:

- **Agent-agnostic architecture** — provider-specific quirks pushed to adapters and edges
- **Production-grade reviewability** — tool calls, permissions, diffs, checkpoints, PR-ready changes
- **Reliable long-running workflows** — serious project work, not one-off demos
- **Durable internal models** — add or replace agents without rewriting the product

## Quick Reference

**Package manager:** `bun` (not `npm`)

```bash
cd packages/desktop
bun run check      # TypeScript check (run after every TS change)
bun test           # Tests
bun run build      # Build
cargo clippy       # Rust lint (in src-tauri/)
```

**Artifact paths:**

| Type | Location |
|------|----------|
| Requirements | `docs/brainstorms/YYYY-MM-DD-<topic>-requirements.md` |
| Plans | `docs/plans/YYYY-MM-DD-<topic>-plan.md` |
| Learnings | `docs/solutions/` |

`docs/solutions/` stores documented solutions to past problems (bugs, best practices, workflow patterns), organized by category with YAML frontmatter (`module`, `tags`, `problem_type`). Relevant when implementing or debugging in documented areas.

## CE Workflow

Acepe uses the Compounding Engineering workflow as its engineering operating system. This is the **single source of truth** — `CLAUDE.md` defers here.

### Flow Diagram

```
                        ┌─────────────────────────────────────┐
                        │         Task arrives                │
                        └──────────────┬──────────────────────┘
                                       │
                              ┌────────▼────────┐
                              │  Trivial task?   │──── yes ──── Direct execution
                              └────────┬────────┘
                                       │ no
                              ┌────────▼────────┐
                              │  Requirements    │
                              │  already clear?  │──── no ───── /ce:brainstorm
                              └────────┬────────┘          docs/brainstorms/
                                       │ yes
                              ┌────────▼────────┐
                              │  Reviewed plan   │
                              │  already exists? │──── no ──┐
                              └────────┬────────┘          │
                                       │ yes               │
                                       │           ┌───────▼───────┐
                                       │           │  /ce:plan     │
                                       │           │  docs/plans/  │
                                       │           └───────┬───────┘
                                       │                   │
                                       │           ┌───────▼────────────┐
                                       │           │  /document-review  │◄──── MANDATORY GATE
                                       │           └───────┬────────────┘
                                       │                   │
                                       │         ┌─────────▼─────────┐
                                       │         │ Unresolved scope? │── yes ── loop to brainstorm
                                       │         └─────────┬─────────┘
                                       │                   │ no
                              ┌────────▼───────────────────▼──┐
                              │  Bug / behavior change /      │
                              │  non-trivial refactor?        │── yes ── TDD: failing test first
                              └────────┬──────────────────────┘
                                       │
                              ┌────────▼────────┐
                              │    /ce:work     │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │   /ce:review    │  (non-trivial work)
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  /ce:compound   │  (meaningful learning)
                              └─────────────────┘
```

### Entry-Point Decision Table

| Situation | Start at |
|-----------|----------|
| Scope, success criteria, or problem framing unclear | `/ce:brainstorm` |
| Requirements exist, no plan yet | `/ce:plan` |
| Reviewed plan exists, matches request | `/ce:work` |
| Bug fix or behavior change | TDD (failing test) then `/ce:work` |
| Non-trivial refactor | `/ce:plan` (refactors are not exempt) |
| Trivial, obvious, no durable plan needed | Direct execution |

### Phase Intent

| Phase | Purpose |
|-------|---------|
| `/ce:brainstorm` | Define **what** to build. Produces a requirements-quality artifact. |
| `/ce:plan` | Define **how** to build it. Decision-complete plan with files, tests, constraints. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flazouh/acepe](https://github.com/flazouh/acepe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
