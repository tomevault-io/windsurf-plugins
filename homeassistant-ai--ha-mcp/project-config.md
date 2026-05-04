---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Repository Structure

This repository uses a worktree-based development workflow.

**Documentation Setup:**
- This file is `AGENTS.md` (the canonical source)
- `CLAUDE.md` is a symlink pointing to `AGENTS.md`
- Read either file - they're the same content
- Commit changes to `AGENTS.md`, the symlink will automatically reflect them

**Directory Structure:**
```
<repo-root>/                           # Main repository (checkout master here)
├── AGENTS.md                          # This file (canonical source)
├── CLAUDE.md -> AGENTS.md             # Symlink for convenience
├── worktree/                          # Git worktrees (gitignored)
│   ├── issue-42/                      # Feature branch worktree
│   └── fix-something/                 # Fix branch worktree
├── local/                             # Scratch work (gitignored)
└── .claude/agents/                    # Custom agent workflows
```

**Why use `worktree/` subdirectory:**
- Keeps worktrees organized in one place
- Gitignored (won't pollute `git status`)
- All worktrees automatically inherit `.claude/agents/` workflows
- Easy cleanup: `git worktree prune` removes stale references

**Quick command:** Use `/wt <branch-name>` skill to create worktree automatically.

## Worktree Workflow

### Creating Worktrees

**ALWAYS create worktrees in the `worktree/` subdirectory**, not at the repository root.

```bash
# Correct - worktrees go in worktree/ subdirectory
cd <repo-root>
git worktree add worktree/issue-42 -b issue-42
git worktree add worktree/feat-new-feature -b feat/new-feature

# Wrong - don't create worktrees at repo root
git worktree add issue-42 -b issue-42          # ❌ Creates orphaned worktree
git worktree add ../issue-42 -b issue-42       # ❌ Outside repo, no .claude/agents/
```

**Cleanup:** `git worktree remove worktree/<name>` or `git worktree prune` for stale references.

### Agent Workflows

Custom agent workflows are located in `.claude/agents/`:

| Agent | File | Model | Purpose |
|-------|------|-------|---------|
| **issue-analysis** | `issue-analysis.md` | Opus | Deep issue analysis - comprehensive codebase exploration, implementation planning, architectural assessment, complexity evaluation. Complements automated Gemini triage with human-directed deep analysis. |
| **issue-to-pr-resolver** | `issue-to-pr-resolver.md` | Sonnet | End-to-end issue implementation: pre-flight checks → worktree creation → implementation with tests → pre-PR checkpoint → PR creation → iterative CI/review resolution until merge-ready. |
| **my-pr-checker** | `my-pr-checker.md` | Sonnet | Review and manage YOUR OWN PRs - check comments, CI status, resolve review threads, monitor until all checks pass. Use for your PRs, not external contributions. |

## Project Overview

**Home Assistant MCP Server** - A production MCP server enabling AI assistants to control Home Assistant smart homes. Provides 92+ tools for entity control, automations, device management, and more.

- **Repo**: `homeassistant-ai/ha-mcp`
- **Package**: `ha-mcp` on PyPI
- **Python**: 3.13 only

## External Documentation

When implementing features or debugging, consult these resources:

| Resource | URL | Use For |
|----------|-----|---------|
| **Home Assistant REST API** | https://developers.home-assistant.io/docs/api/rest | Entity states, services, config |
| **Home Assistant WebSocket API** | https://developers.home-assistant.io/docs/api/websocket | Real-time events, subscriptions |
| **HA Core Source** | `gh api /search/code -f q="... repo:home-assistant/core"` | Undocumented APIs (don't clone) |
| **HA Add-on Development** | https://developers.home-assistant.io/docs/add-ons | Add-on packaging, config.yaml |
| **FastMCP Documentation** | https://gofastmcp.com/getting-started/welcome | MCP server framework |
| **MCP Specification** | https://modelcontextprotocol.io/docs | Protocol details |

## Issue & PR Management

### Automated Code Review (Gemini Code Assist)

**Gemini Code Assist** runs automatically on all PRs, providing immediate feedback on:
- Code quality (correctness, efficiency, maintainability)
- Test coverage (enforces `src/` modifications must have tests)
- Security patterns (eval/exec, SQL injection, credentials)
- Tool naming conventions and MCP patterns
- Safety annotation accuracy
- Return value consistency

**Configuration**: `.gemini/styleguide.md` and `.gemini/config.yaml`

**Division of Labor:**
- **Gemini (automatic)**: Code quality, test coverage, generic security, MCP conventions
- **Claude `contrib-pr-review` (on-demand)**: Repo-specific security (AGENTS.md, .github/), detailed test analysis, PR size assessment, issue linkage
- **Claude `my-pr-checker` (lifecycle)**: Resolve threads, fix issues, monitor CI, create improvement PRs

### Issue Labels
| Label | Meaning |
|-------|---------|
| `ready-to-implement` | Clear path, no decisions needed |
| `needs-choice` | Multiple approaches, needs stakeholder input |
| `needs-info` | Awaiting clarification from reporter |
| `priority: high/medium/low` | Relative priority |
| `triaged` | Automated Gemini triage complete |
| `issue-analyzed` | Deep Claude analysis complete |

### Issue Analysis Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
