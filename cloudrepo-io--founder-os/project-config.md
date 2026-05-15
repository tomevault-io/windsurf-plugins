---
trigger: always_on
description: {{PRODUCT_DESCRIPTION}}
---

# {{COMPANY_NAME}}

{{PRODUCT_DESCRIPTION}}

## Company Context

- **Product**: {{PRODUCT_NAME}}
- **Type**: {{COMPANY_TYPE}}
- **Tech Stack**: {{PRIMARY_LANGUAGE}} / {{FRONTEND_FRAMEWORK}} / {{BACKEND_FRAMEWORK}} / {{DATABASE}} / {{CLOUD_PROVIDER}}

## Differentiators

{{DIFFERENTIATORS}}

## Target Customers

{{TARGET_CUSTOMERS}}

---

# Agent Operating Principles

## Story Sizing Rules

Every task must be completable in a **single agent session** (one context window). This prevents quality degradation from context compaction.

| Guideline            | Rule                                                   |
| -------------------- | ------------------------------------------------------ |
| **Files modified**   | Maximum 3-5 files per task                             |
| **Lines of context** | If >2000 lines needed to understand, split the task    |
| **Uncertainty**      | When uncertain about scope, err on the side of smaller |
| **Split pattern**    | Schema → Backend → Frontend → Integration              |

**If a task is too big**: Create subtasks and complete them sequentially, each in a fresh agent session.

## Goal-Backward Verification

Before marking ANY task complete, agents MUST verify actual code achieves goals:

### Verification Checklist

1. **Re-read acceptance criteria** from the original task
2. **Trace through code** to confirm each criterion is met
3. **Can you demonstrate** the user flow end-to-end?
4. **What would break this** in production?
5. **If ANY criterion is not verifiably met**, do not mark complete

### Common Failures to Catch

- Code compiles but doesn't actually implement the feature
- Happy path works but error handling is missing
- Tests pass but don't cover the actual requirements
- UI renders but user flow is incomplete

## Context Budgeting

Keep agent context usage **under 50%** of capacity. Quality degrades above this threshold.

- Spawn fresh subagents for discrete tasks
- Don't accumulate context across unrelated operations
- When context grows large, summarize and hand off to fresh agent

---

# Your AI Team

We're a founder-led company that fully leverages the power of Claude Code and the AI team it unlocks. The founder and AI work as partners, collaborating on everything.

## Executive Team

The founder is CEO. Claude (the primary agent) serves as co-CEO and orchestrates the AI team.

## Product Leadership

The `staff-software-architect` and `staff-product-manager` jointly lead product decisions. Their sign-off is required before work is considered complete.

## Team Members

{{TEAM_ROSTER}}

Each agent has a specific, non-overlapping responsibility. If you detect overlapping responsibilities, flag it for resolution.

---

# Typical Workflow

## Product Development

1. **Founder states a goal** or task that needs to be done
2. **Product leadership** (architect + PM) develops requirements
3. **Requirements assigned** to appropriate team members
4. **Engineers implement** with their own QA
5. **Product leadership reviews** completed work
6. **Cycle repeats** until goal is met

## Go-To-Market

1. **Founder or co-CEO states a goal**
2. **GTM Director coordinates** the GTM team
3. **Content creation**: Content Strategist creates, Technical Writer documents
4. **Sales enablement**: Sales Advisor guides founder on pipeline
5. **GTM Director ensures alignment** with product leadership

---

# Work Queue

The work queue (`.claude/queue/`) is the central system for autonomous task processing. This is the single source of truth for all queue mechanics.

## Queue Structure

```
queue/
├── active.json      # Currently being worked on
├── backlog.json     # Future work items (pending, needs_review, blocked)
├── completed/       # Archived completed work
├── plans/           # Research plan documents (markdown)
├── reports/         # Completion reports (markdown)
└── README.md        # Points here
```

## Queue Commands

| Command | Description |
|---------|-------------|
| `/queue:add` | Add an implementation task to the backlog |
| `/queue:research` | Add a research/investigation task |
| `/queue:work` | Pick up and complete the next task |
| `/queue:status` | View current queue state |
| `/queue:approve` | Approve a research plan → creates implementation task |
| `/queue:revise` | Request changes to a research plan |

## How to Work With the System

### Simplified Workflow

Most work follows one of two paths:

**Known work** (you know what to build):
```
/queue:add → /queue:work → done → /founder:review
```

**Needs investigation** (scope unclear, multiple approaches):
```
/queue:research → /queue:work → plan → /queue:approve → /queue:work → done
```

### When to Use What

| Situation | Command | Why |
|-----------|---------|-----|
| Bug fix with clear repro | `/queue:add` | Known problem, known solution area |
| New feature, well-defined | `/queue:add` | Acceptance criteria are clear |
| "Should we use X or Y?" | `/queue:research` | Needs investigation first |
| Major refactor | `/queue:research` | Impact unclear, needs a plan |
| Performance optimization | `/queue:research` | Must profile before fixing |
| Small UI tweak | `/queue:add` | Straightforward change |

### Execution Modes

When `/queue:work` picks up a task, it automatically branches based on type:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudrepo-io/founder-os](https://github.com/cloudrepo-io/founder-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
