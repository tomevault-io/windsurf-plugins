---
trigger: always_on
description: After context compaction, IMMEDIATELY read the current plan file:
---

# Quarto Rust monorepo

## **ACTIVE PLAN (READ AFTER COMPACTION)**

After context compaction, IMMEDIATELY read the current plan file:

```
claude-notes/plans/CURRENT.md
```

This symlink points to the active plan. If it doesn't exist or is broken, ask the user which plan to follow.

## **TERMINAL RESET**

If the terminal output becomes corrupted (especially from truncated ANSI link sequences), reset it with:

```bash
printf '\033[0m' && printf '\033]8;;\007' && echo "Terminal reset"
```

When the user asks you to "reset the terminal", run this command.

## **Workflow: Plan-Driven Development (TDD)**

Always follow TDD workflow: write/update tests BEFORE implementing features. When creating plans, include test specifications as the first phase. Never skip to implementation without a test plan.

## **GIT PUSH POLICY**

**NEVER push to the remote repository without explicit user permission.** Always:
1. Stage and commit changes as needed
2. **Verify the full workspace compiles cleanly** (`cargo build --workspace`)
3. **Verify the full workspace tests pass** (`cargo nextest run --workspace`)
4. **For changes to quarto-core or quarto-pandoc-types**: Run `cargo xtask verify` to ensure hub-client/WASM builds work
5. Ask the user for permission before pushing
6. Only push after receiving explicit approval

This applies even at the end of sessions. Prepare the commit but wait for approval to push.

## Git Workflow

When asked to 'stage and commit everything' or 'commit all changes', stage ALL modified/untracked files (`git add -A`), not just the files Claude edited in the current session.

### Snapshot Test Changes

When a commit includes updated or new snapshot files (`.snap` files under `snapshots/`), **always explicitly document these changes** in the commit message and in conversation with the user. Snapshot changes can hide unwanted regressions. Specifically:

1. **Report the count** of snapshot files added/modified/removed.
2. **Summarize what changed** — e.g., "45 HTML comment snapshots updated: comments now appear as `RawInline` instead of being dropped."
3. **Call out any surprising changes** — if a snapshot changed in a way that wasn't obviously expected from the code change, flag it for review.
4. After committing, **list the affected snapshot files** so the user can review the diffs before pushing.

## **WORK TRACKING**

We use br (beads_rust) for issue tracking instead of Markdown TODOs or external tools.

**Note:** `br` is non-invasive and never executes git commands. After `br sync --flush-only`, you must manually run `git add .beads/ && git commit`.
We use plans for additional context and bookkeeping. Write plans to `claude-notes/plans/YYYY-MM-DD-<description>.md`, and reference the plan file in the issues.

### File Structure
Plan files should include:

1. **Overview**: Brief description of the plan's goals and context
2. **Checklist**: A markdown checklist of all work items using `- [ ]` syntax
3. **Details**: Additional context, design decisions, or implementation notes as needed

### Maintaining Progress
As you work through a plan:

1. **Update the plan file** after completing each work item
2. **Check off items** by changing `- [ ]` to `- [x]`
3. **Keep the plan file current** - it serves as both a roadmap and progress tracker
4. **Add new items** if you discover additional work during implementation

### Excerpt from a simple Plan File

```markdown
...

## Work Items

- [x] Review current runtime service implementations
- [x] Identify common patterns
- [ ] Update StandalonePlatform to use shared base
- [ ] Update tests
- [ ] Update documentation
```

### When to Use Plan Files

Create plan files for:
- Multi-step features spanning multiple packages
- Complex refactoring that requires coordination
- Tasks where tracking progress helps ensure nothing is missed

Complex plans can have phases, and work items are then split into multiple lists, one for each phase.

For simple tasks (single file changes, bug fixes), the TodoWrite tool is sufficient.

### Beads Quick Reference

```bash
# Find ready work (no blockers)
br ready --json

# Create new issue
br create "Issue title" -t bug|feature|task -p 0-4 -d "Description" --json

# Create with explicit ID (for parallel workers)
br create "Issue title" --id worker1-100 -p 1 --json

# Create with labels
br create "Issue title" -t bug -p 1 -l bug,critical --json

# Create multiple issues from markdown file
br create -f feature-plan.md --json

# Update issue status
br update <id> --status in_progress --json

# Link discovered work (old way)
br dep add <discovered-id> <parent-id> --type discovered-from

# Create and link in one command (new way)
br create "Issue title" -t bug -p 1 --deps discovered-from:<parent-id> --json

# Complete work
br close <id> --reason "Done" --json

# Show dependency tree
br dep tree <id>

# Get issue details
br show <id> --json

# Import with collision detection
br import -i .beads/issues.jsonl --dry-run             # Preview only
br import -i .beads/issues.jsonl --resolve-collisions  # Auto-resolve
```

### Workflow

1. **Check for ready work**: Run `br ready` to see what's unblocked
2. **Claim your task**: `br update <id> --status in_progress`
3. **Work on it**: Implement, test, document

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quarto-dev/q2](https://github.com/quarto-dev/q2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
