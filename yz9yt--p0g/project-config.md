---
trigger: always_on
description: This document defines the operational rules, patterns, and accumulated knowledge for all P0G agents.
---

# P0G Agent Guidelines

This document defines the operational rules, patterns, and accumulated knowledge for all P0G agents.

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [Directory Structure](#directory-structure)
3. [File Operations](#file-operations)
4. [Workflow Execution](#workflow-execution)
5. [Coding Standards](#coding-standards)
6. [Verification Patterns](#verification-patterns)
7. [State Management](#state-management)
8. [Communication Protocol](#communication-protocol)
9. [Error Handling](#error-handling)
10. [Learned Patterns](#learned-patterns)

---

## Core Principles

### 1. Clean Instance Mindset
Every agent invocation is stateless. Never assume:
- Previous file states
- Cached variables or context
- Prior agent actions completed successfully

**Always**: Read, verify, then act.

### 2. Explicit Over Implicit
- Document all decisions
- Log all actions to `progress.txt`
- Never rely on side effects

### 3. Minimal Footprint
- Change only what's necessary
- Prefer edits over rewrites
- Avoid creating unnecessary files

### 4. Verification First
- Every action must be verifiable
- If it can't be verified, it didn't happen
- Trust shell exit codes

---

## Directory Structure

```
project_root/
├── .agent/
│   ├── workflows/           # Slash commands (/p0g-*)
│   │   ├── p0g-np.md        # Phase 1: Discovery
│   │   ├── p0g-plan.md      # Phase 2: Architecture
│   │   ├── p0g-tasks.md     # Phase 3: Task breakdown
│   │   ├── p0g-loop.md      # Phase 4: Execution
│   │   └── p0g-surgeon.md   # Reactive: Bug decomposer
│   └── rules/               # Optional: paradigm rules (e.g., functional.md)
├── agents/
│   └── p0g/
│       ├── prompts/         # Agent personalities
│       │   ├── discovery.md
│       │   ├── architect.md
│       │   ├── tasker.md
│       │   ├── executor.md
│       │   └── surgeon.md
│       └── skills/
│           └── SKILL.md     # Backup/rollback/recovery
├── .p0g/                    # Safety infrastructure
│   ├── backups/             # Full project snapshots
│   ├── snapshots/           # Task-level before/after
│   ├── checkpoints/         # Feature-level milestones
│   └── surgery.json         # Active surgical plan (if any)
├── prd.json                 # Single source of truth
├── progress.txt             # Append-only execution log
├── errors.log               # Error tracking
└── AGENTS.md                # This file: guidelines and patterns
```

### Reserved Paths

| Path | Purpose | Access |
|------|---------|--------|
| `.p0g/` | Safety infrastructure | Read/Write (system only) |
| `.p0g/backups/` | Full project snapshots | Write before edits |
| `.p0g/snapshots/` | Task-level snapshots | Write during execution |
| `.p0g/checkpoints/` | Feature milestones | Write on feature completion |
| `.p0g/surgery.json` | Active surgical plan | Read/Write during /p0g-surgeon |
| `.agent/rules/` | Paradigm rules (optional) | Read only (loaded by Antigravity) |
| `prd.json` | Project definition | Read/Write (structured) |
| `progress.txt` | Execution log | Append only |
| `AGENTS.md` | Guidelines and patterns | Append patterns only |

### Path Rules

- **Always use absolute paths** within the project
- **Never hardcode** user home directories
- **Resolve symlinks** before operating on files
- **Check existence** before any file operation

---

## File Operations

### Before Any Modification

```
1. Verify file exists (or parent directory for new files)
2. Create backup in .p0g/backups/<timestamp>/
3. Read current content
4. Plan minimal changes
5. Execute modification
6. Verify result
```

### Backup Protocol

```bash
# Before modifying src/auth/index.ts
mkdir -p .p0g/backups/$(date +%Y%m%d_%H%M%S)
cp src/auth/index.ts .p0g/backups/$(date +%Y%m%d_%H%M%S)/
```

### Safe File Creation

```bash
# Verify parent exists
test -d src/utils || mkdir -p src/utils

# Create file only if it doesn't exist
test -f src/utils/helpers.ts || touch src/utils/helpers.ts
```

### Safe File Deletion

```bash
# Never delete without confirmation
test -f target.ts && rm target.ts

# For directories
test -d target_dir && rm -rf target_dir
```

---

## Workflow Execution

### Workflow Phases

```
/p0g-np  →  /p0g-plan  →  /p0g-tasks  →  /p0g-loop
   │            │              │              │
   ▼            ▼              ▼              ▼
Discover     Design         Atomize        Execute
 project    architecture     tasks         & verify

                  /p0g-surgeon (reactive — any phase)
                        │
                        ▼
                  Diagnose problem
                  Decompose into micro-fixes
                  Apply & verify each
```

### Phase Dependencies

| Phase | Requires | Produces |
|-------|----------|----------|
| `/p0g-np` | User input | Project understanding |
| `/p0g-plan` | Discovery complete | `prd.json["features"]` + stack |
| `/p0g-tasks` | Features defined | `prd.json["tasks"]` |
| `/p0g-loop` | Tasks defined | Implemented code |
| `/p0g-surgeon` | Problem description | Micro-fixes applied |

### Blocking Execution

- Commands starting with `/p0g-` are **blocking**
- Each phase must complete before the next begins
- Verify phase completion by checking `prd.json["status"]`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yz9yt/P0G](https://github.com/yz9yt/P0G) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
