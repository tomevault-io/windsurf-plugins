---
trigger: always_on
description: Temper reference: build
---



# Build: Execute Plan with Quality Gates

**Goal:** Implement the approved plan, task by task, with TDD and graduated quality gates.

## Active Skills

- **Context Engineering** — load hierarchical context at stage start (rules → arch → source → errors, under 2K lines/task)
- **Temper Core** — stack detection, pack resolution, quality gates
- **Source-Driven Development** — before writing framework-specific code: detect installed version → fetch current docs → cite sources → surface API conflicts. Skip for plain logic or known patterns

## Prerequisites

- Approved plan exists (from `/temper:plan`)
- OR: user provides inline instructions for trivial tasks

## Execution

### Context Loading

This stage may run in two modes:
- **Standalone** (`/temper:build`) — runs in current context, handles its own gate
- **Agent subprocess** (from `/temper`) — starts with CLEAN context, only loads what's listed below

**Subprocess mode override:** When running as an Agent subprocess, do NOT show AskUserQuestion gates or clear context. Return the build summary to the orchestrator. The orchestrator handles all gate decisions and context transitions.

In both modes, the build methodology is identical.

Files to load at start:
1. `.temper/specs/{feature}/tasks.md`
2. `.temper/specs/{feature}/intent.md` (if exists)
3. `$CLAUDE_PLUGIN_ROOT/.claude-plugin/reference/build.md` (this file)
4. `.temper/specs/{feature}/review-context.json` (if exists — loaded when re-entering from feedback loop)
5. `.temper/specs/{feature}/check-context.json` (if exists — loaded when re-entering from Check failure)

### Step 1: Load Plan

```
1. Check for .temper/build-state.json
   - If found: Validate it before offering resume:
     a. Parseable JSON — if malformed, warn and offer "Start over / Delete / Cancel"
     b. Valid stage — must be "plan_complete" or "build_complete" (with last_task_completed)
     c. Spec directory exists — .temper/specs/{spec}/ must exist on disk
     d. Artifacts exist — tasks.md and intent.md (if listed) must exist
     e. Timestamp — if updated > 30 days ago, warn about staleness
   - If valid: Ask user "Resume from Task {last_task_completed + 1}? [Y/n]" (skip in subprocess mode — use build-state.json directly)
   - If invalid: Show what's wrong, offer "Start over / Delete saved state / Cancel"
2. Check for active plan in .temper/specs/*/tasks.md
3. If multiple specs exist, ask user which to execute (skip in subprocess mode — use spec from build-state.json or orchestrator args)
4. Load tasks.md + quickstart.md (quickstart.md may not exist for Simple features — skip if absent)
5. Read plan.md for architecture decisions and blast radius (skip if no plan.md — Simple/Medium features)
6. Read all files listed in plan's "Prerequisites" or "Must Read" sections (skip if no plan.md)
7. Read active pack rules from .claude/packs/ (enabled packs only, skip if directory doesn't exist)
8. Read stack file from .claude/packs/stacks/{detected-stack}.md (skip if file doesn't exist)
9. Load .temper/specs/{feature}/intent.md if it exists
   - Parse scenario names and Given/When/Then blocks
   - If no intent.md: proceed with current behavior (unchanged)
```

**Build State Schema:**

```json
{
  "stage": "build_complete",
  "spec": "{feature-name}",
  "spec_path": ".temper/specs/{feature-name}",
  "original_args": "{user's original feature description}",
  "next_stage": "review",
  "artifacts": ["intent.md", "tasks.md"],
  "started": "2026-03-10T10:00:00Z",
  "last_task_completed": 3,
  "tasks": [
    { "id": 1, "status": "completed", "timestamp": "..." },
    { "id": 2, "status": "completed", "timestamp": "..." },
    { "id": 3, "status": "in_progress", "timestamp": "..." }
  ],
  "deviations": {
    "unplanned_files": [],
    "skipped_tasks": [],
    "approach_changes": []
  },
  "updated": "{ISO timestamp}"
}
```

**If no plan exists (trivial task):**

```
1. User gave direct instructions → treat as single-task build
2. Detect stack (same as /temper:check Step 1)
3. Read active pack rules
4. Read related existing code before implementing
5. Skip Step 2 (branch) — user decides if feature branch needed
```

### Step 2: Verify Branch

> Note: When running as an Agent subprocess from `/temper`, the orchestrator may have already created the branch at the plan gate. `git branch --show-current` will confirm.

```
1. Run: git branch --show-current
2. If on main/master:
   - Check if git pack is enabled in .claude/temper.config
   - If git pack enabled: auto-create feature/{spec-slug} branch
   - If no git pack: ask user to create feature branch
   - Suggest name: feature/{ticket}-{description}
3. If already on feature branch: proceed
```

### Step 3: Execute Tasks (in order)

For each task in tasks.md:

**a. Read context** - Read existing files, understand patterns, check adjacent code
**b. Write test first (priority order — first match wins)**

   1. **intent.md exists** → scenario-driven testing (regardless of TDD pack)
      - Each test maps to a Gherkin scenario by name
      - Given block → test setup
      - When block → action under test
      - Then block → assertions
      - One test per scenario minimum (some scenarios may need multiple tests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
