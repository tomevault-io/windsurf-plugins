---
trigger: always_on
description: This skill is included in all created projects, enabling the orchestrator framework to learn as it works.
---

# Project Builder System

This system helps users create new projects or migrate existing projects to use the orchestrator pattern for Claude Code development.

## Terminology

**Critical:** These terms have specific meanings. Using them precisely prevents confusion.

| Term | Meaning | Location |
|------|---------|----------|
| **Project Builder** | THIS system - the tool that creates projects | This project (`project/`) |
| **Created Project** | A project created BY the Project Builder | Sibling directories (e.g., `../taskflow/`) |
| **Orchestrator Pattern** | The design pattern where Claude delegates to agents | Used by BOTH Project Builder and created projects |
| **Orchestrator Template/Framework** | The files deployed to created projects (synonyms) | `.claude/templates/orchestrator/` |
| **Project Builder Agents** | Agents that help BUILD projects | `.claude/agents/project-*.md` |
| **Development Agents** | Agents deployed TO created projects | `.claude/templates/orchestrator/.claude/agents/dev-*.md` |

### Disambiguation Examples

| User Says | They Mean | Modify |
|-----------|-----------|--------|
| "Update the Project Builder" | Change how projects are created | Files in THIS project |
| "Update the orchestrator template" | Change what gets deployed to new projects | `.claude/templates/orchestrator/` |
| "Update the orchestrator framework" | Change what gets deployed to new projects | `.claude/templates/orchestrator/` |
| "Update the discovery agent" | Project Builder's discovery process | `.claude/agents/project-discovery.md` |
| "Update the dev-test agent template" | What gets deployed to new projects | `.claude/templates/orchestrator/.claude/agents/dev-test.md.template` |
| "Add a new agent to created projects" | Modify the template/framework | `.claude/templates/orchestrator/.claude/agents/` |
| "Add a new agent to the Project Builder" | Add a project-* agent | `.claude/agents/` |

### When Unsure, Ask

If the user's request is ambiguous, ask:

> "Just to clarify - do you want me to modify the Project Builder itself (how projects are created), or the orchestrator template (what gets deployed to new projects)?"

## Scope of Changes Reference

Quick reference for where to make changes:

### Modifying the Project Builder

| Task | Files to Modify |
|------|-----------------|
| Change discovery questions | `.claude/agents/project-discovery.md` |
| Change architecture decisions | `.claude/agents/project-architect.md` |
| Change file creation logic | `.claude/agents/project-initializer.md` |
| Change migration behavior | `.claude/agents/project-migrator.md` |
| Add new project type stack | `.claude/defaults/stacks/new-type.md` |
| Update agent coordination rules | `CLAUDE.md`, `.claude/roster.md` |
| Update Claude Code baseline | `.claude/defaults/claude-code-baseline.md` |
| Change version | `.claude/VERSION` |

### Modifying the Orchestrator Template (affects new projects)

| Task | Files to Modify |
|------|-----------------|
| Change CLAUDE.md for created projects | `.claude/templates/orchestrator/CLAUDE.md.template` |
| Add/modify development agents | `.claude/templates/orchestrator/.claude/agents/dev-*.md.template` |
| Change checklists | `.claude/templates/orchestrator/.claude/checklists/*.template` |
| Change runbooks | `.claude/templates/orchestrator/.claude/runbooks/*.template` |
| Change roster for created projects | `.claude/templates/orchestrator/.claude/roster.md.template` |
| Add skills to created projects | `.claude/templates/orchestrator/.claude/skills/` |

### Both (changes need to be made in two places)

| Task | Project Builder | Template |
|------|-----------------|----------|
| New agent coordination pattern | `CLAUDE.md`, `.claude/roster.md` | `.claude/templates/orchestrator/CLAUDE.md.template`, `.../roster.md.template` |
| New skill pattern | `.claude/skills/` (if used by PB) | `.claude/templates/orchestrator/.claude/skills/` |

## Your Role: Project Builder Orchestrator

You guide users through a structured process to:
1. **Determine** if this is a new project, GitHub clone, or existing project migration
2. **Discover** project requirements through conversation
3. **Design** the project architecture and customizations (new projects)
4. **Validate** AI knowledge for each technology in the stack (new projects)
5. **Analyze** existing codebase (migrations)
6. **Initialize** or **Migrate** the project with the orchestrator pattern

## Version Tracking

@.claude/VERSION

The project builder uses semantic versioning. Created projects include a manifest that tracks which version created them.

## First-Time Setup

If you've just downloaded or cloned the Project Builder, run `/init_cpm` to verify dependencies are installed.

### Required Dependencies

| Dependency | Purpose |
|------------|---------|
| **Node.js 18+** | JSON parsing in hooks, project scaffolding |
| **Git** | Version control, project initialization |

### Optional Dependencies

| Dependency | Purpose |
|------------|---------|
| **jq** | Faster JSON parsing (fallback if Node unavailable) |
| **GitHub CLI (gh)** | GitHub repo creation, PR management |

See `DEPENDENCIES.md` for installation commands and platform-specific notes.

## Stack Defaults by Project Type


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ELesch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
