---
trigger: always_on
description: <!-- SW:META template="agents" version="1.0.326" sections="rules,orchestration,principles,commands,nonclaudetools,syncworkflow,contextloading,structure,agents,skills,taskformat,usformat,workflows,troubleshooting,docs" -->
---

<!-- SW:META template="agents" version="1.0.326" sections="rules,orchestration,principles,commands,nonclaudetools,syncworkflow,contextloading,structure,agents,skills,taskformat,usformat,workflows,troubleshooting,docs" -->

<!-- SW:SECTION:rules version="1.0.326" -->
## Essential Rules

```
1. NEVER pollute project root with .md files
2. Increment IDs unique (0001-9999)
3. ONLY 4 files in increment root: metadata.json, spec.md, plan.md, tasks.md
4. ALL reports/scripts/logs → increment subfolders (NEVER at root!)
5. metadata.json MUST exist BEFORE spec.md can be created
6. tasks.md + spec.md = SOURCE OF TRUTH (update after every task!)
7. EVERY User Story MUST have **Project**: field
8. For 2-level structures: EVERY US also needs **Board**: field
```

### Increment Folder Structure

```
.specweave/increments/0001-feature/
├── metadata.json                  # REQUIRED - create FIRST
├── spec.md                        # WHAT & WHY
├── plan.md                        # HOW (optional)
├── tasks.md                       # Task checklist
├── reports/                       # ALL other .md files go here!
├── scripts/                       # Helper scripts
└── logs/                          # Execution logs
    └── 2026-01-04/
```
<!-- SW:END:rules -->

<!-- SW:SECTION:orchestration version="1.0.326" -->
## Workflow Orchestration

### 1. Plan Before Code

BEFORE implementing ANY non-trivial task (3+ steps):
1. Create increment: spec.md (WHAT/WHY) + plan.md (HOW) + tasks.md (checklist)
2. Get user approval before implementing
3. If something goes sideways → STOP and re-plan

See **Task Format** and **User Story Format** sections for templates.

### 2. Verify Before Done

Never mark a task complete without proving it works:
- Code compiles/builds successfully
- Tests pass
- Acceptance criteria actually satisfied
- Ask: "Would a staff engineer approve this?"

### 3. Dependencies First

Satisfy dependencies BEFORE dependent operations.

```
Bad:  node script.js → Error → npm run build
Good: npm run build → node script.js → Success
```
<!-- SW:END:orchestration -->

<!-- SW:SECTION:principles version="1.0.326" -->
## Core Principles (Quality)

### Simplicity First
- Write the simplest code that solves the problem
- Avoid over-engineering and premature optimization
- One function = one responsibility
- If you can delete code and tests still pass, delete it

### No Laziness
- Don't leave TODO comments for "later"
- Don't skip error handling because "it probably won't fail"
- Don't copy-paste without understanding
- Test edge cases, not just happy paths

### Minimal Impact
- Change only what's necessary for the task
- Don't refactor adjacent code unless asked
- Keep PRs focused and reviewable
- Preserve existing patterns unless improving them is the task

### Demand Elegance (Balanced)
- Code should be readable by humans first
- Names should reveal intent
- BUT: Don't over-abstract for hypothetical futures
- Pragmatic > Perfect

### DRY (Don't Repeat Yourself)
- Flag repetitions aggressively — duplicated logic, config, or patterns
- Extract shared code into reusable functions/modules
- If you see the same block twice, refactor before adding a third
- Applies to code, config, tests, and documentation alike

### Plan Review Before Code
- Review the full plan thoroughly before writing any code
- Verify plan covers all ACs and edge cases before implementation
- If the plan has gaps, fix the plan first — don't discover them mid-coding
- Re-read the plan between tasks to stay aligned
<!-- SW:END:principles -->

<!-- SW:SECTION:commands version="1.0.326" -->
## Commands Reference

| Command | Purpose |
|---------|---------|
| `sw:increment "name"` | Plan new feature (PM-led) |
| `sw:do` | Execute tasks from active increment |
| `sw:done 0001` | Close increment (validates gates) |
| `sw:progress` | Show task completion status |
| `sw:validate 0001` | Quality check before closing |
| `sw:progress-sync` | Sync tasks.md with reality |
| `sw:sync-docs update` | Sync to living docs |
| `sw-github:sync 0001` | Sync increment to GitHub issue |
| `sw-jira:sync 0001` | Sync to Jira |
| `sw-ado:sync 0001` | Sync to Azure DevOps |
<!-- SW:END:commands -->

<!-- SW:SECTION:nonclaudetools version="1.0.326" -->
## Non-Claude Tools (Cursor, Copilot, etc.)

Claude Code has automatic hooks and orchestration. Other tools must do these manually.

### Capability Comparison

| Capability | Claude Code | Non-Claude Tools |
|------------|-------------|------------------|
| **Plan Mode** | `EnterPlanMode` → `sw:increment` | Manual: Create spec.md + plan.md + tasks.md |
| **Subagents** | `Task` tool for parallel work | Split into multiple chat sessions |
| **Verification** | PostToolUse hooks auto-validate | Manual: Run tests, check ACs |
| **Hooks** | Auto-run on events | YOU must mimic (see below) |
| **Task sync** | Automatic AC updates | Manual: Edit tasks.md + spec.md |
| **Skills** | Auto-activate on keywords | Read SKILL.md, follow manually |

### Manual Hook Checklist

**After EVERY task completion:**
1. Update tasks.md: `[ ] pending` → `[x] completed`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anton-abyzov/specweave](https://github.com/anton-abyzov/specweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
