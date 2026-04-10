---
trigger: always_on
description: Development documentation for the Elixir/Phoenix Claude Code plugin.
---

# Plugin Development Guide

Development documentation for the Elixir/Phoenix Claude Code plugin.

## Overview

This plugin provides **agentic workflow orchestration** with specialist agents and reference skills for Elixir/Phoenix/LiveView development.

## Workflow Architecture

The plugin implements a **Plan → Work → Review → Compound** lifecycle:

```
/phx:plan → /phx:work → /phx:review → /phx:compound
     │           │            │              │
     ↓           ↓            ↓              ↓
plans/{slug}/  (in namespace) (in namespace) solutions/
```

> **Migration note**: The `--depth` flag replaces the old
> `--detail` flag. Use `quick|standard|deep` instead of
> `minimal|more|comprehensive`.

**Key principle**: Filesystem is the state machine. Each phase reads from previous phase's output. Solutions feed back into future cycles.

### Workflow Commands

| Command | Phase | Input | Output |
|---------|-------|-------|--------|
| `/phx:plan` | Planning | Feature description | `plans/{slug}/plan.md` |
| `/phx:plan --existing` | Enhancement | Plan file | Enhanced plan with research |
| `/phx:brief` | Understanding | Plan file | Interactive walkthrough (ephemeral) |
| `/phx:work` | Execution | Plan file | Updated checkboxes, `plans/{slug}/progress.md` |
| `/phx:review` | Quality | Changed files | `plans/{slug}/reviews/` |
| `/phx:compound` | Knowledge | Solved problem | `solutions/{category}/{fix}.md` |
| `/phx:full` | All | Feature description | Complete cycle with compounding |

### Artifact Directories

Each plan owns all its artifacts in a namespace directory:

```
.claude/
├── plans/{slug}/              # Everything for ONE plan
│   ├── plan.md                # The plan itself
│   ├── research/              # Research agent output
│   ├── reviews/               # Review agent output (individual tracks)
│   ├── summaries/             # Context-supervisor compressed output
│   ├── progress.md            # Progress log
│   └── scratchpad.md          # Auto-written decisions, dead-ends, handoffs
├── audit/                     # Audit namespace (not plan-specific)
│   ├── reports/               # 5 specialist agent outputs
│   └── summaries/             # Supervisor compressed output
├── reviews/                   # Fallback for ad-hoc reviews (no plan)
├── skill-metrics/             # Skill effectiveness dashboards and recommendations
│   ├── dashboard-{date}.json  # Per-skill aggregate metrics
│   └── recommendations-{date}.md  # Improvement recommendations
└── solutions/{category}/      # Global compound knowledge (unchanged)
    ├── ecto-issues/
    ├── liveview-issues/
    └── ...
```

### Context Supervisor Pattern

Orchestrators that spawn multiple sub-agents use a generic
`context-supervisor` (haiku) to compress worker output before
synthesis. This prevents context exhaustion in the parent:

```
Orchestrator (thin coordinator)
  └─► context-supervisor reads N worker output files
      └─► writes summaries/consolidated.md
          └─► Orchestrator reads only the summary
```

Used by: planning-orchestrator, parallel-reviewer, audit skill, docs-validation-orchestrator.

## Structure

```
claude-elixir-phoenix/
├── .claude-plugin/
│   └── marketplace.json
├── .claude/                         # Contributor tooling (NOT distributed)
│   ├── agents/
│   │   ├── phoenix-project-analyzer.md  # Analyze external codebases
│   │   └── docs-validation-orchestrator.md  # Plugin docs compatibility
│   ├── commands/
│   │   ├── psql-query.md
│   │   └── techdebt.md
│   └── skills/
│       ├── docs-check/              # /docs-check — validate against Claude Code docs
│       ├── session-scan/            # /session-scan — Tier 1 metrics
│       ├── session-deep-dive/       # /session-deep-dive — Tier 2 analysis
│       └── session-trends/          # /session-trends — trend reporting
├── scripts/
│   └── fetch-claude-docs.sh         # Download Claude Code docs for validation
├── plugins/
│   └── elixir-phoenix/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── agents/                  # 20 specialist agents
│       │   ├── workflow-orchestrator.md   # Full cycle coordination
│       │   ├── planning-orchestrator.md
│       │   ├── context-supervisor.md     # Generic output compressor (haiku)
│       │   └── ...
│       ├── hooks/
│       │   └── hooks.json           # Format, progress tracking, Stop warning
│       └── skills/                  # 38 skills
│           ├── work/                # Execution phase
│           ├── full/                # Autonomous cycle
│           ├── plan/                # Planning + deepening (--existing)
│           ├── review/              # Enhanced: Todo creation
│           ├── compound/            # Knowledge capture phase
│           ├── compound-docs/       # Solution documentation system
│           ├── investigate/
│           └── ...
├── CLAUDE.md
└── README.md
```

## Conventions

### Agents

Agents are specialist reviewers that analyze code without modifying it.

**Frontmatter:**

```yaml
---
name: my-agent
description: Description with "Use proactively when..." guidance
tools: Read, Grep, Glob, Bash
disallowedTools: Write, Edit, NotebookEdit
permissionMode: bypassPermissions
model: sonnet
effort: medium
memory: project
skills:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oliver-kriska) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
