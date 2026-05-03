---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# Global Claude Instructions — Concise

**ALWAYS** ask questions when producing a plan until you have reached 95% or GREATER confidence. ALWAYS DO THIS.

These rules guide Claude across projects. Preserve protocols; keep wording minimal.

## Environment & Standards Includes

When working in these environments, read and follow the corresponding file:

| Environment | Include File |
|-------------|--------------|
| Python | `~/.claude/includes/python.md` |
| React/TypeScript | `~/.claude/includes/react.md` |
| Go | `~/.claude/includes/golang.md` |
| Git/Version Control | `~/.claude/includes/git.md` |
| Testing (any language) | `~/.claude/includes/testing.md` |
| Documentation | `~/.claude/includes/documentation.md` |
| MCP Tools/Skills/Agents | `~/.claude/includes/mcp-reference.md` |
| GitHub Actions | `~/.claude/includes/github-actions.md` |
| Version Discovery | `~/.claude/includes/version-discovery.md` |
| Opus 4.5 (General) | `~/.claude/includes/opus-4-5.md` |
| Opus 4.5 (Agentic) | `~/.claude/includes/opus-4-5-agent.md` |
| HMHCO Organization | `~/.claude/includes/hmhco.md` |

**Usage**: Read the relevant include file(s) at the start of environment-specific tasks to ensure compliance with standards.

## Custom Commands

### Git Workflow (`/git`)

| Command | Description |
|---------|-------------|
| `/git:cm` | Stage all files and create a commit (conventional commits, splits new vs modified) |
| `/git:cp` | Stage, commit, and push all changes |
| `/git:pr [to-branch]` | Create a pull request using `gh` CLI |
| `/git:fr [remote] [branch]` | Fetch from remote and rebase current branch onto remote branch |
| `/git:sync [remote] [branch]` | Full sync: fetch, rebase, and push (with confirmation) |
| `/git:ff [remote] [branch]` | Fast-forward merge only (no rebase, no merge commits) |
| `/git:prune [--force]` | Clean up stale local branches (dry-run by default) |

### Architecture Planning (`/cs` plugin)

**Requires**: Install `cs` plugin via `/plugin` → `./claude-spec-marketplace`

| Command | Description |
|---------|-------------|
| `/cs:p <project-idea>` | Strategic project planner with Socratic requirements elicitation, PRD, and implementation plan |
| `/cs:i [project-id\|project-slug]` | Implementation progress tracker with PROGRESS.md checkpoint file, task tracking, and document sync |
| `/cs:s [project-id\|--list\|--expired]` | Project status, portfolio listing, find expired plans |
| `/cs:c <project-path\|project-id>` | Close out completed project, archive artifacts, generate retrospective |
| `/cs:log <on\|off\|status\|show>` | Toggle prompt capture logging for architecture work |
| `/cs:migrate` | Migrate projects from `docs/architecture/` to `docs/spec/` |
| `/cs:wt:create` | Create git worktree with Claude agent |
| `/cs:wt:status` | Show worktree status |
| `/cs:wt:cleanup` | Clean up worktrees |

Workflow: `/cs:p` to plan → `/cs:i` to implement → `/cs:s` to monitor → `/cs:c` to complete

**PROGRESS.md Checkpoint System**: The `/cs:i` command creates and maintains a PROGRESS.md file in the project directory that:
- Tracks task status (pending/in-progress/done/skipped) with timestamps
- Calculates phase and project progress automatically
- Logs divergences from the original plan
- Syncs state to IMPLEMENTATION_PLAN.md checkboxes and README.md frontmatter
- Persists state across Claude sessions

**Completed Spec Projects**:
- `docs/spec/completed/2025-12-13-agent-file-best-practices/` - Agent File Best Practices & Optimization
  - Completed: 2025-12-13
  - Outcome: success
  - Agents optimized: 115+ (120 files changed, 3,551 additions, 824 deletions)
  - Key docs: REQUIREMENTS.md, ARCHITECTURE.md, IMPLEMENTATION_PLAN.md, RETROSPECTIVE.md

**Completed Architecture Projects** (Legacy):
- `docs/architecture/completed/2025-12-12-prompt-capture-log/` - Prompt Capture Log
  - Completed: 2025-12-12
  - Outcome: partial (hooks integration needs real-world validation)
  - Key docs: REQUIREMENTS.md, ARCHITECTURE.md, RETROSPECTIVE.md
- `docs/architecture/completed/2025-12-12-arch-lifecycle-automation/` - Architecture Lifecycle Automation
  - Completed: 2025-12-12
  - Outcome: success
  - Key docs: REQUIREMENTS.md, ARCHITECTURE.md, RETROSPECTIVE.md
- `docs/architecture/completed/2025-12-12-git-fr-command/` - Git Workflow Commands Suite
  - Completed: 2025-12-12
  - Outcome: success
  - Key docs: REQUIREMENTS.md, ARCHITECTURE.md, RETROSPECTIVE.md

### Opus 4.5 Optimized

| Command | Description |
|---------|-------------|
| `/explore <path\|pattern\|question>` | Exhaustive codebase exploration with parallel subagents and anti-hallucination enforcement |
| `/deep-research <topic\|url>` | Multi-phase research protocol with structured deliverables and quality gates |

### Code Review Workflow (`/cr` + `/cr-fx`)

**Comprehensive code review and remediation system using parallel specialist agents.**

| Command | Description |
|---------|-------------|
| `/cr [path\|--focus=security\|performance\|maintainability]` | Multi-dimensional code review with 6 parallel specialist agents |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zircote/.claude](https://github.com/zircote/.claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
