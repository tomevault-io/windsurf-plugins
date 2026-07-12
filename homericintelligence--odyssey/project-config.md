---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ML Odyssey is a Mojo-based AI research platform for reproducing classic research papers. The project uses a
comprehensive 4-level hierarchical planning structure with automated GitHub issue creation.

**Current Status**: Active implementation — Mojo 1.0 migration complete, 6 neural network
architectures implemented (~198K lines of Mojo), 298+ tests across 433 test files.

## Ecosystem Context

Odyssey is part of the
[HomericIntelligence](https://github.com/HomericIntelligence) organization.
It is a **standalone ML training framework** — not a distributed agent or
microservice.

**This repo's role**: Mojo-based ML framework for reproducing classic research
papers. The shared library (`src/projectodyssey/`) provides tensor ops, autograd, layers,
and training infrastructure that all paper implementations build on.

**What this repo is NOT** (to prevent AI agents from making incorrect
assumptions):

- **Not part of a distributed agent mesh.** Zero integration with ai-maestro,
  NATS, or any agent registration/task queue. AchaeanFleet and Myrmidons
  handle that.
- **No promotion path to AchaeanFleet.** Implementations live entirely in this
  repo as Mojo libraries and executables.
- **No REST API.** No agent registration endpoint, no REST client.
- **"Agents" here = Claude Code automation** (`.claude/agents/`) for
  development workflow (code generation, PR creation, CI), not runtime
  distributed services.

**Key sibling repos** (see
[README.md](README.md#part-of-homericintelligence) for the full table):

| Repository | Role |
| --- | --- |
| [Odysseus](https://github.com/HomericIntelligence/Odysseus) | Ecosystem meta-repo and architecture docs |
| [AchaeanFleet](https://github.com/HomericIntelligence/AchaeanFleet) | Container images for the agent mesh (separate from this repo) |
| [ProjectMnemosyne](https://github.com/HomericIntelligence/ProjectMnemosyne) | Skills marketplace and team learnings |
| [ProjectHephaestus](https://github.com/HomericIntelligence/ProjectHephaestus) | Shared utilities used across the ecosystem |

## ⚠️ CRITICAL RULES - READ FIRST

### 🚫 NEVER Push Directly to Main

**The `main` branch is protected. ALL changes MUST go through a pull request.**

❌ **ABSOLUTELY PROHIBITED:**

```bash
git checkout main
git add <files>
git commit -m "changes"
git push origin main  # ❌ BLOCKED - Will be rejected by GitHub
```

**Why this is prohibited:**

- Bypasses code review and CI checks
- Can break production immediately
- Violates GitHub branch protection rules
- Makes it impossible to track changes properly

✅ **CORRECT WORKFLOW (Always Use PRs):**

```bash
# 1. Create feature branch
git checkout -b <issue-number>-description

# 2. Make changes and commit
git add <files>
git commit -m "type(scope): description"

# 3. Push feature branch
git push -u origin <issue-number>-description

# 4. Create pull request
gh pr create \
  --title "Brief description" \
  --body "Closes #<issue-number>" \
  --label "appropriate-label"

# 5. Enable auto-merge
gh pr merge --auto --rebase
```

**Emergency Situations:**

- Even for critical CI fixes, CREATE A PR
- Even for one-line changes, CREATE A PR
- Even if you're fixing your own mistake, CREATE A PR
- NO EXCEPTIONS - Always use the PR workflow

**See Also:**

- PR Best Practices: [PR Workflow](/.claude/shared/pr-workflow.md)

## Quick Links

### Core Guidelines

- [Mojo Syntax & Patterns](/.claude/shared/mojo-guidelines.md)
- [Mojo Anti-Patterns](/.claude/shared/mojo-anti-patterns.md) - 64+ failure patterns
- [Mojo UnsafePointer Memory Safety](docs/dev/mojo-memory-safety.md) - when/how to use UnsafePointer safely
- [Mojo TSAN/tcmalloc Incompatibility](docs/dev/mojo-tsan-tcmalloc-incompatibility.md) - why test-mojo-tsan aborts
- [PR Workflow](/.claude/shared/pr-workflow.md)
- [GitHub Issue Workflow](/.claude/shared/github-issue-workflow.md)
- [Common Constraints](/.claude/shared/common-constraints.md)
- [Documentation Rules](/.claude/shared/documentation-rules.md)
- [Error Handling](/.claude/shared/error-handling.md)
- [Git Commit Policy](/.claude/shared/git-commit-policy.md)
- [Output Style Guidelines](/.claude/shared/output-style-guidelines.md)
- [Tool Use Optimization](/.claude/shared/tool-use-optimization.md)

### Agent System & Skills

- [Agent Hierarchy](/agents/hierarchy.md) - 6-level hierarchy
- [Agent Configurations](/.claude/agents/) - 29 agents
- [Skills](https://github.com/HomericIntelligence/ProjectMnemosyne) - 61 skills total;
  57 live in the external ProjectMnemosyne repo, 4 remain local. Use `/mnemosyne:advise`
  to search skills by category or functionality.

## Working with Agents

This project uses a hierarchical agent system for all development work. **Always use agents** as the primary
method for completing tasks.

### Agent Hierarchy

See [agents/hierarchy.md](agents/hierarchy.md) for the complete agent hierarchy including:

- 6-level hierarchy (L0 Chief Architect → L5 Junior Engineers)
- Model assignments (Opus, Sonnet, Haiku)
- All 29 agents with roles and responsibilities

### Key Agent Principles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HomericIntelligence/Odyssey](https://github.com/HomericIntelligence/Odyssey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
