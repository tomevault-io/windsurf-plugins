---
trigger: always_on
description: > **For floop contributors.** If you're a user, see [docs/integrations/](docs/integrations/) for setup guides.
---

# Floop - Agent Instructions

> **For floop contributors.** If you're a user, see [docs/integrations/](docs/integrations/) for setup guides.

## Floop Integration (REQUIRED)

You have persistent memory via floop. Learned behaviors are loaded via MCP and,
where supported, auto-injected via hooks.

**When corrected, IMMEDIATELY capture it:**
```
mcp__floop__floop_learn(right="what to do instead")

# With optional context about what went wrong:
mcp__floop__floop_learn(right="what to do instead", wrong="what you did (optional)")

# With explicit tags (optional, for pack filtering):
mcp__floop__floop_learn(right="what to do instead", tags=["topic", "category"])
```

Do NOT wait for permission. Capture learnings proactively. The hooks will also auto-detect corrections, but explicit capture is more reliable.

For non-Claude agents, see `docs/integrations/agent-prompt-template.md`.

**Available MCP tools:**
- `floop_active` - See currently active behaviors for this context
- `floop_learn` - Capture a correction (USE PROACTIVELY)
- `floop_feedback` - Signal whether a behavior was helpful (`confirmed`) or contradicted (`overridden`)
- `floop_list` - List all stored behaviors
- `floop_deduplicate` - Merge duplicate behaviors

### Codex Runtime Cadence (No Lifecycle Hooks)

In Codex environments, treat these as required pseudo-hooks:

1. **Task start**: Call `floop_active` with current `file` and `task`.
2. **Context change** (file/task/mode shift): Re-call `floop_active`.
3. **Correction received**: Immediately call `floop_learn` (no permission needed).
4. **Behavior outcome**: Call `floop_feedback` with `confirmed` or `overridden`.

If MCP is unavailable, use CLI fallback immediately:

```bash
floop active --file <path> --task <task> --json
floop learn --right "what to do instead" --wrong "what happened" --file <path>
floop list --json
```

---

## Project Overview

**floop** is a CLI tool that enables AI agents to learn from corrections and maintain consistent behavior across sessions.

**Tech stack:** Go 1.25+, Cobra CLI, YAML, Beads (issue tracking)

## Essential Reading

1. `docs/GO_GUIDELINES.md` - Go coding standards (read before writing code)

## Quick Reference

### Feedback Loop (Dogfooding) ⭐

Use floop MCP tools proactively. Capture corrections as they happen - don't wait for permission.

### Development
```bash
go build ./cmd/floop        # Build CLI
go install ./cmd/floop      # Install globally
go test ./...               # Run all tests
go test -v -cover ./...     # Verbose with coverage
go fmt ./...                # Format code
```

## Development Workflow

### Starting Work
1. Run `bv --robot-triage` to find available tasks
2. Read the task with `bd show <id>`
3. Claim it: `bd update <id> --claim`
4. Check dependencies - some tasks block others

### Writing Code
1. **Read GO_GUIDELINES.md first** - follow the patterns
2. **Write tests** - all packages need `*_test.go` files
3. **Test coverage** - test both success and error paths
4. **Format code** - run `go fmt ./...` before committing

### Making Commits
- Make small, incremental commits
- Use conventional commit format:
  - `feat:` new features
  - `fix:` bug fixes
  - `docs:` documentation
  - `test:` test additions
  - `chore:` maintenance

### Completing Work
1. **Run quality gates** (if code changed):
   - `go test ./...` — Run tests
   - `go fmt ./...` — Format code
   - If `cmd/floop/` changed: verify `docs/CLI_REFERENCE.md` is current
2. Close the issue: `bd close <id> --reason "..."`
3. Commit Dolt changes: `bd dolt commit`
4. Commit changes on a feature branch
5. Push and create a PR — **never commit directly to main**
6. Wait for review before merging

## Project Structure

- **`cmd/floop/`** — CLI entry point
- **`internal/`** — All application packages. Run `ls internal/` for current list.
- **`docs/`** — Documentation (`GO_GUIDELINES.md`, `FLOOP_USAGE.md`, `integrations/`)
- **`.floop/`** — Learned behaviors (JSONL + manifest tracked; DB + audit.jsonl gitignored)
- **`.beads/`** — Issue tracking (Dolt backend, server at `~/.dolt-data/beads`)

## Code Patterns

### CLI Commands
```go
func newXxxCmd() *cobra.Command {
    cmd := &cobra.Command{
        Use:   "xxx",
        Short: "One line description",
        RunE: func(cmd *cobra.Command, args []string) error {
            jsonOut, _ := cmd.Flags().GetBool("json")
            // Implementation
            if jsonOut {
                json.NewEncoder(os.Stdout).Encode(result)
            }
            return nil
        },
    }
    return cmd
}
```

### Error Handling
```go
result, err := doSomething()
if err != nil {
    return fmt.Errorf("context for what failed: %w", err)
}
```

### Testing
```go
func TestFunction(t *testing.T) {
    tests := []struct {
        name    string
        input   Type
        want    Type
        wantErr bool
    }{
        {"valid case", input, expected, false},
        {"error case", badInput, nil, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // test implementation
        })
    }
}
```

## When You Make Mistakes

Use `floop_learn` to capture corrections immediately. This builds the dataset for the learning loop.

## Current Phase


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nvandessel/floop](https://github.com/nvandessel/floop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
