---
trigger: always_on
description: [Canonical file: `CLAUDE.md`. Keep `AGENTS.md` as a symlink to `CLAUDE.md`.]
---

# CLAUDE.md (symlinked as AGENTS.md)

[Canonical file: `CLAUDE.md`. Keep `AGENTS.md` as a symlink to `CLAUDE.md`.]

## Project

**Name:** Architect
**Description:** A terminal multiplexer and AI-assisted coding environment built on SDL3 and Zig. Architect lets developers run multiple terminal sessions in a tiled layout, annotate diffs, and integrate Claude agents directly into their workflow.
**Stack:** Zig 0.15, SDL3, ghostty-vt (terminal emulation), Nix dev shell, `just` task runner
**Status:** Active development

## Build & Run

```bash
# Worktree bootstrap (run in a fresh worktree)
just setup   # pre-caches the ghostty-vt tarball; skip if already cached

# Environment activation
nix develop  # or: direnv allow
#
# Minimal host prerequisites:
# - Nix with flakes enabled (or direnv + nix-direnv)
# - macOS (primary platform; Linux support is partial)

# Build
zig build

# Run
zig build run

# Test
zig build test

# Type check
N/A  # Zig build covers type checking

# Lint
just lint

# Format check
zig fmt --check src/
```

## Infrastructure

- **Source code hosting:** GitHub — URL: `https://github.com/forketyfork/architect` — CLI: `gh` — Skill: `managing-github`
- **Issue tracker:** GitHub Issues — URL: `https://github.com/forketyfork/architect/issues` — CLI: `gh issue` — Skill: `managing-github`
- **CI/CD:** GitHub Actions — config: `.github/workflows/`
- **Issue/PR linkage convention:** Reference issues in commit messages and PR descriptions using `#<issue-number>`. Use `Closes #N` or `Fixes #N` in the PR body to auto-close issues on merge.

Use the `managing-github` skill for all GitHub operations: creating issues, pull requests, fetching review threads, posting comments, and searching.

## Project Documentation

Read these before making any changes:

- `README.md` — User-facing product overview; skim for expected behavior, do not duplicate here
- `docs/ARCHITECTURE.md` — How it's built (layers, modules, dependencies)
- `docs/configuration.md` — Config shape for `config.toml` and `persistence.toml`
- `docs/development.md` — Developer setup and workflow notes

## Agent Rules

1. Read the project documentation listed above before writing any code.
2. For Zig changes, use the `zig-best-practices` skill; install it via the skill installer if not available.
3. Every new feature must have corresponding tests.
4. Do not introduce new dependencies without asking first.
5. If you change the architecture (new modules, new data flows), update `docs/ARCHITECTURE.md`.
6. Use conventional commit messages.
7. After making changes, verify the build, tests, and linting all pass (`zig build`, `zig build test`, `just lint`) before considering the task done.
8. Always update documentation alongside code changes (see Documentation Hygiene below).
9. Use `managing-github` skill for all GitHub interactions (issues, PRs, comments).

## Observability

### What the agent can do independently

- Run tests and see output: `zig build test`
- Build and check for compiler errors: `zig build`
- Run linting: `just lint`
- Search repo: `rg`/`fd`

### What requires developer assistance

- SDL3 window/rendering — developer must describe visual state or provide a screenshot
- Terminal emulation behavior — developer provides screen recordings or descriptions

### Debug mode

- `zig build -Doptimize=Debug` for debug builds (default)
- Add `std.log` calls at the relevant site; logs appear on stderr during `zig build run`

## Coding Conventions

- Favor self-documenting code; keep comments minimal and meaningful.
- Default to ASCII unless the file already uses non-ASCII.
- **Error handling**: Always handle errors explicitly—propagate, recover, or log. Never use bare `catch {}` or `catch unreachable`. Even for "impossible" failures like action queue appends, log them:
  ```zig
  // WRONG: silently swallows the error
  actions.append(.SomeAction) catch {};

  // CORRECT: log the error for debugging
  actions.append(.SomeAction) catch |err| {
      log.warn("failed to queue action: {}", .{err});
  };
  ```
- Run `zig fmt src/` (or `zig fmt` on touched Zig files) before wrapping up changes.
- Avoid destructive git commands and do not revert user changes.

## Git Workflow

When creating a new feature or fix branch:
1. Always start from an up-to-date `main` branch
2. Pull the latest changes: `git checkout main && git pull origin main`
3. Create your branch from main: `git checkout -b <branch-name>`
4. Never create branches from other feature branches unless explicitly intended

This ensures PRs are based on the latest code and avoids unrelated changes in your PR.

**Working in git worktrees:**
If you are executing in a git worktree, stay within that worktree and do not attempt to access the root repository directory. All your work should remain in the worktree's local directory structure.

## SDL3 Usage Notes

### Window-close handling on macOS
- When you handle Cmd+W yourself, set `SDL_HINT_QUIT_ON_LAST_WINDOW_CLOSE` to `"0"` so SDL does not emit `SDL_EVENT_QUIT` and bypass your custom close logic.

### Adding New SDL3 Key Codes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forketyfork/architect](https://github.com/forketyfork/architect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
