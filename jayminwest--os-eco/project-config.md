---
trigger: always_on
description: Guidance for AI agents (and humans) working in the **os-eco** meta-repo.
---

# AGENTS.md — os-eco

Guidance for AI agents (and humans) working in the **os-eco** meta-repo.

os-eco is the meta-project for the AI agent tooling ecosystem. It has **no
package.json, no source, and no test suite of its own** — it is a thin
coordination layer that tracks cross-cutting concerns across **eight
integrated tools**, each of which lives in its own independent sub-repo. (Two
further tools, **greenhouse** and **overstory**, have been archived — see
"Retired tools" below.) When an agent works here, it is almost always editing root-level docs,
governance scaffolding, or cross-repo coordination state — not tool source.

**Warren is the headline project.** It is the self-hostable control plane that
orchestrates ephemeral cloud agents and closes the autonomous loop
(GitHub → dispatch → PR). The rest of os-eco is the toolchain warren stands
on. Each piece works standalone too, but warren is where the ecosystem
narrative begins.

> Each sub-repo has its own `CLAUDE.md` / `AGENTS.md` with tool-specific
> conventions, architecture, and commands. This file documents only the
> **ecosystem-level** contract. For tool-internal work, read the sub-repo's
> own agent docs first.

## Ecosystem Overview

Tools group by role: **warren** is the flagship orchestrator. Underneath sit
the **substrate** (sandbox + coordination), the **context primitives** (what
agents read & write), and a **runtime** (single-agent execution). Alongside
the fleet sits a **standards** layer (readiness audit) that measures every
repo, warren included.

### Flagship — agent control plane

| Tool | CLI | npm | Purpose | Sub-repo |
|------|-----|-----|---------|----------|
| **Warren** | `warren` | `@os-eco/warren-cli` | Self-hostable control plane for ephemeral cloud agents; polls GitHub → dispatches runs → opens PRs. Absorbed greenhouse's autonomous-loop role 2026-05. | `warren/` |

### Substrate — sandbox & coordination

| Tool | CLI | npm | Purpose | Sub-repo |
|------|-----|-----|---------|----------|
| **Burrow** | `burrow` / `bw` | `@os-eco/burrow-cli` | OS-isolated sandbox runtime (bwrap on Linux, sandbox-exec on macOS). Warren embeds it per run; usable standalone. | `burrow/` |
| **Plot** | `plot` | `@os-eco/plot-cli` | Typed, queryable coordination object — binds seeds issues, mulch records, prompts, runs, and PRs around a unit of work. | `plot/` |

### Context primitives — what agents read & write

| Tool | CLI | npm | Purpose | Sub-repo |
|------|-----|-----|---------|----------|
| **Mulch** | `mulch` / `ml` | `@os-eco/mulch-cli` | Structured expertise management; the memory layer for agent workflows. | `mulch/` |
| **Seeds** | `sd` | `@os-eco/seeds-cli` | Git-native issue tracking with structured plans for LLM decomposition. | `seeds/` |
| **Canopy** | `cn` | `@os-eco/canopy-cli` | Prompt management & composition (inheritance, pinning, schema validation). | `canopy/` |

### Runtime — single-agent execution

| Tool | CLI | npm | Purpose | Sub-repo |
|------|-----|-----|---------|----------|
| **Sapling** | `sapling` / `sp` | `@os-eco/sapling-cli` | Headless coding agent with proactive context management. Alternative runtime warren can dispatch alongside Claude Code. | `sapling/` |

### Standards — fleet readiness & audit

| Tool | CLI | npm | Purpose | Sub-repo |
|------|-----|-----|---------|----------|
| **Trellis** | `trellis` | `@os-eco/trellis-cli` | Agentic-readiness audit & sync — scores repos against a versioned 9-category rubric (Level 1–5), detects canonical-config drift, keeps a fleet in standard. Pre-release. | `trellis/` |

### How they fit together

```
                              Warren  (cloud control plane)
                                 │
                  ┌──────────────┼──────────────┐
                  │              │              │
              Substrate       Context        Runtime
              ─────────       ───────        ───────
              Burrow          Mulch          Sapling
              Plot            Seeds
                              Canopy
        ────────────────────────────────────────────────
              Trellis  (standards — audits every repo)
```

- **Warren** polls GitHub for triaged issues, dispatches runs, opens PRs, and
  runs each agent inside a burrow sandbox.
- **Burrow** is the sandbox primitive — bwrap on Linux, sandbox-exec on macOS.
- **Plot** is the coordination object — a binder holding intent, attachments,
  and the event log for a unit of work.
- **Mulch** is the expertise layer — `ml record` / `ml prime` store and
  retrieve learnings across sessions.
- **Seeds** is the issue tracker — `sd create` / `sd ready` / `sd close`.
- **Canopy** is the prompt library — `cn emit` renders prompts for agents.
- **Sapling** is an alternative coding-agent runtime.
- **Trellis** is the standards layer — `trellis audit` scores agent-readiness;
  `trellis drift` / `trellis fleet` keep repos on canonical configs.

## Root-Level Commands

There is no build/test at the os-eco root. The commands an agent runs here are
coordination tooling and the per-repo session-prime rituals:

- `scripts/run-plan.sh <plan-id>` — sequentially work a seeds plan. For each
  open child of the plan, runs `claude -p` with
  `--permission-mode bypassPermissions`, logs to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jayminwest/os-eco](https://github.com/jayminwest/os-eco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
