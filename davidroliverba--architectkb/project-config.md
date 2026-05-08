---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Language

All generated text must be in UK English.

## Model Preferences

At the start of each new question thread, ask which Claude model the user would like to use (unless they explicitly specify one). Use this guidance:

- **Haiku**: Fast research, straightforward queries, quick analysis
- **Sonnet**: Balanced for complex tasks, code review, planning
- **Opus**: Deep analysis, complex multi-step tasks, extended reasoning

### Model Selection Guide

Use this rubric for automatic model selection when users don't specify:

| Task Type                                                | Model                              | Rationale                            |
| -------------------------------------------------------- | ---------------------------------- | ------------------------------------ |
| Quick capture (`/daily`, `/meeting`, `/task`, `/person`) | Haiku                              | Speed, low cost, simple templates    |
| Research & exploration (read-only search)                | Haiku subagents                    | Parallel execution, isolated context |
| Document processing (`/pdf-to-page`, `/pptx-to-page`)    | Sonnet                             | Balanced extraction quality          |
| Code review, analysis, summarisation                     | Sonnet                             | Good reasoning, moderate cost        |
| Architecture decisions, ADRs                             | Opus                               | Deep analysis, extended thinking     |
| Complex multi-step orchestration                         | Sonnet coordinator + Haiku workers | Cost-effective parallelism           |
| Vendor evaluation (`/score-rfi`)                         | Sonnet                             | Consistent scoring, parallel agents  |
| Quality reports, audits                                  | Sonnet                             | Comprehensive analysis               |

**Cost Considerations:**

- Haiku: ~$1/$5 per MTok (input/output) - use for high-volume, simple tasks
- Sonnet: ~$3/$15 per MTok - default for most work
- Opus: ~$15/$75 per MTok - reserve for complex reasoning

**Prompt Caching:** For repeated operations, structure prompts with static context first to benefit from 90% cache savings after 2+ requests.

## Task Management

Always use the task list tools to track work items during sessions. For any request involving 2 or more steps, create tasks to track progress. This provides visibility of progress and ensures complex work is properly organised.

### Task Tool Reference

| Tool         | Purpose                                   | When to Use                          |
| ------------ | ----------------------------------------- | ------------------------------------ |
| `TaskCreate` | Create new task with subject, description | Starting any multi-step work         |
| `TaskUpdate` | Update status, add dependencies, modify   | Marking progress, setting blockers   |
| `TaskList`   | View all tasks and their status           | Checking what's pending/blocked      |
| `TaskGet`    | Retrieve full task details by ID          | Getting context before starting work |

### Task Workflow

```
1. TaskCreate - Create tasks for each major step
2. TaskUpdate (addBlockedBy) - Set dependencies between tasks
3. TaskUpdate (status: in_progress) - Mark task as started
4. [Do the work]
5. TaskUpdate (status: completed) - Mark task as done
6. TaskList - Check for next available task
```

### Status Values

- `pending` - Not started, waiting for dependencies
- `in_progress` - Currently being worked on
- `completed` - Successfully finished
- `deleted` - Removed (use sparingly)

### Best Practices

- Use `activeForm` for spinner text (present continuous: "Creating meeting note")
- Set `blockedBy` for tasks that depend on others completing first
- Update status to `in_progress` before starting work
- Always mark tasks `completed` when done (don't leave orphaned tasks)

## Repository Overview

This is an Obsidian vault template designed for **Solutions Architects** to manage professional knowledge effectively. The vault supports architecture decisions, project documentation, meeting notes, research ideas, and personal productivity tracking.

**Key Features:**

- Seven-pillar ontology (Entities, Nodes, Events, Views, Artifacts, Governance, Navigation)
- Architecture Decision Records (ADRs) with relationship tracking
- Incubator system for research and idea development
- Quality indicators for content freshness and confidence
- Claude Code skills for automation and AI-assisted workflows

### Knowledge Model: Seven Pillars

The vault is organised around a **node-centric ontology** with seven pillars:

| Pillar         | Nature             | Location          | Purpose                          |
| -------------- | ------------------ | ----------------- | -------------------------------- |
| **Entities**   | Things that exist  | Root              | Actors and objects in the world  |
| **Nodes**      | Units of knowledge | Root              | Understanding that persists      |
| **Events**     | Things that happen | Folders           | Temporal occurrences             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DavidROliverBA/ArchitectKB](https://github.com/DavidROliverBA/ArchitectKB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
