---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run right-sized quality gates** (if code changed) - Targeted tests during active development; full suite only for final merge-readiness checks
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

## Test Scope Policy

- For bugfixes and new features under active development or investigation, prefer targeted `pytest` runs (single test, file, or focused subset).
- Run the full test suite only when branch changes are finalized and you want a final merge-readiness signal.
- Expand test scope incrementally if risk is broader, instead of defaulting to full-suite runs early.


<!-- bv-agent-instructions-v1 -->

---

## Beads Workflow Integration

This project uses [beads_viewer](https://github.com/Dicklesworthstone/beads_viewer) for issue tracking. Issues are stored in `.beads/` and tracked in git.

### Beads DB Mode (SQLite + beads-sync branch)

This repo uses `no-db: false` (see `.beads/config.yaml`) with a `beads-sync` branch for protected-branch compatibility.

**Data flow (mutations write to SQLite only, not JSONL):**
```
bd create/update/close  →  SQLite DB (.beads/beads.db)
bd export -o .beads/issues.jsonl  →  Main JSONL (flushes DB to file)
bd sync  →  Sync worktree JSONL (.git/beads-worktrees/beads-sync/.beads/issues.jsonl)
bd sync --merge  →  Merges beads-sync branch into main
```

**Critical:** `bd sync` does NOT auto-export from DB to main JSONL. It only copies the main JSONL to the sync worktree. Without a prior `bd export`, `bd sync` propagates stale JSONL. Always run `bd export -o .beads/issues.jsonl` before `bd sync`.

**Other notes:**
- **`bd sync --import-only`** reimports JSONL into the DB (useful after pulling changes from others).
- **If SQLite becomes corrupt**, delete `.beads/beads.db` and run `bd init` to reimport from JSONL, then repair (see below).
- **`bd init` reimport bug:** leaves `dependencies.metadata` and `dependencies.thread_id` as empty strings, which breaks all mutations with `sqlite3: SQL logic error: malformed JSON` during blocked_issues_cache rebuild. Fix with: `python3 -c "import sqlite3; c=sqlite3.connect('.beads/beads.db'); c.execute(\"UPDATE dependencies SET metadata=NULL WHERE metadata=''\"); c.execute(\"UPDATE dependencies SET thread_id=NULL WHERE thread_id=''\"); c.commit()"`.
- **`bd init` status import bug:** importing from JSONL may not faithfully restore `status`/`close_reason`. Verify with `bd doctor` and fix mismatches with direct SQL if needed.
- **`skip-worktree` flags:** beads may set `skip-worktree` or `assume-unchanged` on `issues.jsonl`, hiding modifications from git. Clear with `git update-index --no-skip-worktree --no-assume-unchanged .beads/issues.jsonl`.
- **v0.49.6 pinned:** v0.55.4 requires `libicui18n.so.74` (ICU 74), not available on Ubuntu 22.04 (has ICU 70).

### Essential Commands

```bash
# View issues (launches TUI - avoid in automated sessions)
bv

# CLI commands for agents (use these instead)
bd ready              # Show issues ready to work (no blockers)
bd list --status=open # All open issues
bd show <id>          # Full issue details with dependencies
bd create --title="..." --type=task --priority=2
bd update <id> --status=in_progress
bd close <id> --reason="Completed"
bd close <id1> <id2>  # Close multiple issues at once
bd sync               # Commit and push changes
```

### Workflow Pattern

1. **Start**: Run `bd ready` to find actionable work
2. **Claim**: Use `bd update <id> --status=in_progress`
3. **Work**: Implement the task
4. **Complete**: Use `bd close <id>`
5. **Export + Sync**: `bd export -o .beads/issues.jsonl && bd sync`

### Worktree Hydration

- In long-lived worktrees, hydrate before looking up freshly created issue IDs:
  ```bash
  bd sync --import-only
  ```
- This imports newer `.beads/issues.jsonl` state into the active worktree so `bd show <new-id>` resolves deterministically.

### Key Concepts

- **Dependencies**: Issues can block other issues. `bd ready` shows only unblocked work.
- **Priority**: P0=critical, P1=high, P2=medium, P3=low, P4=backlog (use numbers, not words)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tzeusy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
