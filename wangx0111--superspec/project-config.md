---
trigger: always_on
description: >-
---


# Superspec

Superspec unifies [spec-kit](https://github.com/github/spec-kit) specification-driven
development with [obra/superpowers](https://github.com/obra/superpowers) agent
capabilities into a single workflow. Spec-kit provides the document structure and
governance; superpowers provides deep clarification, task decomposition, and
engineering execution discipline.

![AI-Powered: End-to-End Development Workflow (SDD)](assets/workflow-overview-en.png)

## Prerequisites

**Required**: None. Superspec works standalone with built-in fallback protocols.

**Optional (enhanced)**: Install [obra/superpowers](https://github.com/obra/superpowers)
skills to `~/.agents/skills/` or `.agents/skills/` for richer brainstorming,
planning, and execution capabilities. See [superpowers-bridge.md](references/superpowers-bridge.md)
for detection and integration details.

## Project Structure

When initialized, superspec relies on spec-kit's two top-level directories
at the project root: `.specify/` for tool metadata and `specs/` for feature
artifacts.

```
your-project/
├── .specify/
│   ├── memory/
│   │   └── constitution.md      # Project governance principles
│   ├── superpowers.yml          # Superpowers detection status (auto-managed)
│   └── templates/               # Document templates (copied on init)
└── specs/
    └── NNN-feature-name/
        ├── spec.md              # Feature specification
        ├── plan.md              # Implementation plan
        ├── tasks.md             # Task breakdown
        ├── progress.yml         # Phase progress tracker (auto-managed)
        └── checklist-*.md       # Generated checklists
```

## Commands

| Command | Purpose |
|---------|---------|
| `/speckit.superspec.status` | Show current progress and suggest next step (resumable) |
| `/speckit.constitution` | Create or update project governance principles |
| `/speckit.specify` | Create a feature specification with user stories |
| `/speckit.superspec.brainstorm` | Deep-dive edge cases and refine a spec document |
| `/speckit.plan` | Create a technical implementation plan |
| `/speckit.superspec.tasks` | Generate a phased task breakdown |
| `/speckit.superspec.execute` | Orchestrate implementation with TDD + subagents |
| `/speckit.superspec.review` | Run code review against spec requirements |
| `/speckit.checklist` | Generate a contextual checklist |

---

## Session Resumability

Superspec is designed to be **fully resumable across sessions**. All state is persisted
in the `.specify/` directory as markdown files. When a session is interrupted (agent
timeout, user leaves, CLI crash), no progress is lost.

### Progress Tracking

Each feature spec directory contains a `progress.yml` file that records phase status:

```yaml
# specs/NNN-feature-name/progress.yml
feature: feature-name
created: 2026-04-22
current_phase: brainstorm
phases:
  constitution: { status: done, updated: 2026-04-22 }
  specify:      { status: done, updated: 2026-04-22 }
  brainstorm:   { status: in_progress, updated: 2026-04-22, sessions: 1 }
  plan:         { status: pending }
  tasks:        { status: pending }
  execute:      { status: pending }
  review:       { status: pending }
```

**Status values**: `pending`, `in_progress`, `done`, `skipped`

Every command updates `progress.yml` when it starts (`in_progress`) and finishes (`done`).

### Superpowers Status Tracking

A project-level file `.specify/superpowers.yml` records which superpowers skills
are available. This makes the superpowers integration **visible in the project docs**
and **persistent across sessions** — no need to re-detect on every command.

```yaml
# .specify/superpowers.yml
last_checked: 2026-04-22T14:30:00
skills:
  brainstorming:
    detected: true
    path: ~/.agents/skills/brainstorming/SKILL.md
  writing-plans:
    detected: true
    path: ~/.agents/skills/writing-plans/SKILL.md
  executing-plans:
    detected: false
  subagent-driven-development:
    detected: false
  test-driven-development:
    detected: true
    path: .agents/skills/test-driven-development/SKILL.md
  requesting-code-review:
    detected: false
```

**When this file is updated**:
- On `/speckit.constitution` (initial creation)
- On `/speckit.superspec.status` (re-check)
- On any command that needs a superpowers skill (lazy re-check if missing)
- User can manually edit this file to override detection results

**Why persist this**: So that project documentation reflects which superpowers are
in use. A new team member reading `.specify/` can immediately see the project's
enhanced capabilities without running any command.

### Resume Protocol

When ANY superspec command is invoked, the agent MUST first run the **resume check**:

1. Check if `.specify/` directory exists
2. If yes, scan for `progress.yml` files in each spec directory
3. Read the most recent `progress.yml` to determine `current_phase`
4. Read `.specify/superpowers.yml` to determine which superpowers are available.
   If the file does not exist, run superpowers detection and create it.
5. Report to user: "Detected existing progress for [feature]: [phase] is [status].
   Superpowers: [list detected skills]. Resuming from this point." or
   "No previous progress found, starting fresh."
6. For `in_progress` phases: re-read all existing artifacts for that phase and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WangX0111/superspec](https://github.com/WangX0111/superspec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
