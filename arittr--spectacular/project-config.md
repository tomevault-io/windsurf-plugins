---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Spectacular** is a Claude Code plugin that enables spec-anchored development with automatic parallel task execution. It extends the [superpowers](https://github.com/obra/superpowers) plugin with commands and skills for feature specification, task decomposition, and parallel execution via git worktrees and git-spice stacked branches.

**Core Philosophy:**

- **Spec anchoring**: Every line of code traces back to spec + constitution
- **Automatic parallelization**: Independent tasks run simultaneously via git worktrees
- **Reviewable PRs**: Auto-stacked branches keep changes small and focused
- **Constitution versioning**: Architectural rules evolve explicitly with immutable history

## Architecture

### Plugin Structure

This is a Claude Code plugin with a pure skills architecture (matching superpowers pattern):

```
spectacular/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata and configuration
├── commands/                     # Thin wrappers (invoke skills)
│   ├── init.md                  # → invokes validating-environment skill
│   ├── spec.md                  # → invokes writing-specs skill
│   ├── plan.md                  # → invokes decomposing-tasks skill
│   └── execute.md               # → invokes executing-plan skill
└── skills/                       # All logic lives here
    ├── validating-environment/  # Environment validation
    ├── writing-specs/           # Complete spec workflow
    ├── decomposing-tasks/       # Complete plan workflow
    ├── executing-plan/          # Execution orchestration
    ├── executing-sequential-phase/
    ├── executing-parallel-phase/
    ├── versioning-constitutions/ # Constitution evolution workflow
    ├── using-git-spice/         # Stacked branch management patterns
    └── ... (supporting skills)
```

### Command vs Skill (Updated)

- **Commands** (`commands/*.md`): Thin wrappers that invoke skills. Each command is ~5 lines that simply calls the appropriate skill.
- **Skills** (`skills/*/SKILL.md`): Contain all orchestration logic. Self-contained workflows that define HOW to do things.

### Core Workflow

1. **`/spectacular:init`** → Validates environment (superpowers, git-spice, git repo)
2. **`/spectacular:spec`** → Generates lean specification in `specs/{runId}-{feature-slug}/spec.md`
3. **`/spectacular:plan`** → Decomposes spec into execution plan with automatic phase grouping
4. **`/spectacular:execute`** → Orchestrates parallel/sequential implementation with git worktrees

## Multi-Repo Support

Spectacular supports features spanning multiple git repositories.

### Workspace Structure

```
workspace/                      # Run Claude from here
├── specs/                      # Specs at workspace root
│   └── abc123-feature/
│       ├── spec.md
│       └── plan.md
├── backend/                    # Repo 1 (has .git/)
│   ├── .worktrees/            # Per-repo worktrees
│   ├── CLAUDE.md              # Per-repo setup commands
│   └── docs/constitutions/current/
├── frontend/                   # Repo 2
│   ├── .worktrees/
│   ├── CLAUDE.md
│   └── docs/constitutions/current/
└── shared-lib/                 # Repo 3
```

### Auto-Detection

Spectacular auto-detects multi-repo mode:
- If current directory contains multiple subdirs with `.git/` → multi-repo mode
- If current directory is a single git repo → single-repo mode (original behavior)

### Multi-Repo Plan Format

Tasks specify which repo they belong to:

```markdown
## Phase 1: Foundation
- [ ] **Task 1.1**: Add shared types | repo: shared-lib | files: src/types.ts

## Phase 2: Implementation (parallel)
- [ ] **Task 2.1**: Add API endpoint | repo: backend | files: src/api.ts
- [ ] **Task 2.2**: Add UI component | repo: frontend | files: src/App.tsx
```

### Per-Repo Requirements

Each repo should have:
- `CLAUDE.md` with setup commands (required for worktrees)
- `docs/constitutions/current/` (optional but recommended)

### Execution Behavior

- **Parallel phases**: Tasks in different repos run simultaneously
- **Sequential phases**: Tasks execute in order, switching repos as needed
- **Stacking**: Each repo has its own git-spice stack (cannot stack across repos)
- **PR submission**: Submit in phase order (foundation repos first)

### Example Workflow

```bash
# From workspace root (parent of all repos)
cd workspace

# Initialize and check all repos
/spectacular:init

# Create spec for cross-repo feature
/spectacular:spec "user preferences with shared types"

# Create plan (will show per-repo tasks)
/spectacular:plan @specs/{runId}-{feature}/spec.md

# Execute (creates per-repo worktrees and branches)
/spectacular:execute @specs/{runId}-{feature}/plan.md

# Submit PRs (in phase order)
cd shared-lib && gs stack submit && cd ..
cd backend && gs stack submit && cd ..
cd frontend && gs stack submit && cd ..
```

### Codex-Specific Commands

**IMPORTANT: Intentional Duplication for Codex Independence**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arittr/spectacular](https://github.com/arittr/spectacular) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
