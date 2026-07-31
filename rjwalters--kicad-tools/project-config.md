---
trigger: always_on
description: This repository uses **Loom** for AI-powered development orchestration.
---

# Loom Orchestration - Repository Guide

This repository uses **Loom** for AI-powered development orchestration.

**Loom Version**: 0.13.0
**Installation Date**: 2026-07-22

## What is Loom?

Loom is a CLI + daemon for AI-powered development orchestration. It coordinates AI development workers using git worktrees and a forge (GitHub or Gitea) as the coordination layer. It supports manual coordination (Manual Orchestration Mode) and continuous autonomous orchestration (Daemon Mode).

**Loom Repository**: https://github.com/rjwalters/loom

> **Forge note**: The `gh` commands shown below are for GitHub. For Gitea repositories, Loom scripts handle API calls internally. The label-based workflow is identical regardless of forge.

## Usage Modes

Loom supports two complementary workflows:

### 1. Manual Orchestration Mode (MOM)

Use Claude Code terminals with specialized roles for hands-on development coordination.

**Setup**:
1. Open Claude Code in this repository
2. Use slash commands to assume roles: `/builder`, `/judge`, `/curator`, etc.
3. Each terminal acts as a specialized agent following role guidelines

**When to use MOM**:
- Learning Loom workflows
- Direct control over agent actions
- Debugging and iterating on processes
- Working with smaller teams

**Example workflow**:
```bash
# Terminal 1: Builder working on feature
/builder
# Claims loom:ready issue, implements, creates PR

# Terminal 2: Judge reviewing PRs
/judge
# Reviews PR with loom:review-requested, provides feedback

# Terminal 3: Curator maintaining issues
/curator
# Enhances unlabeled issues, marks as loom:ready
```

### 2. Daemon Mode

Run the Loom daemon for fully autonomous system orchestration.

**Setup**:
```bash
./.loom/scripts/daemon.sh start   # auto-build enabled (default)
```

```bash
/loom           # Activate daemon orchestration (daemon must be started first)
/loom --merge   # Aggressive autonomous development
```

**When to use Daemon Mode**:
- Production-scale development
- Fully autonomous agent workflows
- Hands-off orchestration

**Graceful shutdown**: `./.loom/scripts/daemon.sh stop` (or `touch .loom/stop-daemon`)

## Agent Roles

Loom provides specialized roles for different development tasks. Each role follows specific guidelines and uses GitHub labels for coordination.

### Available Roles

**Builder** (Manual, `builder.md`)
- **Purpose**: Implement features and fixes
- **Workflow**: Claims `loom:issue` → implements → tests → creates PR with `loom:review-requested`
- **When to use**: Feature development, bug fixes, refactoring

**Judge** (Autonomous 5min, `judge.md`)
- **Purpose**: Evaluate pull requests
- **Workflow**: Finds `loom:review-requested` PRs → evaluates → approves or requests changes
- **When to use**: Code quality assurance, automated evaluations

**Curator** (Autonomous 5min, `curator.md`)
- **Purpose**: Enhance and organize issues
- **Workflow**: Finds unlabeled issues → adds context → marks as `loom:issue`
- **When to use**: Issue backlog maintenance, quality improvement

**Architect** (Autonomous 15min, `architect.md`)
- **Purpose**: Create architectural proposals
- **Workflow**: Analyzes codebase → creates proposal issues with `loom:architect`
- **When to use**: System design, technical decision making

**Hermit** (Autonomous 15min, `hermit.md`)
- **Purpose**: Identify code simplification opportunities
- **Workflow**: Analyzes complexity → creates removal proposals with `loom:hermit`
- **When to use**: Code simplification, reducing technical debt

**Doctor** (Manual, `doctor.md`)
- **Purpose**: Fix bugs and address PR feedback
- **Workflow**: Claims bug reports or addresses PR comments → fixes → pushes changes
- **When to use**: Bug fixes, PR maintenance

**Guide** (Autonomous 15min, `guide.md`)
- **Purpose**: Prioritize and triage issues
- **Workflow**: Reviews issue backlog → updates priorities → organizes labels
- **When to use**: Project planning, issue organization

**Driver** (Manual, `driver.md`)
- **Purpose**: Direct command execution
- **Workflow**: Plain shell environment for custom tasks
- **When to use**: Ad-hoc tasks, debugging, manual operations

### Role Definitions

Full role definitions with detailed guidelines are available in:
- `.loom/roles/builder.md`
- `.loom/roles/judge.md`
- `.loom/roles/curator.md`
- And more...

## Label-Based Workflow

Agents coordinate work through GitHub labels. This enables autonomous operation without direct communication.

### Label Flow

**Issue Lifecycle**:
```
(created) → loom:issue → loom:building → (closed)
           ↑ Curator      ↑ Builder
```

**PR Lifecycle**:
```
(created) → loom:review-requested → loom:pr → (merged)
           ↑ Builder                ↑ Judge    ↑ Human
```

**Proposal Lifecycle**:
```
(created) → loom:architect → (approved) → loom:issue
           ↑ Architect       ↑ Human      ↑ Ready for Builder

(created) → loom:hermit → (approved) → loom:issue
           ↑ Hermit       ↑ Human      ↑ Ready for Builder
```

### Label Definitions

- **`loom:issue`**: Issue approved for work, ready for Builder to claim
- **`loom:building`**: Issue being implemented OR PR under review
- **`loom:review-requested`**: PR ready for Judge to review
- **`loom:pr`**: PR approved by Judge, ready for human to merge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rjwalters/kicad-tools](https://github.com/rjwalters/kicad-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
