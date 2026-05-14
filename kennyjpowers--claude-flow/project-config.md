---
trigger: always_on
description: **claudeflow** is a workflow orchestration npm package that provides a complete end-to-end feature development lifecycle for AI-assisted development. It provides custom workflow commands that work with Claude Code, OpenCode, and other AI development tools.
---

# claudeflow - AI-Assisted Development Workflow

## Project Overview

**claudeflow** is a workflow orchestration npm package that provides a complete end-to-end feature development lifecycle for AI-assisted development. It provides custom workflow commands that work with Claude Code, OpenCode, and other AI development tools.

**Version:** 2.0.0 (January 2026)
**Package:** @33strategies/claudeflow
**Distribution:** npm, yarn, pnpm

## Architecture

Standalone workflow package providing custom commands for AI-assisted feature development.

**System Requirements:**
- Node.js 20+
- npm/yarn/pnpm (any package manager)
- AI coding assistant (Claude Code, OpenCode, etc.) - optional but recommended

See [docs/INSTALLATION_GUIDE.md](docs/INSTALLATION_GUIDE.md) for detailed prerequisites and installation instructions.

## Core Workflow

Complete feature lifecycle in 6 phases:

```
BRAINSTORM → CLARIFY → SPECIFICATION → DECOMPOSITION → IMPLEMENTATION → FEEDBACK → COMPLETION
```

### Phase 1: Brainstorm
- **Command:** `/brainstorm:start <task-brief>`
- **Output:** `doc/specs/<slug>/01-brainstorm.md`
- **Purpose:** Enforce complete investigation before code changes
- **Includes:** Intent, pre-reading, codebase mapping, root cause analysis, research, clarifications

### Phase 2: Clarify
- **Command:** `/brainstorm:clarify <path-to-brainstorm>`
- **Output:** Updates `doc/specs/<slug>/01-brainstorm.md` with resolved questions
- **Purpose:** Resolve open questions interactively before creating specification
- **Process:** Detect questions → present with context → record answers → re-validate

### Phase 3: Specification
- **Command:** `/brainstorm:spec <path-to-brainstorm>`
- **Output:** `doc/specs/<slug>/02-specification.md`
- **Purpose:** Transform brainstorm into validated technical specification
- **Process:** Extract decisions → build spec → validate

### Phase 4: Decomposition
- **Command:** `/spec:decompose <path-to-spec>`
- **Output:** `doc/specs/<slug>/03-tasks.md`
- **Purpose:** Break specification into actionable tasks
- **Pattern:** Full implementation details copied into tasks (NOT summaries)

### Phase 5: Implementation
- **Command:** `/spec:execute <path-to-spec>`
- **Output:** `doc/specs/<slug>/04-implementation.md`
- **Purpose:** Implement tasks incrementally with session continuity
- **Process:** For each task: implement → test → code review → fix → commit
- **Tracks:** Progress, files modified, tests added, known issues, next steps

### Phase 6: Feedback
- **Commands:** `/feedback:add` then `/feedback:resolve`
- **Output:** `doc/specs/<slug>/05-feedback.md`
- **Purpose:** Capture and process post-implementation feedback with structured decisions
- **Two-Step Process:**
  1. **Capture:** `/feedback:add` - Loop to capture multiple feedback items (save-as-you-go)
  2. **Resolve:** `/feedback:resolve` - Batch analyze and resolve all pending items
- **Decision Outcomes:**
  - **Implement Now:** Update spec changelog → incremental `/spec:decompose` → resume `/spec:execute`
  - **Defer:** Log for future consideration in feedback file
  - **Out of Scope:** Log decision with rationale → no further action
- **Integration:** Works with incremental `/spec:decompose` and resume `/spec:execute`

### Phase 7: Completion
- **Commands:** `/spec:doc-update`, git commit & push
- **Purpose:** Finalize changes, update documentation, push to remote

## Key Commands

### Custom Commands (6)
| Command | Purpose |
|---------|---------|
| `/brainstorm:start <task-brief>` | Structured investigation workflow |
| `/brainstorm:clarify <path>` | Resolve open questions interactively |
| `/brainstorm:spec <path>` | Transform brainstorm → validated spec |
| `/feedback:add [path]` | Quick capture of feedback items |
| `/feedback:resolve [path]` | Batch analyze and resolve pending feedback |
| `/spec:doc-update <path>` | Review docs with parallel agents |

### Command Overrides (4)
Enhanced versions of spec commands:

| Command | Enhancement |
|---------|------------|
| `/spec:create <desc>` | Feature-directory aware with output path detection |
| `/spec:decompose <path>` | Incremental mode: preserves completed work, creates only new tasks |
| `/spec:execute <path>` | Resume mode: continues from previous session, skips completed work |
| `/spec:migrate` | Convert old flat structure to feature directories

## Document Organization

**Feature-Based Directories** - All docs for a feature in one place:

```
doc/specs/<feature-slug>/
├── 01-brainstorm.md        # Investigation & research
├── 02-specification.md     # Technical specification
├── 03-tasks.md             # Task breakdown
├── 04-implementation.md    # Progress tracking
└── 05-feedback.md          # Post-implementation feedback log
```

**Benefits:**
- Single source of truth per feature
- Clear lifecycle progression (01 → 02 → 03 → 04)
- Easy to find related documents
- Git-friendly tracking

## Workflow Features

### Interactive Question Resolution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kennyjpowers/claude-flow](https://github.com/kennyjpowers/claude-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
