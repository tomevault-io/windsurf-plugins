---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Code-foundations is a Claude Code plugin providing software engineering skills based on *Code Complete* (McConnell) and *A Philosophy of Software Design* (Ousterhout). It includes a building workflow with gated phases (BUILD, REVIEW, orchestrator commit) and an experimental code review system.

## Architecture

### Skill Families

| Family | Prefix | Focus |
|--------|--------|-------|
| Code Complete | `cc-*` | Process rigor, metrics, checklists |
| APOSD | `aposd-*` | Design philosophy, complexity reduction |
| GoF Design Patterns | `gof-*` | 23 Gang of Four patterns, decision trees, structural recipes |
| Clean Architecture | `ca-*` | System-level boundaries, SRP-by-actor, dependency direction |
| Legacy Code | `welc-*` | Safely modifying untested code (conditional, invoked from cc-refactoring-guidance) |

### Directory Structure

- `skills/` - Individual skill definitions (SKILL.md + checklists.md)
- `commands/` - User-invocable commands (slash commands)
- `agents/` - Agent templates (build-agent, post-gate-agent, debug-agent)
- `references/` - Shared reference materials
- `docs/` - Case study examples

### Code Review System

**Single entry point:** `/code-foundations:review`

**Two presets:**

**Sanity Flow (--sanity):** 14 core checks, intelligent batching
```
┌────────────┐   ┌─────────────┐   ┌───────────┐   ┌───────────────┐
│ EXTRACTION │ → │ ORCHESTRATE │ → │ CHECKING  │ → │ INVESTIGATION │
│  (haiku)   │   │  (sonnet)   │   │ (sonnet)  │   │   (sonnet)    │
└────────────┘   └─────────────┘   └───────────┘   └───────────────┘
      ↓                 ↓                 ↓                  ↓
  1 per 5 files   • Triage files    1 agent per      1 agent per
  Extract units   • Smart batching  batch, runs      5 findings,
  + diffs                           14 core checks   provides fixes
```

**PR Flow (--pr):** 546 checks, prefix-based grouping
```
┌────────────┐   ┌─────────────┐   ┌───────────┐   ┌─────────────┐   ┌───────────────┐
│ EXTRACTION │ → │ CHECK ORCH  │ → │ CHECKING  │ → │ ORCHESTRATE │ → │ INVESTIGATION │
│  (haiku)   │   │   (haiku)   │   │ (sonnet)  │   │   (haiku)   │   │   (sonnet)    │
└────────────┘   └─────────────┘   └───────────┘   └─────────────┘   └───────────────┘
      ↑                ↑                 ↑                ↑                  ↑
   Batch by        Group by         1 agent per      Dedupe &          1 agent per
   files (5)       ID prefix        prefix group     batch             5 findings
                   (GC-, EH-...)    + skills
```

| Preset | Checks | Use Case |
|--------|--------|----------|
| `--sanity` | 14 core (consensus-distilled) | Pre-commit sanity |
| `--pr` | 546 (8 checklists) | Full PR review |

### Skill Checklist Counts

| Skill | Checks |
|-------|--------|
| cc-defensive-programming | 41 |
| aposd-simplifying-complexity | 44 |
| aposd-reviewing-module-design | 42 |
| code-clarity-and-docs | 87 |
| cc-control-flow-quality | 104 |
| aposd-verifying-correctness | 33 |
| cc-quality-practices | 125 |
| performance-optimization | 70 |
| **Total (PR preset)** | **546** |

### Review Execution Flow

1. **Load preset** → Parse checklists and skills
2. **Validate** → Check checklist paths exist, warn on missing skills
3. **Get target** → Ask for diff args (staged, unstaged, branch)
4. **Create phase tasks** → TaskCreate for each phase (enforces flow)
5. **Extraction** → Parallel haiku agents (batch by files)
6. **Check Orchestrate** → Single haiku agent parses checklists, groups by ID prefix
7. **Checking** → Parallel sonnet agents use `add-finding.sh` to record results
8. **Orchestrate** → Single haiku agent batches findings
9. **Investigation** → Parallel sonnet agents use `add-verdict.sh` to record verdicts + fixes
10. **Summary** → Display results, offer actions (open dashboard, fix all)

**Phase enforcement via TaskCreate/TaskUpdate** - agent cannot skip phases.
**Schema enforcement via bash scripts** - `add-finding.sh` and `add-verdict.sh` validate all inputs.

### Development Workflows

**Choose based on scope:**

| Situation | Command | Ceremony |
|-----------|---------|----------|
| Bug investigation | `/code-foundations:debug` | Minimal |
| Technical uncertainty | `/code-foundations:prototype` | Minimal |
| Feature needs planning | `/code-foundations:whiteboarding` | Medium |
| Executing approved plan | `/code-foundations:building` | Full |

### Prototype → Whiteboarding → Building Workflow

Three-stage pattern for feature development:

| Command | Purpose | Output |
|---------|---------|--------|
| `/code-foundations:prototype` | Prove feasibility with minimum code | Prototype log in `docs/prototypes/` |
| `/code-foundations:whiteboarding` | Discovery-oriented brainstorming | Plan file in `docs/plans/` |
| `/code-foundations:building` | Checklist-based execution | Working code + tests |

**Full Flow:**
```
/code-foundations:prototype "can I show a notification?"
  → One question to prove
  → Minimum code (~50 lines max)
  → Binary answer: YES/NO/PARTIAL
  → Capture learnings to docs/prototypes/

        ↓ (if feasible)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanthedev/code-foundations](https://github.com/ryanthedev/code-foundations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
