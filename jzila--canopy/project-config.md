---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking and **canopy** for parallel agent orchestration.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking and **canopy** for parallel agent orchestration.

Run `bd onboard` to learn beads, and `canopy help` to learn the orchestrator.

## IMPORTANT: Filing vs Implementing

**When the user asks to "file beads", "file issues", "create tickets", or similar phrasing, STOP after filing.** You may investigate to understand the problem, but once the bead is created with `bd create`, you are DONE. Do NOT proceed to implement or fix.

- "file a bead for X" → investigate if needed, `bd create`, then **stop**
- "create tickets for these bugs" → `bd create` for each, then **stop**
- "track this as an issue" → `bd create`, then **stop**

If the user wants implementation, they will say "fix", "implement", "resolve", or explicitly ask you to work on it.

## Git Setup
Before any git operations, verify identity is configured:
```bash
git config user.name && git config user.email  # Must both exist
```
If missing, check `~/.gitconfig` or set locally for this repo.

Use plain `git` commands—the working directory is already set, so `-C` is unnecessary.

## Commit Standards
- Format: `type: concise description` (feat, fix, refactor, test, docs, chore)
- Reference bead ID when relevant: `fix(canopy-abc): description`
- Run `go test ./...` before committing code changes
- Run `go build ./...` to verify compilation (enforced by pre-commit hook)

## Beads API Invariant

Canopy treats beads as a **minimal issue tracker with dependencies**. It MUST NOT depend on:
- Gates (human, timer, gh:*, bead)
- Formulas or workflow automation
- Watchers or notifications
- GitHub integration
- Cross-rig (gastown) references
- Any field not in the core schema

### Core Schema (canopy may use)

```go
type Task struct {
    ID          string   `json:"id"`
    Title       string   `json:"title"`
    Description string   `json:"description,omitempty"`
    Type        string   `json:"type,omitempty"`        // bug, feature, task, chore
    Priority    int      `json:"priority,omitempty"`    // 0-4
    Status      string   `json:"status,omitempty"`      // open, in_progress, closed, deferred
    Labels      []string `json:"labels,omitempty"`
    Assignee    string   `json:"assignee,omitempty"`
    Blockers    []string `json:"blockers,omitempty"`    // Dependency IDs
    UpdatedAt   string   `json:"updated_at,omitempty"`
}
```

### Task Selection (Strict)

Canopy filters tasks using ONLY these fields:
- `priority` - numeric range (0-4)
- `type` - exact match (bug, feature, task, chore)
- `labels` - set membership
- `assignee` - exact match or empty

**No fuzzy/natural language filtering.** The `--prompt` flag is deprecated.

### Commands (canopy may use)

```bash
bd ready                    # Actionable work (open, unblocked)
bd ready --priority 2       # Filter by max priority
bd ready --type bug         # Filter by type
bd list --status=open       # Query by status
bd show <id>                # Task details
bd update <id> --status=X   # Change status
bd close <id>               # Complete
bd close <id> --reason="X"  # Complete with reason
bd dep add <a> <b>          # a depends on b
bd sync                     # Git sync
```

Reference: https://github.com/jzila/beads-protocol

## Git Hooks

The pre-commit hook (`scripts/hooks/pre-commit`) enforces:
1. **Go build check**: Runs `go build ./...` when `.go` files are staged
2. **Beads sync**: Flushes pending beads changes to JSONL

Install hooks after cloning:
```bash
cp scripts/hooks/pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit
```

Or use devenv which automatically installs hooks on shell entry.

## Error Handling Guidelines

See `pkg/errors/errors.go` for defined error types and full guidelines.

### Core Rules

1. **Always wrap errors with context** using `fmt.Errorf("context: %w", err)`:
   ```go
   // Good
   if err := doSomething(); err != nil {
       return fmt.Errorf("failed to do something: %w", err)
   }

   // Bad - loses context
   if err := doSomething(); err != nil {
       return err
   }
   ```

2. **Never silently swallow errors**. If you can't return an error, log it:
   ```go
   // Good - explicit about ignoring
   _ = optionalCleanup() // Non-fatal: cleanup is best-effort

   // Good - log non-fatal errors
   if err := cleanup(); err != nil && verbose {
       fmt.Fprintf(os.Stderr, "warning: cleanup failed: %v\n", err)
   }

   // Bad - silent failure
   cleanup()
   ```

3. **Use sentinel errors** for programmatic handling:
   ```go
   import "github.com/jzila/canopy/pkg/errors"

   if errors.Is(err, errors.ErrMergeConflict) {
       // Handle conflict specifically
   }
   ```

4. **Use typed errors** for detailed information:
   ```go
   var mergeErr *errors.MergeError
   if errors.As(err, &mergeErr) {
       fmt.Printf("Merge failed for task %s\n", mergeErr.TaskID)
   }
   ```

### When to Return vs Log

- **Return errors** when the caller can handle them or needs to know
- **Log errors** only for truly non-fatal side effects (cleanup, telemetry, etc.)
- Use `warning:` prefix for non-fatal errors in verbose output

## Concurrency Guidelines

Use consistent concurrency patterns based on the access pattern of your data:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jzila/canopy](https://github.com/jzila/canopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
