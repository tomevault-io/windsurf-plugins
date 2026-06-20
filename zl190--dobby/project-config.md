---
trigger: always_on
description: Summon Dobby to work on substantial tasks autonomously in the background. Use when the user wants to delegate a project (research paper, build an app, design a system) to an autonomous agent that works while they do other things. Triggers on "dobby", "summon", "cast", "delegate project", "run in background", "work on this while I".
---

# Dobby

Dobby is a free elf. Give Dobby a task and Dobby works on it autonomously in the background while you do other things. Each task gets its own folder, CLAUDE.md, convergence loop, and budget cap.

## Usage

```bash
/dobby <request>                     # Give Dobby a task
/dobby --role <slug> <request>       # Give Dobby a task with a specific roster role
/dobby --converge <request>          # Run with convergence loop (build → evaluate → fix)
/dobby status                        # Show all tasks and progress
/dobby progress <task-name>          # Live progress bar for a specific task
/dobby notify                        # Show notification config status
/dobby stop <task-name>              # Stop a running task
/dobby deliver <task-name>           # Read output and present results
/dobby roster                        # Show project agent roster (from .dobby/roster-config.md)
/dobby roster init                   # Create a starter .dobby/roster-config.md
/dobby tell <task-name> "message"    # Send async message to a running task (Tier 2 inbox)
/dobby tell <task-name> STOP         # Signal agent to pause at next Tier 2 checkpoint (escalate to Tier 3)
```

## First Run: Bootstrap `.dobby/`

If `.dobby/` does not exist in the current working directory, create it:

```bash
mkdir -p .dobby/records
```

Create empty roster at `.dobby/records/roster.md`:

```markdown
| Task | Folder | Status | Budget | Spent | Started | Last Update |
|---|---|---|---|---|---|---|
```

After bootstrap, proceed to routing.

## Routing Protocol

Not everything needs a background agent. Classify first, route second:

| Classification | Route | When |
|---|---|---|
| **Quick** (< 30 sec) | Do it yourself | "What time is it?", "Read this file" |
| **Task** (< 5 min) | Task tool (subagent) | Code review, quick analysis, draft email |
| **Project** (hours/days) | Dobby works on it in tmux | Research paper, build an app, design a system |
| **Multi-domain** | Multiple tasks in parallel | "Build an app AND write docs" |

**Default to the lightest route.** For Quick and Task: do the work directly (or via Task tool).

### Roster-Aware Routing (for Project classification)

After classifying as Project, check for a roster config:

```bash
ROSTER_CONFIG=".dobby/roster-config.md"
```

If `$ROSTER_CONFIG` exists:
1. If the user passed `--role <slug>`, use that role directly
2. Otherwise, score each role's keywords against the request text; pick the top match
3. Extract `MODEL`, `BUDGET`, and `DESCRIPTION` from the matched role row
4. Tell the user: `Using role: {Role} ({Model}, {Budget})`

If `$ROSTER_CONFIG` does not exist or no role matches, fall back to defaults:
- Model: `claude-sonnet-4-6`
- Budget: ask user or default `$5`

**Budget normalization:** Always strip the leading `$` from budget values before use in shell arithmetic or `--max-budget-usd`:
```bash
BUDGET="${BUDGET#\$}"
```

### Multi-Domain Orchestration (for Multi-domain classification)

When a request spans multiple domains ("build an app AND write docs", "implement the API, test it, and write a blog post"), decompose and launch a team:

**Step 1: Decompose**
Break the request into 2-5 independent subtasks. Each subtask should be:
- Self-contained (can run without the others)
- Mappable to a roster role (or use defaults)
- Named with a short slug

Example: "build a REST API and write docs" →
- `build-api` (keywords: build, implement → Builder role)
- `write-docs` (keywords: write, document → Writer role)

**Step 2: Match roles**
For each subtask, run the same Roster-Aware Routing:
- If roster-config.md exists, keyword-match to pick role
- Extract MODEL, BUDGET, DESCRIPTION per subtask
- If no roster, all subtasks use defaults (claude-sonnet-4-6, $5)

**Step 3: Scaffold all**
For each subtask:
```bash
SUBTASK="<slug>"
TASK_DIR=".dobby/${SUBTASK}"
mkdir -p "${TASK_DIR}/records" "${TASK_DIR}/output"
```

Write each CLAUDE.md with:
1. Role injection (if matched)
2. Mission (the subtask, not the full request)
3. **Team context block:**
   ```
   ## Team Context
   You are part of a team working on: "{original request}"
   Your specific task: "{subtask description}"
   Other team members and their output directories:
   - {role}: {subtask} → .dobby/{slug}/output/
   You may read other team members' output/ directories for coordination.
   ```
4. HITL protocol, state management, constraints (same as single-agent)

**Step 4: Set up ALL listeners**
For each subtask, start completion + question listeners (same pattern as single-agent, but repeated for each). Use unique signal names: `dobby_${SUBTASK}_done`, `dobby_${SUBTASK}_question`.

**Step 5: Launch ALL agents**
For each subtask, launch in its own tmux session: `dobby-${SUBTASK}`

Use the same launch command as single-agent Step 4, but with each subtask's TASK_DIR, MODEL, and BUDGET:
```bash
tmux new-session -d -s "dobby-${SUBTASK}"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zl190/dobby](https://github.com/zl190/dobby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
