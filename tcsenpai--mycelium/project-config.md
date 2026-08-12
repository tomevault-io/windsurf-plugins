---
trigger: always_on
description: This repo uses the `hindsight-memory` skill. Project memory lives in the
---

# Agent Instructions

## Memory: Hindsight bank

This repo uses the `hindsight-memory` skill. Project memory lives in the
Hindsight bank named in `.bank` (do not edit by hand).

- Recall against the project bank AND `coding-knowledge` at the start of
  relevant tasks (parallel calls).
- Retain learnings as atomic, self-contained, clear memories tagged
  `user`/`feedback`/`project`/`reference`.
- Cross-project rules go in `coding-knowledge` as memories tagged
  `coding-rule` (not as directives — directives are not recallable).
- Use `retain` (async); avoid `sync_retain` in normal flow (it blocks).
- Manual ops: `/hindsight-memory-operations` (subcommands: `bootstrap`,
  `migrate [path]`, `status`, `disable`, `enable`, `forget <query>`).
- Never touch banks not prefixed with `coding-`.
- Disable per-repo: `/hindsight-memory-operations disable` or add
  `enabled: false` to `.bank`.
- Disable globally: `HINDSIGHT_MEMORY=off` or remove
  `~/.claude/hindsight-memory.enabled`.
<!-- hindsight-memory:end -->

<!-- myc:agents-start v=7 -->
## Project Management with Mycelium

This project uses [Mycelium](https://github.com/tcsenpai/mycelium) (`myc`) for task and epic management.

### Quick Reference

```bash
# Initialize mycelium in this project (creates .mycelium/ directory)
myc init

# Create an epic (a large body of work)
myc epic create --title "Feature X" --description "Build feature X"

# Create tasks within an epic
myc task create --title "Implement Y" --description "Build the implementation for Y" --epic 1 --priority high --due 2025-12-31

# Task priorities: low, medium, high, critical
# Task status: open, in_progress, closed
# Mark a task as in progress (there is no `task start`; use update):
myc task update 1 --status in_progress

# List tasks. `myc list` (top-level) shows a TREE with dependencies and epic
# grouping — use it to see the overall state. `myc task list` is a flat list.
myc list
myc task list
myc task list --epic 1
myc task list --overdue
myc task list --blocked
myc task list --all          # include closed tasks

# Manage dependencies (task 1 blocks task 2)
myc task link blocks --task 1 2
myc deps show 2

# Close tasks (blocked tasks cannot be closed without --force)
myc task close 1

# Assign tasks
myc assignee create --name "Alice" --github "alice"
myc task assign 1 1

# Link to external resources
myc task link github-issue --task 1 "owner/repo#123"
myc task link github-pr --task 1 "owner/repo#456"
myc task link url --task 1 "https://example.com"

# Project overview
myc summary

# Export data
myc export json
myc export csv
```

### Data Model

- **Epic**: A large body of work with a title and optional description (e.g., a feature or milestone)
- **Task**: A unit of work with a title and optional description, optionally linked to an epic
- **Dependency**: Task A blocks Task B (B cannot close until A is closed)
- **Assignee**: Person assigned to a task (can have GitHub username)
- **External Ref**: Link to GitHub issues/PRs or URLs

### ID Prefixes (v5)

Each entity has its **own** integer sequence, so a bare number is ambiguous
across categories. Mycelium now **displays** IDs with a one-letter category
prefix so they can't be confused:

| Category | Prefix | Example |
|---|---|---|
| Epic | `E` | `E3` |
| Task | `T` | `T3` |
| Follow-up | `F` | `F3` |
| Assignee | `A` | `A3` |
| External ref | `R` | `R3` |

**Input is backward compatible.** Every command still accepts a bare integer
(`myc task show 3`) *and* the prefixed form (`myc task show T3`,
case-insensitive). Passing the **wrong** category prefix is a hard error with a
hint — e.g. `myc task show E3` tells you `E3` is an epic and suggests
`myc epic show E3`. This catches copy/paste mix-ups.

`--format json` output is unchanged: the `id` field stays a raw integer, so
existing scripts and the Linear sync keep working.

### Git Tracking

The `.mycelium/` directory contains the SQLite database and should be committed to git:

```bash
git add .mycelium/
git commit -m "Add mycelium project tracking"
```

### Follow-up Stop hook (Claude Code)

`myc init` installs a project-local Claude Code Stop hook into `.claude/`
(script + `settings.json` wiring) that enforces the end-of-task follow-up
check automatically. Commit `.claude/` so the whole team gets it.

```bash
myc init --no-hooks          # skip the hook install
myc hooks install            # (re)install into the project's .claude/
myc hooks install --global   # install into ~/.claude instead
myc hooks uninstall          # remove (add --global for ~/.claude)
myc hooks status             # show where it's installed
```

The hook self-dedups, so a global and a local copy can coexist without
firing the check twice.

### Updating

```bash
myc update   # cargo install --force, then resync AGENTS.md + hook to the new version
```

`myc update` updates the binary via cargo, then re-runs `prime-agents --force`
and `hooks install` so this project's AGENTS.md and hook match the new version.
If cargo isn't available it skips the binary step and just resyncs the
artifacts (update the binary by hand, then rerun).

### Follow-ups (`myc followup`, alias `myc fu`)

Lightweight scratch table for non-blocking "oh-by-the-way" items

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tcsenpai/mycelium](https://github.com/tcsenpai/mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
