---
trigger: always_on
description: Staged execution pipeline with plan→prd→exec→verify→fix loop, dual reasoning (think/act), parallel sub-agent spawning, and resumable state. Inspired by OMC and Chat2Graph. Use for any complex multi-step task that benefits from structured decomposition, verification, and automatic retry.
---


# Pipeline Orchestrator

A structured execution engine for complex, multi-step tasks. Decomposes work into a dependency graph, writes acceptance criteria, executes with parallel sub-agents, verifies results, and auto-retries failures — all with resumable state.

## When to Use This Skill

**Trigger phrases:**
- "Run pipeline: ..."
- "Pipeline status"
- "Resume pipeline ..."
- "Cancel pipeline ..."

**Good fit when:**
- Task has 3+ distinct sub-tasks
- Sub-tasks have dependencies (some must finish before others start)
- Quality matters — you want verification, not just execution
- Task is complex enough that a single-shot attempt risks missing pieces
- You want parallel execution to save time

**Skip this when:**
- Simple single-step task (just do it)
- Pure conversation / Q&A
- Task is exploratory with no clear deliverables

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    PIPELINE                          │
│                                                     │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌────────┐      │
│  │ PLAN │──▶│ PRD  │──▶│ EXEC │──▶│ VERIFY │      │
│  └──────┘   └──────┘   └──────┘   └────────┘      │
│                                        │            │
│                                   pass │ fail       │
│                                   ▼    ▼            │
│                                 DONE  ┌─────┐      │
│                                       │ FIX │──┐   │
│                                       └─────┘  │   │
│                                         ▲      │   │
│                                         └──────┘   │
│                                      (max 3 loops) │
│                                                     │
│  State: ~/.openclaw/workspace/state/pipeline-{id}/  │
└─────────────────────────────────────────────────────┘
```

Each stage uses the **Dual Reasoning** pattern: Think first (analyze, plan, assess risk), then Act (execute the plan, no improvisation).

---

## Pipeline Stages

### Stage 1: PLAN

**Purpose:** Decompose the task into a directed acyclic graph (DAG) of sub-tasks.

**Think Phase:**
Before creating the plan, reason through:
- What are ALL the discrete pieces of work?
- What depends on what? (draw the DAG mentally)
- Which tasks can run in parallel?
- What's the complexity of each? (simple: <5 min, medium: 5-20 min, complex: 20+ min)
- What could go wrong at each step?
- Are there any implicit dependencies the user didn't mention?

**Act Phase:**
Write `state/pipeline-{id}/plan.json` with this structure:

```json
{
  "id": "pipeline-abc123",
  "task": "Original task description from user",
  "created": "2026-04-04T04:00:00Z",
  "tasks": [
    {
      "id": "t1",
      "name": "Short descriptive name",
      "description": "What needs to be done",
      "dependencies": [],
      "complexity": "simple|medium|complex",
      "parallelGroup": 1,
      "contextFiles": ["path/to/relevant/file.js"]
    },
    {
      "id": "t2",
      "name": "Another task",
      "description": "Depends on t1 output",
      "dependencies": ["t1"],
      "complexity": "medium",
      "parallelGroup": 2,
      "contextFiles": []
    }
  ],
  "parallelGroups": {
    "1": ["t1", "t3"],
    "2": ["t2", "t4"],
    "3": ["t5"]
  },
  "estimatedDuration": "~15 minutes",
  "risks": ["Risk 1", "Risk 2"]
}
```

**Key rules:**
- `parallelGroup` numbers define execution waves. Group 1 runs first, then group 2 after group 1 completes, etc.
- Tasks in the same `parallelGroup` MUST have no dependencies on each other.
- `dependencies` lists task IDs that must complete before this task starts.
- `contextFiles` — only include files the sub-agent actually needs. Less is more.

**Advisory model:** Sonnet (text generation, no code execution needed).

---

### Stage 2: PRD (Product Requirements Document)

**Purpose:** Define explicit, testable acceptance criteria for every sub-task.

**Think Phase:**
For each task in the plan, reason through:
- What does "done" look like, specifically?
- How will we verify this programmatically vs. manually?
- What edge cases should the acceptance criteria cover?
- Are the criteria tight enough to catch a bad implementation?

**Act Phase:**
Write `state/pipeline-{id}/prd.json`:

```json
{
  "pipelineId": "pipeline-abc123",
  "requirements": [
    {
      "taskId": "t1",
      "taskName": "Short descriptive name",
      "acceptanceCriteria": [
        "File X exists at path Y",
        "Function Z returns correct output for inputs [a, b, c]",
        "No lint errors in modified files"
      ],
      "verificationMethod": "automated|manual|hybrid",
      "verificationCommands": [
        "test -f path/to/file.js && echo PASS || echo FAIL",
        "node -e \"require('./module').test()\" 2>&1"
      ],
      "verificationChecklist": [
        "Output matches expected format",
        "No regressions in existing functionality"
      ]
    }
  ]
}
```

**Key rules:**
- Every acceptance criterion must be binary: pass or fail. No "looks good enough."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cat-tj/pipeline-orchestrator](https://github.com/Cat-tj/pipeline-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
