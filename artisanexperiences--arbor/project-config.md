---
trigger: always_on
description: This file provides important context for developing the Arbor project.
---

# AGENTS.md - Development Guide for Arbor

This file provides important context for developing the Arbor project.

## Documentation

The project documentation is split between user-facing and development documentation:

**User Documentation** (kept up-to-date via release process):
- [README.md](./README.md) - Complete command reference, configuration guide, and scaffold step documentation

**Development Documentation:**
- This file (AGENTS.md) - Development workflow, architecture, and contribution guidelines
- `.ai/plans/arbor.md` - Historical development plan (phases 1-5, complete). Note: This file is gitignored and contains the original implementation plan.

## Development Location

All development occurs **inside a worktree**. This allows:
- Feature development on dedicated branches
- Clean separation from the bare repository
- Easy worktree creation/removal for testing

```bash
# Start development in a worktree
arbor work feature/my-feature
cd feature-my-feature
# Make changes, test, commit
arbor remove feature-my-feature  # When done
```

## Quick Reference

### File Locations

| Purpose | Location |
|---------|----------|
| CLI commands | `internal/cli/` |
| Config management | `internal/config/` |
| Git operations | `internal/git/` |
| **Workspace abstraction** | `internal/workspace/` |
| Scaffold system | `internal/scaffold/` |
| Presets | `internal/presets/` |
| Utilities | `internal/utils/` |
| Entry point | `cmd/arbor/main.go` |
| Tests | Alongside implementation files (`*_test.go`) |
| Deployment plans | `.ai/plans/` |

### Workspace Architecture

Arbor supports two workspace modes, abstracted by `internal/workspace/`:

| | Worktree Mode | CoW Mode |
|-|---------------|----------|
| Project marker | `.bare/` | `.arbor/` |
| Main workspace | git worktree | normal clone |
| Feature workspaces | git worktrees | CoW clones (`cp -c` / `cp --reflink`) |
| Config field | `workspace_mode: worktree` (or absent) | `workspace_mode: cow` |
| Disk sharing | git object store | filesystem reflinks |

The `internal/workspace.Manager` type provides a mode-agnostic API used by all CLI commands. `FindProjectRoot()` searches for `.bare/` (worktree mode) or `.arbor/` (CoW mode) in parent directories.

### Config Files

| Config | Location | Purpose | Versioned? |
|--------|----------|---------|------------|
| Project | `<project-root>/arbor.yaml` | Project-specific settings, scaffold config, pre-flight checks | No (not in a repo) |
| Repository | `<worktree>/arbor.yaml` | Team defaults (copied during init) | Yes (committed to git) |
| Local State | `<worktree>/.arbor.local` | Runtime state (db_suffix) | No (gitignored) |
| Global | `~/.config/arbor/arbor.yaml` | User defaults | No (local machine) |

### Step Naming

Steps use simplified dot notation where the tool namespace maps to the binary:

**Binary Steps:** Execute the corresponding tool with configured arguments
- `php` - Run PHP with args
- `php.composer` - Run composer (e.g., `install`, `update`)
- `php.laravel` - Run artisan commands
- `node.npm` - Run npm (e.g., `ci`, `run build`)
- `node.yarn` - Run yarn
- `node.pnpm` - Run pnpm
- `node.bun` - Run bun
- `herd` - Run herd (e.g., `link --secure`, `unlink`)

**Special Steps:** Perform scaffold operations
- `file.copy` - Copy files
- `env.read` - Read .env values
- `env.write` - Write .env values
- `env.copy` - Copy between env files
- `db.create` - Create database
- `db.destroy` - Drop database
- `bash.run` - Run bash commands
- `command.run` - Run arbitrary commands

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Worktree not found |
| 4 | Git operation failed |
| 5 | Configuration error |
| 6 | Scaffold step failed |

## Testing

### Running Tests

```bash
# All tests
go test ./... -v

# With race detector
go test ./... -race

# With coverage
go test ./... -cover

# Specific package
go test ./internal/utils/... -v
```

### Linting

Install golangci-lint (pinned to v2.1.2):

```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@v2.1.2
```

Ensure `~/go/bin` is in your PATH:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```

Run linter:

```bash
golangci-lint run ./...
```

### Test Requirements

- New functionality requires unit tests
- CLI commands require integration tests
- All tests must pass before commit
- Linting must pass before commit (`golangci-lint run ./...`)

### Test-Driven Development (TDD)

Before implementing new functionality:

1. **Write failing tests first** - Create test cases that describe the expected behavior
2. **Run tests to verify they fail** - Confirm the tests fail with current implementation
3. **Implement the feature** - Write code until tests pass
4. **Refactor if needed** - Improve implementation while keeping tests green
5. **Run full test suite** - Ensure no regressions in existing functionality

Example workflow for a new scaffold step:
```bash
# 1. Create test file for the step
touch internal/scaffold/steps/composer_install_test.go

# 2. Write failing tests that describe expected behavior
# 3. Run tests to confirm they fail
go test ./internal/scaffold/steps/... -v

# 4. Implement the step
# 5. Run tests again to verify they pass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artisanexperiences/arbor](https://github.com/artisanexperiences/arbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
