---
trigger: always_on
description: >-
---


> *授人以鱼，不如以渔。* ——《淮南子·说林训》
> 技能教会智能体怎么做事，这份文件教会技能怎么诞生。

# Skill Builder — AI Agent Skills Creation Guide

> **Position**: meta tier skill. Output = new skill files (SKILL.md + openai.yaml), not business code.
>
> **Deployment**: `cp -r SKILL-BUILDER-GUIDE/ target-project/.claude/skills/skill-builder/` — then auto-loaded as `/skill-builder`.
>
> **Progressive loading**: This file is L2. Deep methodology in `references/` — loaded only when a phase or scenario triggers them (§2.7).

## Trigger Conditions

- Creating a skill system for a project
- "create skill" / "generate project-specific skill" / "skill template"
- "model tier" / "L0 delegation" / "skill validation"
- "openai.yaml" / "frontmatter spec"
- "how to decompose" / "delegation pattern"

---

## 1. Core Concepts

### 1.1 Dual-Axis Model

Every skill is defined on **two independent, orthogonal dimensions**:

| | Execution (model_tier) | Composition (skill_tier) |
|--|------|------|
| Question | Who executes? | Where in the dependency graph? |
| Values | L0 / L1 / L2 / L3 | meta / planning / functional / atomic |

**Execution**: L0=Haiku (mechanical) · L1=Sonnet (bounded) · L2=Sonnet/Opus (multi-step) · L3=Opus (architectural)

**Composition**: meta=creates skills · planning=task routing · functional=multi-step routines · atomic=single source

**Consistency**: L0+meta invalid · L0+planning invalid · L0+functional not recommended · L0+atomic valid

### 1.2 Model Tier Routing

| Tier | Cognitive Load | Model | Typical Tasks |
|:----:|---------|---------|---------|
| **L0** | Mechanical | Haiku | File lookup, info query, command exec, static tracing |
| **L1** | Bounded | Sonnet | Single-module changes, narrow search, bounded implementation |
| **L2** | Reasoning | Sonnet/Opus | Root cause diagnosis, cross-module implementation |
| **L3** | Strategic | Opus | Architectural decisions, security audits |

**Mandatory rule**: L0 tasks → Haiku. Main model executing L0 = 5-15x token waste.

| L0 Task | Example | Delegate To |
|---------|------|---------|
| File lookup | "Where is the entry file" | Haiku |
| Info query | "What framework version" | Haiku |
| Command exec | "Run deploy script" | Haiku |
| Mechanical edit | "Update version number" | Haiku |

**Upgrade triggers**: ≥2 correction rounds failed → upgrade model. Same code ≥3 modifications → stop, re-analyze. 3 rounds without narrowing → upgrade to re-diagnose.

### 1.3 Skill Type Catalog

| Skill Type | Execution | Composition | Required Sections | Handoff | Template |
|---------|:--:|:--:|------|---------|------|
| Standards (dev) | L1 | atomic | Tech stack, layered arch, code standards | → code-map (new files ≥3) | `templates/example-dev/` |
| Code Map (code-map) | **L0** | atomic | Dir structure, quick lookup, routes | → dev (conventions lookup) | `templates/example-code-map/` |
| Workflow (workflow) | L1 | functional | Phase→input→steps→output, checklists | → change-model (changes ≥2 modules) | `templates/example-workflow/` |
| Change Model (change-model) | L1 | functional | WHY/WHAT/HOW/VALIDATION, call-chain check. **Implicit trigger**: at CONFIRM phase, Agent asks whether to generate a change report for the completed work. | → call-chain (changes ≥3 layers) | `references/change-model.md` |
| Call-Chain (call-chain) | L1 | functional | Tracing method, type checking, final-call checklist | → change-model (new change found) | `templates/example-call-chain/` |
| Scripts (scripts) | L0 | atomic | Script inventory, execution flow, error handling. **Min threshold**: ≥5 standalone scripts or ≥1 complex deployment pipeline. | — | — |
| Delegation (delegation) | L1 | planning | Decomposition criteria, model routing | → any (as needed) | `references/delegation.md` |

### 1.4 Skill Selection — Decision Guide

More skills ≠ better. During initialization, the agent guides the human through three questions. The answers naturally lead to a skill set. No preset combos.

**Three Decision Questions**:

| # | Question | Options |
|---|------|------|
| 1 | What's your project scope? | Quick fixes / solo prototype · Ongoing product / small team · Multi-module / multi-service |
| 2 | How autonomous should the AI be? | Passive lookup only · Independent within a module · Cross-module analysis, proactive reporting |
| 3 | Do you have existing conventions documented? | No, extract from code · Yes, use them |

**How answers map to skills**:

```
dev — every project needs it
code-map — recommended when project scope > "solo prototype"
workflow — recommended when AI autonomy > "passive"
change-model — recommended when AI autonomy = "cross-module"
call-chain — multi-service + cross-module autonomy
delegation — team >1 or cross-module autonomy
```

**Quick-start or full pipeline**: The human can always say "quick start" (skip deep scan and validation, ≤5 min) or "full pipeline". The agent doesn't presume which is better.

Full methodology: `references/decision-guide.md`.

### 1.5 Skill Chains

Skills shouldn't trigger in isolation. After completing one skill, recommend the next based on output characteristics.

| Entry Skill | After Completion, Recommend | Trigger Condition |
|---------|-----------|---------|
| dev | `{project}-code-map` | new files ≥3 |
| dev | `{project}-workflow` | non-standard directory structure found |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ang-XWBWZ/SKILL-BUILDER-GUIDE-SKILL](https://github.com/ang-XWBWZ/SKILL-BUILDER-GUIDE-SKILL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
