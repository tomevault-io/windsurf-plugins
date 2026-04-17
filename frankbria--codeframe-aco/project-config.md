---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Note**: This project uses [bd (beads)](https://github.com/steveyegge/beads) for issue tracking. Use `bd` commands instead of markdown TODOs. See AGENTS.md for workflow details.

## Project Overview

**Autonomous Code Orchestrator (ACO)** is an AI-powered software development system that autonomously manages the complete lifecycle of code projects from specification to deployment. It combines:

- **Beads** - Dependency-aware issue tracking system with Git integration
- **3D Vector Memory System** - Coordinate-based context management (x=DAG position, y=cycle state, z=memory layer)
- **State Machine Orchestrator** - Processes issues through a unified development cycle

The system treats software development as a graph traversal problem where issues form nodes in a directed acyclic graph (DAG), each processed through consistent development stages.

## Core Architecture

### Three Primary Layers

1. **DAG Layer (Beads)** - Manages issue dependencies and project structure
2. **State Machine Layer** - Processes each issue through development cycle:
   ```
   Pick Issue → Architect → Write Tests → Implement → Test →
   Review → Respond to Feedback → [Revise/Spawn New Issue/Merge]
   ```
3. **Orchestrator Layer** - Traverses DAG and coordinates execution

### 3D Vector Memory System

Every piece of information has coordinates (x, y, z):
- **x**: Position in DAG (which issue)
- **y**: Position in development cycle (which state)
- **z**: Memory layer:
  - 1 = Architecture (immutable once set)
  - 2 = Interfaces
  - 3 = Implementation
  - 4 = Ephemeral

This enables precise context retrieval, efficient rollback, and cross-agent knowledge sharing.

## Beads Issue Tracker Integration

This project uses Beads for issue tracking. The `.beads/` directory contains the project-specific database.

### Essential Beads Commands

```bash
# Create issues
bd create "Issue title"
bd create "Issue title" -p 0 -t feature  # priority 0 (highest), type feature

# View issues
bd list                    # List all issues
bd list --status open      # Filter by status
bd show <issue-id>         # Show issue details

# Manage dependencies
bd dep add <blocked> <blocker>  # blocker must complete before blocked
bd dep tree <issue-id>          # Visualize dependency tree
bd dep cycles                   # Detect circular dependencies

# Find ready work (no blocking dependencies)
bd ready

# Update issues
bd update <issue-id> --status in_progress
bd update <issue-id> --priority 0
bd close <issue-id>
```

### Dependency Types
- **blocks** - Task B must complete before task A
- **related** - Soft connection, doesn't block progress
- **parent-child** - Epic/subtask hierarchical relationship
- **discovered-from** - Auto-created when discovering related work

### Git Workflow
Beads automatically syncs with Git:
- Exports to JSONL after operations (5s debounce)
- Imports from JSONL when newer than DB (after git pull)
- No manual export/import needed

## Spec-kit Integration

This project uses Spec-kit for structured specification and planning workflows.

### Available Slash Commands

```bash
# Specification workflow
/speckit.specify     # Create/update feature spec from natural language
/speckit.clarify     # Identify underspecified areas, ask clarifying questions
/speckit.analyze     # Cross-artifact consistency and quality analysis

# Planning workflow
/speckit.plan        # Execute implementation planning workflow
/speckit.tasks       # Generate actionable, dependency-ordered tasks.md
/speckit.checklist   # Generate custom checklist for current feature

# Implementation workflow
/speckit.implement   # Execute implementation plan from tasks.md

# Project governance
/speckit.constitution  # Create/update project constitution
```

### Feature Directory Structure

When working on a feature (branch format: `###-feature-name`):
```
specs/###-feature-name/
├── spec.md          # Feature specification
├── plan.md          # Implementation plan
├── tasks.md         # Actionable tasks with dependencies
├── research.md      # Research notes
├── data-model.md    # Data model documentation
├── quickstart.md    # Getting started guide
└── contracts/       # Interface contracts
```

### Common Spec-kit Scripts

Located in `.specify/scripts/bash/`:
- `create-new-feature.sh` - Initialize new feature branch and spec
- `setup-plan.sh` - Set up planning artifacts
- `update-agent-context.sh` - Update agent context files
- `check-prerequisites.sh` - Verify environment setup

## Development Workflow

### For New Features

1. **Initialize**: Use Beads to create feature issue or `/speckit.specify` for detailed spec
2. **Plan**: Use `/speckit.plan` to generate implementation plan
3. **Break Down**: Use `/speckit.tasks` to generate dependency-ordered tasks
4. **Track**: Create Beads issues for each major task with proper dependencies
5. **Implement**: Use `/speckit.implement` or work through `bd ready` issues
6. **Review**: Use quality gates before marking issues complete

### Quality Gates

Each stage requires:
- **Architecture**: Success criteria, constraints, timeline defined

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frankbria) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
