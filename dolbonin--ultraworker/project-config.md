---
trigger: always_on
description: This file is a complete guide for Claude Code to automatically orchestrate the Ultrawork system.
---

# CLAUDE.md - Ultrawork Orchestration Guide

This file is a complete guide for Claude Code to automatically orchestrate the Ultrawork system.

## System Overview

Ultrawork is a 4-stage approval workflow system that detects Slack mentions and automatically processes tasks.

```
Slack Mention → Context Exploration → TODO Creation → [Stage 1 Approval]
                                                            ↓
                        Tech Spec Writing ← ─ ─ ─ ─ ─ ─ ─ ─ ┘
                              ↓
                       [Stage 2 Approval] → Code Implementation → [Stage 3 Approval]
                                                                        ↓
                              Final Report ← ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
                                   ↓
                            [Stage 4 Approval] → Done
```

---

## Orchestration Rules

### Automatic Trigger Conditions

Claude should automatically invoke the appropriate skill/agent in the following situations:

| User Request | Skill to Execute | Example |
|--------------|------------------|---------|
| "Sync Slack" | `/sync-slack` | "Update channel list" |
| "Analyze this thread" | `/explore-context` | "Get context for C0123-1706500000" |
| "Create TODO" | `/create-todo` | "Create tasks from EXP-001" |
| "Write spec" | `/write-spec` | "Write tech spec for TASK-001" |
| "Approve" | `/approve` | "Approve TASK-001" |
| "Reject" | `/reject` | "Request revision for TASK-001" |
| "Create report" | `/report` | "Generate final report for TASK-001" |
| "Create image with Gemini and post to X" | `/gemini-x-image-post` | "Generate image with Gemini and post to X" |
| "Create/manage cron job" | `/manage-cronjob` | "Create a cronjob to check threads every morning" |

### Automatic Workflow Progression

After completing each stage, Claude should guide or automatically execute the next step:

```python
# Pseudocode
if current_stage == "exploration_complete":
    guide("Would you like to create a TODO? `/create-todo {exploration_id}`")

if current_stage == "todo_approved":
    if workflow_type == "full":
        guide("Writing tech spec: `/write-spec {task_id}`")
    else:
        guide("After completing work, generate report with `/report {task_id}`")

if current_stage == "tech_spec_approved":
    guide("Start code implementation. After completion, `/approve {task_id}`")

if current_stage == "code_work_approved":
    guide("Generating final report: `/report {task_id}`")

if current_stage == "final_report_approved":
    complete_processing()
```

### Execution Persistence and Progress Sharing (Important)

Regardless of model type (e.g., Opus 4.6), Claude should **never stop in an idle state after sending a progress message**.

#### Required Rules

1. **Progress Message = Start Signal**
   - Messages like "Proceeding", "Processing" are considered execution starts, not endings.
   - Continue immediately with the next task after such messages.

2. **Continue After Slack Sharing**
   - Even after posting intermediate results/reports to Slack, continue if there's remaining work.
   - Slack sharing is a checkpoint, not a termination condition.

3. **Maintain Progress Sharing Loop**
   - Briefly share current status and next action with the user after each major step.
   - After sharing, execute the next step immediately without waiting.

4. **Only Valid Termination Conditions**
   - All TODO/approval stages are complete
   - User explicitly requests to stop
   - External input/permission is absolutely required to proceed (in this case, clearly request the needed input)

#### Execution Loop Pseudocode

```python
send_progress_message()

while remaining_work_exists:
    execute_next_task()
    share_progress("current_status", "next_action")

    if slack_sharing_complete and remaining_work_exists:
        share_progress("Slack sharing complete", "continuing remaining work")
        continue

send_final_completion_report()
```

---

## Agent Session Tracking Rules

### Agent Roles

Agent roles transition according to workflow progression:

| Role | Description | Related Skills |
|------|-------------|----------------|
| `RESPONDER` | Initial response (immediately after mention detection) | - |
| `PLANNER` | Context exploration, TODO creation | `/explore-context`, `/create-todo` |
| `SPEC_WRITER` | Tech spec writing | `/write-spec` |
| `IMPLEMENTER` | Code implementation | Code writing/modification |
| `REPORTER` | Final reporting | `/report` |

### Role Transition Rules by Skill

| Skill | Start Role | End Role |
|-------|------------|----------|
| `/explore-context` | RESPONDER/Any | PLANNER |
| `/create-todo` | PLANNER | PLANNER |
| `/write-spec` | PLANNER | SPEC_WRITER |
| `/approve` (spec) | SPEC_WRITER | IMPLEMENTER |
| `/approve` (code) | IMPLEMENTER | REPORTER |
| `/report` | IMPLEMENTER | REPORTER |
| `/approve` (report) | REPORTER | (Session complete) |

### Session Creation Timing

Agent sessions are automatically created in the following situations:

1. **On Slack mention detection**: New thread mentions create new sessions
2. **On skill invocation**: Automatically created if no session exists
3. **Explicit creation**: Manual creation from dashboard

### Session States

| State | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DolbonIn/ultraworker](https://github.com/DolbonIn/ultraworker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
