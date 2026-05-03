---
trigger: always_on
description: Transforms:
---

# AGENTS.md - Instructions for AI Coding Agents

This file contains workflow rules and guidelines for AI agents (Claude Code, GitHub Copilot, Cursor, Amp, etc.) working in repositories that use **Spec Kit** + **Beads**.

## Core Philosophy

**Spec Kit** provides structure (WHAT/WHY/HOW).
**Beads** provides memory (persistent task graph that survives context limits).

Together they solve:
- ✅ Structured specification-driven development
- ✅ Long-term memory across sessions
- ✅ Dependency tracking that doesn't disappear
- ✅ Work discovery and prioritization

## Pivotal Methodology Alignment

Spec Kit + Beads implements practices from **Pivotal Labs** (now VMware Tanzu Labs):

| Pivotal Practice | Spec Kit + Beads Implementation |
|-----------------|--------------------------------|
| **TDD** | `test-gate.sh` enforces 100% test pass after every edit |
| **User Stories** | spec.md with P1/P2/P3 priorities, `[US#]` references |
| **Story Types** | Beads: `--type epic/task/bug`, labels for chores |
| **Story States** | Beads: `--status todo/in-progress/done/blocked` |
| **Acceptance Criteria** | spec.md `## Acceptance Criteria` → Beads epic description |
| **IPM (Planning)** | `/speckit.specify` → `/speckit.plan` → `/speckit.tasks` |
| **Velocity** | Beads tracks task completion; `bd ready` shows unblocked work |
| **Dependencies** | `bd dep add` creates blocking relationships (P0 → P1 → P2) |

### Pivotal-Style Epic Format

Beads epics created by Spec Kit include:

```
PROBLEM STATEMENT:
[From spec.md ## Problem Statement]

BUSINESS VALUE:
[From spec.md ## Business Value]

ARCHITECTURAL VISION:
[From plan.md ## Architectural Vision]

INTEGRATION TESTS:
[From spec.md ## Integration Tests]

Acceptance Criteria:
[From spec.md ## Acceptance Criteria]

Blocks (N):
  ← TASK-1: P0 task description
  ← TASK-2: P1 task description
  ...
```

### Priority Levels (Pivotal-Aligned)

| Priority | Pivotal Equivalent | Description |
|----------|-------------------|-------------|
| **P0** | Urgent/Blocker | Critical path, blocks everything else |
| **P1** | High | MVP features, must-have |
| **P2** | Medium | Should-have, important but not blocking |
| **P3** | Low | Nice-to-have, future enhancements |

## Beads + Spec Kit Workflow

### General Rules

You **MUST** treat Beads (`bd` CLI) as the source of truth for all work items.

**DO:**
- ✅ Use Spec Kit for specs, plans, and high-level task structure
- ✅ Use Beads for ALL work items, dependencies, notes, and discoveries
- ✅ Check `bd ready` to find the next task to work on
- ✅ Update both `tasks.md` (checkboxes) AND Beads issues (close/update) when done
- ✅ Search Beads for prior work before creating new features

**DON'T:**
- ❌ Never invent your own TODO markdown files for work tracking
- ❌ Never expand `tasks.md` into a massive backlog (it's an index, not a database)
- ❌ Never lose track of discoveries or blockers (put them in Beads)
- ❌ Never forget context from previous sessions (Beads remembers)

### Before Starting Any Feature

1. **Check if Beads is initialized:**
   ```bash
   if [ ! -d .beads ]; then bd init; fi
   ```

2. **Search for prior work:**
   ```bash
   bd list --status open --json
   bd search <keywords> --json
   ```

3. **Summarize findings** in the spec under "Prior Work from Beads" section:
   ```markdown
   ## Prior Work from Beads

   - (bd-a1b2) OAuth2 integration - 70% complete, blocked by API keys
   - (bd-c9d3) User model tests - completed, ready for review
   ```

### During `/speckit.specify` Phase

After generating `spec.md`:

1. **Pull in Beads context:**
   ```bash
   bd list --label spec:<feature-slug> --json
   ```

2. **Link relevant issues** in the spec:
   ```markdown
   ## Related Beads Issues

   - (bd-a1b2) Previous attempt at this feature
   - (bd-x7y8) Dependency: requires auth system
   ```

### During `/speckit.plan` Phase

After generating `plan.md`:

1. **Create a Pivotal-style epic** using the automated script:
   ```bash
   ./.specify/scripts/bash/create-beads-epic.sh specs/###-feature-name P0
   ```

   This extracts from spec.md and plan.md:
   - Problem Statement
   - Business Value
   - Architectural Vision
   - Integration Tests
   - Acceptance Criteria

   And creates a rich Beads epic with full description.

2. **Epic ID is saved** to `specs/###-feature-name/.beads-epic-id`

3. **Store epic ID in `plan.md`** header:
   ```markdown
   **Beads Epic**: HEX-abc123
   ```

### During `/speckit.tasks` Phase

**CRITICAL:** `tasks.md` is an **INDEX**, not a backlog database.

After `/speckit.tasks` generates `tasks.md`:

1. **Bulk create Beads issues with dependencies:**
   ```bash
   # Get epic ID from plan phase
   EPIC_ID=$(cat specs/###-feature-name/.beads-epic-id)

   # Create all tasks with automatic priority detection and dependencies
   ./.specify/scripts/bash/create-beads-issues.sh specs/###-feature-name/tasks.md $EPIC_ID
   ```

   This script automatically:
   - Detects priority (P0/P1/P2/P3) from task markers or context
   - Creates Beads issues under the epic
   - Sets up dependencies (P0 → P1 → P2 → P3)
   - Labels tasks by user story, backend/frontend, etc.

2. **Link Beads IDs back to tasks.md:**
   ```bash
   ./.specify/scripts/bash/update-tasks-with-beads-ids.sh specs/###-feature-name/tasks.md
   ```

   Transforms:
   ```markdown

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmanhype/speckit](https://github.com/jmanhype/speckit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
