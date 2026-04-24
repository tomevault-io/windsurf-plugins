---
trigger: always_on
description: > This file was converted from CLAUDE.md. OpenCode uses AGENTS.md for project instructions.
---

# Project Instructions for OpenCode

> This file was converted from CLAUDE.md. OpenCode uses AGENTS.md for project instructions.
> See: https://opencode.ai/docs/rules/

# Deciduous - Decision Graph Tooling

Decision graph tooling for AI-assisted development. Track every goal, decision, and outcome. Survive context loss. Query your reasoning.

---

## The Two Modes

Every system has two stories:

| Mode | Question | Skill |
|------|----------|-------|
| **Now** | "How does this work?" | `/pulse` |
| **History** | "How did we get here?" | `/narratives` → `/archaeology` |

**Now mode** maps current design as decisions. **History mode** captures evolution and pivots.

---

## Skills Overview

### /pulse - Map Current Design

Take the pulse of a system - what decisions define how it works TODAY.

```bash
# Start with a goal
deciduous add goal "API rate limiting behavior" -c 90

# Map options (possible approaches from the goal)
deciduous add option "Identify users by auth token" -c 85
deciduous link 1 2 -r "possible_approach"

deciduous add option "Use IP-based rate limiting" -c 85
deciduous link 1 3 -r "possible_approach"
```

**Output:** Decision tree of the current model (goal -> options -> decisions).

### /narratives - Understand Evolution

Understand how the system evolved. Narratives are conceptual, not tied to commits.

1. Look at the current system
2. Ask "how did this get this way?"
3. Infer narratives from the design
4. Find evidence (commits, PRs, docs)
5. Identify pivots - where the model changed

**Output:** `.deciduous/narratives.md` with evolution stories.

### /archaeology - Structure for Query

Transform narratives into a queryable graph.

| Narrative Element | Graph Node |
|-------------------|------------|
| Title | `goal` |
| Possible approach | `option` |
| Choosing an approach | `decision` |
| What was learned | `observation` |
| **PIVOT** | `revisit` |

**Output:** Connected graph with Now ← revisit ← History.

---

## Slash Commands

All slash commands are bootstrapped by `deciduous init` and updated by `deciduous update`.

| Command | Purpose |
|---------|---------|
| `/decision` | Manage decision graph - add nodes, link edges, sync |
| `/recover` | Recover context from decision graph on session start |
| `/work` | Start a work transaction - creates goal node before implementation |
| `/document` | Generate comprehensive documentation for a file or directory |
| `/build-test` | Build the project and run the test suite |
| `/serve-ui` | Start the decision graph web viewer |
| `/sync-graph` | Export decision graph to GitHub Pages |
| `/decision-graph` | Build a decision graph from commit history (archaeology) |
| `/sync` | Multi-user sync - pull events, rebuild, push |

---

## The Revisit Node

When a design approach is abandoned and replaced:

```
[Old Decision] ──► [Observation: why it failed] ──► [REVISIT] ──► [New Decision]
```

The revisit captures:
- WHAT is being reconsidered
- WHY (linked observations)
- Connects old approach to new

```bash
deciduous add observation "JWT too large for mobile"
deciduous add revisit "Reconsidering token strategy"
deciduous link <observation> <revisit> -r "forced rethinking"
deciduous status <old_decision> superseded
```

---

## Node Status

| Status | Meaning |
|--------|---------|
| `active` | Current truth |
| `superseded` | Replaced by newer approach |
| `abandoned` | Tried and rejected |

```bash
# Now mode - only active
deciduous nodes --status active

# History mode - everything
deciduous nodes

# Pivot points
deciduous nodes --type revisit
```

---

<!-- deciduous:start -->
## Decision Graph Workflow

**THIS IS MANDATORY. Log decisions IN REAL-TIME, not retroactively.**

### Available Slash Commands

| Command | Purpose |
|---------|---------|
| `/decision` | Manage decision graph - add nodes, link edges, sync |
| `/recover` | Recover context from decision graph on session start |
| `/work` | Start a work transaction - creates goal node before implementation |
| `/document` | Generate comprehensive documentation for a file or directory |
| `/build-test` | Build the project and run the test suite |
| `/serve-ui` | Start the decision graph web viewer |
| `/sync-graph` | Export decision graph to GitHub Pages |
| `/decision-graph` | Build a decision graph from commit history |
| `/sync` | Multi-user sync - pull events, rebuild, push |

### Available Skills

| Skill | Purpose |
|-------|---------|
| `/pulse` | Map current design as decisions (Now mode) |
| `/narratives` | Understand how the system evolved (History mode) |
| `/archaeology` | Transform narratives into queryable graph |

### The Node Flow Rule - CRITICAL

The canonical flow through the decision graph is:

```
goal -> options -> decision -> actions -> outcomes
```

- **Goals** lead to **options** (possible approaches to explore)
- **Options** lead to a **decision** (choosing which option to pursue)
- **Decisions** lead to **actions** (implementing the chosen approach)
- **Actions** lead to **outcomes** (results of the implementation)
- **Observations** attach anywhere relevant
- Goals do NOT lead directly to decisions -- there must be options first
- Options do NOT come after decisions -- options come BEFORE decisions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [notactuallytreyanastasio/deciduous](https://github.com/notactuallytreyanastasio/deciduous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
