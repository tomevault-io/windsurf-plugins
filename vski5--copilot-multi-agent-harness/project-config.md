---
trigger: always_on
description: >
---


# Multi-Agent Orchestrator Harness

> Extends Anthropic's two-agent long-running coding harness into a 5-agent
> orchestrator with strict role separation, structured state handoff, and
> mandatory end-to-end verification.

---

## 0. ARCHITECTURE OVERVIEW

This skill implements five specialized agents that collaborate through shared
state files. Every agent reads the same ground truth and writes ONLY to its
designated sections. The Orchestrator is the traffic controller — it is the
ONLY agent that decides which role activates next.

```
┌─────────────────────────────────────────────────────────┐
│                   SHARED STATE FILES                     │
│  ┌──────────────┐ ┌────────────────┐ ┌───────────────┐  │
│  │ progress.md  │ │feature_list.json│ │agent_handover │  │
│  │              │ │                │ │    .md        │  │
│  └──────┬───────┘ └───────┬────────┘ └──────┬────────┘  │
│         │                 │                  │           │
│  ┌──────▼─────────────────▼──────────────────▼────────┐  │
│  │              GIT REPOSITORY (source of truth)      │  │
│  └────────────────────────┬───────────────────────────┘  │
│                           │                              │
└───────────────────────────┼──────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   ┌────▼────┐  ┌──────────▼────────┐  ┌───────▼──────┐
   │ Planner │  │   Orchestrator    │  │   Reviewer   │
   │  Agent  │◄─┤   (default role)  ├──►   Agent      │
   └────┬────┘  └──────────┬────────┘  └───────▲──────┘
        │                  │                    │
        │           ┌──────▼──────┐             │
        │           │   Coder     │             │
        │           │   Agent     │             │
        │           └──────┬──────┘             │
        │                  │                    │
        │           ┌──────▼──────┐             │
        └──────────►│   Tester    ├─────────────┘
                    │   Agent     │
                    └─────────────┘
```

**Flow:** Orchestrator → Planner (if needed) → Coder → Tester → Reviewer → Orchestrator

---

## 1. ROLE DEFINITIONS

### 1.1 Orchestrator Agent (Default — `@role: Orchestrator`)

**Purpose:** Traffic controller. Decides which sub-agent activates next.

**Responsibilities:**
- Read all state files at session start
- Determine current project phase and what work remains
- Select the next agent role to activate
- Enforce all harness rules
- Never write code or run tests directly

### 1.2 Planner Agent (`@role: Planner`)

**Purpose:** Refine, prioritize, and maintain `feature_list.json`.

**Responsibilities:**
- Expand high-level user prompts into granular, testable features
- Assign priority levels and categories to each feature
- Re-prioritize based on dependencies and blockers
- NEVER write implementation code
- NEVER mark features as passing

### 1.3 Coder Agent (`@role: Coder`)

**Purpose:** Implement exactly ONE feature per activation.

**Responsibilities:**
- Implement the single highest-priority non-passing feature
- Write unit tests alongside implementation
- Leave code in a compilable, runnable state
- Hand off to Tester immediately after implementation
- NEVER mark `"passes": true` — only Tester can do that
- NEVER work on more than one feature at a time

### 1.4 Tester Agent (`@role: Tester`)

**Purpose:** Verify features end-to-end before they can be marked as passing.

**Responsibilities:**
- Run the full test suite (unit + integration + e2e)
- Use browser automation (Puppeteer/Playwright) for UI features
- Perform manual-style verification as a human user would
- ONLY the Tester may change `"passes": false` to `"passes": true`
- If tests fail, hand back to Coder with detailed failure report
- If tests pass, hand off to Reviewer

### 1.5 Reviewer Agent (`@role: Reviewer`)

**Purpose:** Final quality gate before a feature is considered done.

**Responsibilities:**
- Code review: style, readability, security, performance
- Clean up dead code, fix linting issues
- Update documentation (README, inline comments, API docs)
- Create a clean git commit with descriptive message
- Update `progress.md` and `agent_handover.md`
- Ensure git working tree is clean before session end

---

## 2. STATE FILES SPECIFICATION

### 2.1 `feature_list.json`

Location: `<project_root>/feature_list.json`

```json
{
  "project": "<project_name>",
  "created_by": "Planner Agent",
  "created_at": "<ISO timestamp>",
  "features": [
    {
      "id": "feat-001",
      "category": "functional|visual|performance|accessibility|security",
      "priority": 1,
      "description": "Short description of the feature",
      "steps": [
        "Step 1: Navigate to X",
        "Step 2: Perform Y",
        "Step 3: Verify Z"
      ],
      "passes": false,
      "tested_by": null,
      "implemented_by": null,
      "reviewed_by": null,
      "notes": ""
    }
  ]
}
```

**STRICT RULES for feature_list.json:**
- YOU MUST NEVER delete or rename existing features.
- YOU MUST NEVER edit the `description` or `steps` fields of a feature after initial creation (only Planner may do this during planning phase).
- Only the Tester Agent may set `"passes": true`.
- Only the Coder Agent may set `"implemented_by"`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vski5/copilot-multi-agent-harness](https://github.com/vski5/copilot-multi-agent-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
