---
trigger: always_on
description: This is a meta-project for creating agents, skills, tools, and templates that standardize documentation and planning workflows for AI-assisted software development using OpenCode.
---

# OpenCode Processing Skills - Agent Instructions

## Project Overview

This is a meta-project for creating agents, skills, tools, and templates that standardize documentation and planning workflows for AI-assisted software development using OpenCode.

## Project Structure

```
.
├── AGENTS.md              # This file - agent instructions
├── README.md              # Project overview (English)
├── config.yaml.example    # Model configuration template
├── skills/                # Reusable skill definitions
├── agents/                # Agent configurations  
└── docs/                  # Project documentation
```

## Core Entities

### Workspace Entities
- **Project**: The entire workspace/repository
- **Module**: Self-contained part (service, container, frontend)
- **File/Directory**: Filesystem manifestation
- **Symbol**: Referenceable language element with meaning

### Planning Entities
- **Plan**: Implementation plan for features (with DoD, tests, requirements)
- **Phase**: Subdivision when plan exceeds single-session capacity (defines scope: what and why)
- **Implementation Plan**: Per-phase technical approach, above code level (defines: how)
- **Persistent Todo List**: Items with status + changelog
- **Session Handover**: Context transfer for session continuity (created on demand, multiple per plan possible)

### Documentation Entities
- **Project Overview**: High-level architecture, modules, references
- **Module Documentation**: Overview + inventories (directories/files + symbols) with explanations for each entry
- **Feature Documentation**: How features work, with implementation references

## Architecture Principles

- **File-based interface**: Subagents write to the defined file structure (templates). The file structure IS the interface, not return values. Every subagent that produces artifacts writes them to disk; the primary agent receives only a short status summary.
- **One subagent per output domain**: Subagents are organized by what they WRITE, not by what they do. `doc-explorer` writes to `docs/` and `plans/`. There is no separate analysis agent -- analysis is an intermediate step within the writing agent's workflow.
- **Self-delegation for scale**: When a subagent's workload would exceed comfortable context limits (e.g., documenting a project with many modules), it spawns additional instances of itself, each scoped to a smaller unit of work.
- **Agent extension over commands**: Skills extend the primary agent's behavior. Subagents handle expensive exploration.
- **Stack-agnostic**: No assumptions about language or framework
- **Two-tier documentation**: Overview with references + detail docs, deliberately curated to manage complexity
- **Redundancy-free**: Templates reference each other instead of duplicating content
- **Session-resilient**: Everything persisted, handover on demand
- **Context-aware**: Documents structured for partial loading (not everything into context at once)

## Design Decisions

Key architectural decisions and their rationale. These explain WHY the framework works the way it does.

### Why separate Phase (What/Why) from Implementation Plan (How)?

Phases define scope and acceptance criteria independent of technical approach. The implementation plan may be revised (e.g., "we chose a different library") without changing what the phase delivers. This separation also allows a plan reviewer to evaluate scope without needing to understand the technical details.

### Why does the primary agent author plans, not doc-explorer?

Plans are conversation-anchored: requirements emerge from user dialogue, trade-offs are negotiated, DoD is agreed upon. This context lives in the primary agent's conversation. Delegating plan creation to a subagent would require serializing all this context into a prompt, risking loss of intent and nuance. Documentation, by contrast, is codebase-anchored -- it can be derived from files without conversation context.

### Why one subagent (doc-explorer) instead of separate analysis and writing agents?

Earlier iterations had a separate `code-analyzer` (read-only analysis) and `doc-explorer` (writing). This created problems: (1) code-analyzer could only return text, violating the file-based interface principle; (2) the delegation chain primary -> doc-explorer -> code-analyzer added indirection without value, since doc-explorer already has the same read capabilities; (3) the primary spawning code-analyzer directly for plan creation would dump the entire analysis into the primary's context, causing token bloat. The solution: doc-explorer handles both analysis and writing. For scale, it self-delegates (spawns additional doc-explorer instances per module) rather than delegating to a different agent type.

### Why does doc-explorer self-delegate instead of the primary spawning per-module instances?

The primary agent should not need to know the internal module structure of a project. Doc-explorer discovers modules during exploration and decides how to partition the work. This keeps the primary's prompt simple ("document this project") and avoids leaking implementation details of the documentation process into the primary's context.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DasDigitaleMomentum/opencode-processing-skills](https://github.com/DasDigitaleMomentum/opencode-processing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
