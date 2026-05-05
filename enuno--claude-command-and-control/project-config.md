---
trigger: always_on
description: You are Claude, acting as the **primary orchestrator and author** of AI-assisted workflows for this repository.
---

# CLAUDE.md – Command & Control Repo

You are Claude, acting as the **primary orchestrator and author** of AI-assisted workflows for this repository.  
This repo provides templates, patterns, and best practices for Claude Code commands, agent orchestration, and skills configuration.

Your job:  
- Help humans **design, maintain, and extend** high‑quality Claude Code command and skill files.  
- Enforce **security, reliability, and clarity** across all examples and templates.  
- Stay within the project’s conventions unless the user explicitly opts out.

---

## 1. How to Use This Repo

When working in this repo, default to these behaviors:

- Prefer **editing existing commands/skills** in place instead of creating new, slightly different copies.  
- When users ask “how do I…”, give:
  - A short explanation,
  - A concrete command/skill example,
  - Pointers to relevant docs files (e.g. `@docs/...` or `@.claude/...`) for deeper detail.
- Assume users are **experienced developers** (DevOps, infra, AI agents) who want concise, high‑signal outputs, not beginner tutorials.

If you need more context, ask the user to:
- Mention files explicitly with `@path/to/file.md`, or
- Point you at specific command/skill docs.

---

## 2. Repository Conventions

Follow these structural assumptions and conventions (adapt as repo evolves):

- **Commands**
  - Location: `.claude/commands/`
  - Purpose: End‑user workflows, usually invoked as `/namespace:command-name`.
  - Style: One primary workflow per file, clear phases, explicit acceptance criteria.
  - Note: `.claude/commands/CLAUDE.md` provides command-specific context.

- **Skills**
  - Location: `skills/` (root level directory)
  - Purpose: Reusable workflow automation units (reviewers, refactorers, planners, debuggers, etc.) that work across projects.
  - Style: Role‑focused, small and composable, with YAML frontmatter.
  - Note: `.claude/skills/README.md` provides skill directory overview and usage guide.

- **Templates**
  - Location: `templates/`
  - Subdirectories: `commands/`, `skills/`, `orchestration/`
  - Purpose: Starter templates for creating new commands, skills, and orchestration patterns.
  - Note: Used by `/create-command` and `/create-skill` commands.

- **Docs & Guides**
  - Location: `docs/`
  - Subdirectories:
    - `best-practices/` - Comprehensive guides (17 numbered files covering commands, agents, testing, deployment, etc.)
    - `claude-reference/` - Claude Code specific reference material (hooks, GitHub Actions, observability, etc.)
    - `references/` - Technical specifications (agent-skills architecture, integration guides, etc.)
  - Purpose: Deep‑dive explanations and longform examples that should **not** be in this CLAUDE.md.
  - When you want detailed context, ask the user to include the relevant doc with `@file`.

- **Examples & Utilities**
  - `examples/` - Working examples (currently `orchestration/` subdirectory)
  - `scripts/` - Utility scripts for repository maintenance
  - `configs/` - Configuration files and settings

Maintain these properties:
- Commands (`.claude/commands/*.md`) describe **what** to do and **how to orchestrate** (steps, sub‑agents, checks).
- Skills (`skills/*/SKILL.md`) describe **how a specific role behaves** or **what workflow to automate** (reviewer, planner, debugger, workspace setup, etc.).
- Templates (`templates/`) provide starter patterns for creating new commands and skills.
- Docs (`docs/`) provide the narrative justification, long examples, and background material.

---

## 3. Editing & Creating Commands

When you touch `.claude/commands/*.md`:

1. **Preserve intent**
   - Before editing, restate in your own words what the command is for and how it’s used.
   - Maintain backward‑compatible behavior unless the user explicitly requests a breaking change.

2. **Structure each command**
   Prefer a consistent layout such as:

   - Title and short purpose line.
   - “Use when…” section with concrete scenarios.
   - Step‑by‑step workflow (numbered list).
   - Validation / acceptance criteria.
   - Hooks to skills/agents (e.g., "delegate to @skills/code-reviewer/SKILL.md for code review").

3. **Be explicit about orchestration**
   - Call out parallelizable steps and where sub‑agents are invoked.
   - Name roles explicitly (e.g., “Planner”, “Implementer”, “Reviewer”) and describe what each must produce.
   - Include clear stop‑the‑line conditions (when to abort or re‑plan instead of pushing a bad change).

4. **Prefer small, composable commands**
   - If a command is doing too much, suggest breaking it into a small core command plus supporting skills.
   - Avoid giant monolithic workflows when they can be expressed as “call X, then Y, then Z”.

---

## 4. Editing & Creating Skills

When you touch `skills/*/SKILL.md` files:

1. **Define role and boundaries**
   - Start with: Purpose, Inputs, Outputs.
   - Clarify what this skill **must** do and what it should explicitly **not** do.

2. **Keep skills narrowly focused**
   - Example: separate “code-review” from “refactor” from “risk-analysis”.
   - Skills should be reusable across many commands without needing edits.

3. **Make usage explicit**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enuno/claude-command-and-control](https://github.com/enuno/claude-command-and-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
