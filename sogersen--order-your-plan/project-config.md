---
trigger: always_on
description: Prompt Expansion / Plan Document Generator: User provides a simple requirement → Domain research → Interactive refinement → Creative expansion → Outputs an ultra-detailed execution plan document (.md). Based on the Milk Tea Ordering + Hyper-Concurrent Questioning methodology. A Plan Mode alternative for any AI Agent.
---


# Prompt Expansion / Plan Document Generator

> **In one sentence**: Forges a user's fragment of inspiration, through research, interaction, and creative expansion, into an ultra-detailed plan document that an AI Agent can directly execute.

## What problem does this skill solve?

A user's thinking is jumpy and intuitive — "make a chaos pendulum demo page", "build a data visualization tool", "write an automatic backup script".

What an AI Agent needs is precise and structured — tech stack, file structure, feature list, UI specifications, acceptance criteria, prohibitions.

**This skill is that bridge.**

| Input | Output |
|------|------|
| "Make a chaos pendulum demo page" | A 3000+ word detailed execution plan .md |
| "I want to make a data dashboard" | A complete document including architecture, tech selection, UI specifications, acceptance criteria |
| "Help me write an automation script" | Feature list, error handling strategy, input/output specifications, test plan |

### Differences from other skills

| Skill | What it does | Output | Characteristics |
|------|--------|------|------|
| **prompt-refiner** | Vague instruction → Clarify → Execute | Direct execution result | Instant task, execute immediately after clarification |
| **claude-code-goal-writer** | Rough input → Goal document | /goal command + goal.md | Designed specifically for Claude Code |
| **ai-dev-flow** | Vague idea → Research → Routing → Delivery | Final product | Complete development pipeline |
| **milk-tea-ordering** | Requirement guidance → Preference learning | Confirmation form → Execution | Includes preference memory and combo system |
| **This skill** | Simple inspiration → Research → Refine → Creative expansion | **Pure plan document .md** | **Planning only, no execution** |

**Core difference**: This skill does not execute any code or perform any implementation. Its sole output is an **extremely detailed plan document** that can be handed to any AI Agent (Claude Code, Cursor, Windsurf, Copilot, Hermes itself) for execution.

---

## Core Methodology: Three-Layer Alchemy

```
User's fragment of inspiration (rough ore)
         │
         ▼
┌─────────────────┐
│  Layer 1: Research  │  ← Hyper-concurrent questioning: What is this thing? How to do it?
│  (Research)     │     Three-pronged parallel drilling, build domain knowledge
└────────┬────────┘
         ▼
┌─────────────────┐
│  Layer 2: Refine    │  ← Milk tea ordering: Ask step by step what the user wants
│  (Refine)       │     One question at a time, interactive options
└────────┬────────┘
         ▼
┌─────────────────┐
│  Layer 3: Forge     │  ← Creative expansion: AI proactively supplements what the user didn't think of
│  (Forge)        │     Smelt all information into an ultra-detailed plan
└────────┬────────┘
         ▼
An extremely detailed execution plan document (.md)
```

---

## Phase 0: Reception and Parsing

After receiving the user's requirement, first perform a quick parse:

### 0.1 Identify Requirement Type

| User says | Identified as | Initial domain |
|--------|--------|---------|
| "Make a website/web page/page" | web-project | Frontend development |
| "Make a tool/script/program" | software-tool | Software development |
| "Make a visualization/dashboard" | data-viz | Data visualization |
| "Make a simulation/emulation" | simulation | Scientific computing |
| "Make a system/platform" | system-platform | System architecture |
| "Write a paper/document" | document | Academic writing |
| "Make a hardware/circuit" | hardware | Embedded/Hardware |
| "Make a game" | game | Game development |
| "Help me plan" | planning | Project management |
| Other | general | General |

### 0.2 Identify Vagueness

Evaluate the vagueness of user input across 3 dimensions:

| Dimension | Clear signal | Vague signal |
|------|---------|---------|
| **Goal** | "Make a blog with login functionality" | "Make a website" |
| **Constraints** | "Use React, deploy to Vercel" | "Make it look nice" |
| **Scope** | "3 pages: Home, Articles, About" | "Something roughly like that" |

**Vagueness Score** (0-3, higher = more vague):
- 0: Goal + Constraints + Scope all clear → Skip research, go directly to refinement confirmation
- 1: Missing 1 item → Light research + quick refinement
- 2: Missing 2 items → Full research + standard refinement
- 3: All vague → Deep research + complete refinement + creative expansion

### 0.3 Startup Message

```
OK, I'll help you turn this idea into a detailed execution plan.

You said "[user's original words]", let me first research this domain, then confirm details with you step by step.
```

---

## Phase 1: Domain Research (Hyper-Concurrent Questioning Engine)

> **Core principle**: Before asking the user, let the AI itself figure out two things first —
> 1. What this thing **is** and **how to do it** (technical knowledge)
> 2. How to **plan** this thing and **which dimensions need user confirmation** (planning knowledge)
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SOGERSEN/order-your-plan](https://github.com/SOGERSEN/order-your-plan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
