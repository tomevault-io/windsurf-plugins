---
trigger: always_on
description: READ ~/Users/chrisedwards~/projects/chris/agent-shared/AGENTS.md BEFORE ANYTHING (skip if missing).
---

READ ~/Users/chrisedwards~/projects/chris/agent-shared/AGENTS.md BEFORE ANYTHING (skip if missing).

# Agent Development Guidelines

**Keep this file concise.** Every line here costs context on every session. Updates to AGENTS.md or CLAUDE.md must be as brief as possible — no verbose explanations, no redundant bullet lists.

## RULE 1 – ABSOLUTE (DO NOT EVER VIOLATE THIS)

You may NOT delete any file or directory unless I explicitly give the exact command **in this session**.

- This includes files you just created (tests, tmp files, scripts, etc.).
- You do not get to decide that something is "safe" to remove.
- If you think something should be removed, stop and ask. You must receive clear written approval **before** any deletion command is even proposed.

Treat "never delete files without permission" as a hard invariant.

---

## IRREVERSIBLE GIT & FILESYSTEM ACTIONS

Absolutely forbidden unless I give the **exact command and explicit approval** in the same message:

- `git reset --hard`
- `git clean -fd`
- `rm -rf`
- Any command that can delete or overwrite code/data

Rules:

1. If you are not 100% sure what a command will delete, do not propose or run it. Ask first.
2. Prefer safe tools: `git status`, `git diff`, `git stash`, copying to backups, etc.
3. After approval, restate the command verbatim, list what it will affect, and wait for confirmation.
4. When a destructive command is run, record in your response:
   - The exact user text authorizing it
   - The command run
   - When you ran it

If that audit trail is missing, then you must act as if the operation never happened.

---

## Code Editing Discipline

- Do **not** run scripts that bulk-modify code (codemods, invented one-off scripts, giant `sed`/regex refactors).
- Large mechanical changes: break into smaller, explicit edits and review diffs.
- Subtle/complex changes: edit by hand, file-by-file, with careful reasoning.

---

## Backwards Compatibility & File Sprawl

We optimize for a clean architecture now, not backwards compatibility.

- No "compat shims" or "v2" file clones.
- When changing behavior, migrate callers and remove old code.
- New files are only for genuinely new domains that don't fit existing modules.
- The bar for adding files is very high.

---

## Development Commands

Use these make targets for all checks and tests:

```bash
make check            # Run linting and static analysis (quiet output)
make test             # Run unit tests only (fast, excludes integration tests)
make test-integration # Run integration tests only (requires bd/br binaries)
make test-all         # Run all tests (unit + integration)
make check-test       # Run checks and unit tests

# Verbose output when debugging failures
make check VERBOSE=1
make test VERBOSE=1
make test-integration VERBOSE=1
```

Integration tests are separated using Go build tags (`//go:build integration`).
They require the `bd` and/or `br` binaries to be installed.

## Quick Reference: br Commands

```bash
# Adding comments - use subcommand syntax
br comments add <issue-id> "comment text"

# Labels
br label add <issue-id> <label>
br label remove <issue-id> <label>
```

## Go Code Size Limits

Keep code modular and maintainable by respecting these limits:

| Metric | Production | Test Files |
|--------|------------|------------|
| File length | 500 lines | 800 lines |
| Function length | 60 lines | 80 lines |
| Function statements | 40 | 60 |
| Line length | 120 chars | 120 chars |
| Cyclomatic complexity | 10 | 15 |

If a file or function exceeds these limits, decompose it:
- Split files by domain concept, lifecycle, or abstraction level
- Extract helper functions for repeated logic
- Use Go naming conventions: `_view.go`, `_handlers.go`, `_types.go`

## Testing Requirements
All code must be implemented with strict TDD (red-green-refactor): write a failing test first, then make it pass, then refactor. Never write production code without a failing test first. All code must build successfully, pass linting, and all tests must pass before marking a bead as closed.

### TUI Design Principles
When building UI features, follow the design principles in [`docs/UI_PRINCIPLES.md`](docs/UI_PRINCIPLES.md). This includes:
- Visual hierarchy and consistent styling
- Toast and overlay patterns
- Context-aware footer hints
- Hotkey design guidelines

### TUI Visual Testing
When making UI changes, use `scripts/tui-test.sh` to verify the layout visually:
```bash
make build                         # Build first after code changes
./scripts/tui-test.sh start        # Launch in tmux
./scripts/tui-test.sh keys 'jjjl'  # Navigate (j=down, l=expand)
./scripts/tui-test.sh enter        # Open detail pane
./scripts/tui-test.sh view         # Capture current state
./scripts/tui-test.sh quit         # Clean up
```
Always verify UI changes look correct before marking work complete.

### Testing the TUI

The abacus repository uses `br` (beads_rust) as its backend. Run the TUI directly from this repo:

```bash
make build
./bin/abacus
```

The TUI will show `[br]` in the status bar.

**Verified br operations (via TUI):**
- Create overlay (n) - creates beads via `br create`
- Edit overlay (e) - updates beads via `br update`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChrisEdwards/abacus](https://github.com/ChrisEdwards/abacus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
