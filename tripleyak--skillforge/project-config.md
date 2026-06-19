---
trigger: always_on
description: Intelligent skill router, proactive advisor, and creator. Analyzes ANY input to recommend existing skills, improve them, or create new ones. Adds proactive Context Skill Advisor suggestions from session, project, and personal context using user-controlled Proactivity Levels.
---


# SkillForge 5.2 - Intelligent Skill Router, Advisor & Creator

Analyzes ANY input to find, improve, or create the right skill. It can also proactively surface evidence-backed skill suggestions through the Context Skill Advisor.

---

## Quick Start

**Any input works.** SkillForge will intelligently route to the right action:

```
# These all work - SkillForge figures out what you need:

SkillForge: create a skill for automated code review
→ Creates new skill (after checking no duplicates exist)

help me debug this TypeError
→ Recommends ErrorExplainer skill (existing)

improve the testgen skill to handle React components better
→ Enters improvement mode for TestGen

do I have a skill for database migrations?
→ Recommends DBSchema, database-migration skills

TypeError: Cannot read property 'map' of undefined
→ Routes to debugging skills (error detected)
```

---

## Triggers

### Creation Triggers
- `SkillForge: {goal}` - Full autonomous skill creation
- `create skill` - Natural language activation
- `design skill for {purpose}` - Purpose-first creation
- `ultimate skill` - Emphasize maximum quality
- `skillforge --plan-only` - Generate specification without execution

### Routing Triggers (NEW in v4.0)
- `{any input}` - Analyzes and routes automatically
- `do I have a skill for` - Searches existing skills
- `which skill` / `what skill` - Recommends matching skills
- `improve {skill-name} skill` - Enters improvement mode
- `help me with` / `I need to` - Detects task and routes

### Proactive Advisor Triggers (NEW in v5.2)
- Session start - Show pending suggestions from the Advisory Queue
- Advisor Checkpoint - Evaluate task shifts, repeated friction, and before-final moments
- Scheduled Background Advising - Run local scheduled advisor checks based on Proactivity Level
- `SkillForge advice` / `context advisor` - Inspect or manage queued suggestions

| Input | Output | Quality Gate |
|-------|--------|--------------|
| Any input | Triage → Route → Action | Phase 0 analysis |
| Explicit create | New skill | Unanimous panel approval |
| Task/question | Skill recommendation | Match confidence ≥60% |
| Advisor checkpoint | Evidence-backed suggestion | Proactivity Level threshold |

---

## Process Overview

```
ANY USER INPUT
(prompt, error, code, URL, question, task request)
    │
    ▼
┌─────────────────────────────────────────────────────┐
│ Phase 0: SKILL TRIAGE (NEW)                         │
│ • Classify input type (create/improve/question/task)│
│ • Scan 250+ skills in ecosystem                     │
│ • Match against existing skills with confidence %   │
│ • Route to: USE | IMPROVE | CREATE | COMPOSE        │
├─────────────────────────────────────────────────────┤
│         ↓ USE_EXISTING    ↓ IMPROVE      ↓ CREATE   │
│      [Recommend]      [Load & Enhance] [Continue]   │
└─────────────────────────────────────────────────────┘
    │ (if CREATE_NEW or IMPROVE_EXISTING)
    ▼
┌─────────────────────────────────────────────────────┐
│ Phase 1: DEEP ANALYSIS                              │
│ • Expand requirements (explicit, implicit, unknown) │
│ • Apply 11 thinking models + Automation Lens        │
│ • Question until no new insights (3 empty rounds)   │
│ • Identify automation/script opportunities          │
├─────────────────────────────────────────────────────┤
│ Phase 2: SPECIFICATION                              │
│ • Generate XML spec with all decisions + WHY        │
│ • Include scripts section (if applicable)           │
│ • Validate timelessness score ≥ 7                   │
├─────────────────────────────────────────────────────┤
│ Phase 3: GENERATION                                 │
│ • Write SKILL.md with fresh context                 │
│ • Generate references/, assets/, and scripts/       │
├─────────────────────────────────────────────────────┤
│ Phase 4: SYNTHESIS PANEL                            │
│ • 3-4 Opus agents review independently              │
│ • Script Agent added when scripts present           │
│ • All agents must approve (unanimous)               │
│ • If rejected → loop back with feedback             │
└─────────────────────────────────────────────────────┘
    │
    ▼
Production-Ready Agentic Skill
```

**Key principles:**
- **Phase 0 prevents duplicates** - Always checks existing skills first
- Evolution/timelessness is the core lens (score ≥ 7 required)
- Every decision includes WHY
- Zero tolerance for errors
- Autonomous execution at maximum depth
- Scripts enable self-verification and agentic operation

### Tool Escalation Policy

Start with least privilege (`Read`, `Glob`, `Grep`, `Write`, `Edit`).

Only add higher-risk tools when explicitly required:
- `Bash` for deterministic local scripts that cannot be replaced with file edits
- `WebFetch` / `WebSearch` only when external facts are required
- `Task` only for true parallel sub-agent orchestration

---

## Commands

| Command | Action |
|---------|--------|
| `SkillForge: {goal}` | Full autonomous execution |
| `SkillForge --plan-only {goal}` | Generate specification only |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tripleyak/SkillForge](https://github.com/tripleyak/SkillForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
