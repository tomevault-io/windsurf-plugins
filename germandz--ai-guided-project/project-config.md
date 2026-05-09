---
trigger: always_on
description: > Instructions for AI coding agents working on this project.
---

# AGENTS.md

> Instructions for AI coding agents working on this project.
> 
> This file follows the [AGENTS.md](https://agents.md) open standard and works with Claude Code, Cursor, Copilot, Codex, Jules, Windsurf, and other AI coding tools.

---

## Project Overview

*One paragraph describing what this project does, who it's for, and its primary purpose.*

---

## Quick Start

```bash
# Clone and enter project
git clone <repository-url>
cd <project-name>

# Install dependencies
# [Add your command: npm install, pip install -r requirements.txt, etc.]

# Start development server
# [Add your command: npm run dev, python manage.py runserver, etc.]

# Run tests
# [Add your command: npm test, pytest, go test ./..., etc.]
```

---

## Workflow

This project uses a documentation-driven workflow with **Unified Process** phase management.

### ⚠️ MANDATORY: Phase-First Workflow

**ALL work must be done within a phase context. No exceptions.**

#### Before ANY Implementation

AI agents MUST follow this sequence:

1. **Check phase plans exist** - Look for `docs/plans/{phase}/roadmap.md` files
   - If NO phase plans exist: STOP and request human initialize Inception phase
   - Cannot proceed without phase context

2. **Identify current phase** - Find which phase has status "In Progress"
   - Projects start with Inception → Elaboration → Construction → Transition
   - Cannot skip phases

3. **Check current iteration** - Find iteration with status "In Progress" in current phase roadmap
   - All work happens within an iteration
   - Cannot work outside iteration context

4. **Verify task exists** - Confirm task is listed in iteration's "Planned Work" table
   - If task missing: Request human add it to roadmap first
   - Cannot implement unplanned tasks

5. **Only then proceed** with reading documentation and implementing

#### Required Documentation Reading Order

Once phase context is confirmed, read in this order:

1. `docs/how-to-work/phase-planning.md` - **MANDATORY** phase workflow
2. Current phase `docs/plans/{phase}/roadmap.md` - Current iteration tasks
3. Current phase `docs/plans/{phase}/outputs.md` - Required deliverables
4. `docs/product/use-cases/README.md` - How features are defined
5. Use case(s) referenced in your task - Feature requirements
6. `docs/how-to-work/agent.md` - Collaboration guidelines
7. `docs/how-to-work/stack.md` - Technology choices
8. `docs/how-to-work/architecture.md` - System design
9. `docs/how-to-work/conventions.md` - Coding standards
10. `docs/how-to-work/tdd.md` - Test-driven development practice
11. `docs/how-to-work/roadmap.md` - High-level project roadmap

**Critical Rules**:
- Use cases are the source of truth for what features should do
- Phase roadmaps define WHEN work happens
- Phase outputs define WHAT artifacts are required
- All three must align for any implementation work

### Key Principles

- **Small PRs**: Each PR solves exactly one issue
- **Atomic commits**: Each commit does one thing and passes all tests
- **Test-Driven Development**: Write tests first (RED), make them pass (GREEN), then refactor
- **Refactor first**: Preparation commits before feature commits
- **Working software**: Every commit is deployable
- **Simplicity**: YAGNI—don't over-engineer

### Task Completion

After completing a task, iteration, or sprint:

1. **Always ask permission first** - Never update roadmaps without explicit human approval
2. **Request roadmap update** - Ask the human if you should update roadmap files
3. **What to update** (if permission granted):

   **For phase roadmaps** (`docs/plans/{phase}/roadmap.md`):
   - Update "Actual Progress" section in iteration
   - Mark completed tasks in "Planned Work" table
   - Document lessons learned
   - Update iteration status

   **For project roadmap** (`docs/how-to-work/roadmap.md`):
   - Move completed tasks from "In Progress" or "Up Next" to "Completed"
   - Include task ID, description, PR number, and completion date
   - For sprints, summarize what was completed

   **For phase outputs** (`docs/plans/{phase}/outputs.md`):
   - Update artifact status (In Progress → Completed)
   - Check off acceptance criteria
   - Update progress percentages

Roadmap and planning documents are state documents that must stay current, but updates require human oversight.

---

## Commands

### Development

```bash
# Install dependencies
# [Add your command]

# Start development server
# [Add your command]

# Build for production
# [Add your command]
```

### Testing

```bash
# Run all tests
# [Add your command]

# Run tests in watch mode
# [Add your command]

# Run tests with coverage
# [Add your command]

# Run a specific test
# [Add your command]
```

### Linting & Formatting

```bash
# Lint code
# [Add your command]

# Format code
# [Add your command]

# Type check (if applicable)
# [Add your command]
```

### Database (if applicable)

```bash
# Run migrations
# [Add your command]

# Rollback migrations
# [Add your command]

# Seed database
# [Add your command]
```

---

## Code Style

### General

- Prefer clarity over cleverness
- Keep functions small and focused
- Write self-documenting code
- Comment *why*, not *what*

### Naming

*Define your conventions. Examples:*

- Variables: `camelCase` / `snake_case`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GermanDZ/ai-guided-project](https://github.com/GermanDZ/ai-guided-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
