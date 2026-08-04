---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Claude Code plugin that provides spec-driven development workflows. It guides feature development through three phases: Requirements (EARS notation), Design (architecture), and Tasks (trackable implementation items).

## Plugin Structure

```
.claude-plugin/plugin.json  - Plugin manifest (name, version, metadata)
commands/                   - Slash command definitions
scripts/                    - Standalone execution scripts (spec-exec, spec-loop)
skills/spec-workflow/       - Main skill with reference docs
agents/                     - Subagent definitions
templates/                  - Document scaffolding for specs
```

## Commands

| Command | Purpose |
|---------|---------|
| `/spec-brainstorm` | Brainstorm a feature idea before spec creation |
| `/spec <name>` | Start new spec with 3-phase workflow |
| `/spec-refine` | Update existing requirements/design |
| `/spec-tasks` | Regenerate tasks from spec |
| `/spec-status` | Show progress and task completion |
| `/spec-validate` | Validate completeness and consistency |
| `/spec-exec` | Run one autonomous implementation iteration |
| `/spec-loop` | Loop implementation until all tasks complete |
| `/spec-accept` | Run user acceptance testing for formal sign-off |
| `/spec-docs` | Generate documentation from spec and implementation |
| `/spec-release` | Generate release notes, changelog, and deployment checklist |
| `/spec-verify` | Run post-deployment smoke tests against a live environment |
| `/spec-retro` | Run a retrospective to capture lessons learned |
| `/research` | Deep research before planning — parallel agents search docs, web, and codebase |
| `/zoom-out` | Step up a layer of abstraction and map modules, interfaces, and callers |
| `/ubiquitous-language` | Extract domain terms into a canonical glossary with flagged ambiguities |

## Model Routing

The plugin routes each agent to a **capability tier** rather than a specific model. Three tiers cover the range of work:

| Tier | Capability | Agents | Rationale |
|------|-----------|--------|-----------|
| `reasoning` | Deep reasoning, design, complex analysis | spec-planner, spec-reviewer | Requirements, architecture, and review are where subtle mistakes are most expensive |
| `standard` | Structured code production, validation, documentation | spec-tasker, spec-validator, spec-implementer, spec-tester, spec-acceptor, spec-consultant, spec-documenter, spec-scanner | Fast and accurate for well-specified work |
| `lightweight` | Targeted fixes, small patches | spec-debugger | Throughput matters more than depth when the failure is already identified |

**Claude Code** resolves these tier aliases automatically (`reasoning` → `opus`, `standard` → `sonnet`, `lightweight` → `haiku`). **Codex** and other CLIs do not resolve aliases — set `SPEC_MODEL_*` environment variables to map each agent to a model your backend supports.

The `/spec` command delegates to these agents via the Task tool. Users don't need to manually switch models unless they want to override defaults.

Each agent's model can be overridden via environment variable (e.g., `SPEC_MODEL_PLANNER=deepseek-reasoner`). See [`docs/advanced/model-routing.md`](docs/advanced/model-routing.md) for mapping tables for OpenAI, Google, DeepSeek, local routers, and per-agent override details.

For implementation after spec completion, the `standard` tier is recommended — the spec provides all the context needed for accurate code generation.

## Key Concepts

### EARS Notation
All acceptance criteria use Easy Approach to Requirements Syntax:
```
WHEN [condition/trigger]
THE SYSTEM SHALL [expected behavior]
```

Variations include `WHILE` (state-driven), `IF/WHEN` (conditional), and `SHALL NOT` (negative).

### Spec File Location
Specs are created in the target project at `.claude/specs/<feature-name>/`:
- `requirements.md` - User stories with EARS acceptance criteria
- `design.md` - Architecture, components, data models
- `tasks.md` - Implementation tasks synced to Claude Code todos

### Project Profile

The project profile (`_project-profile.md`) captures codebase intelligence that agents use to wire new code correctly. Located at `.claude/specs/_project-profile.md`, it contains six sections:

1. **Stack** - Framework, language, backend, database, styling
2. **Patterns** - Detected code patterns with confidence levels (high/medium/low)
3. **Entity Registry** - Table of domain entities and their CRUD implementation status
4. **Registration Points** - Specific `file:line` locations where new artifacts must be registered
5. **Regression Markers** - Bug fixes with affected files and regression check descriptions
6. **Manual Overrides** - User-editable section preserved across rescans

The profile is auto-created on the first `/spec` run (Phase 0 auto-scan). For large codebases or monorepos, the scanner may split profiles by domain into `_profile-<domain>.md` files with a `_profile-index.md` listing.

### Task Synchronization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Habib0x0/spec-driven-plugin](https://github.com/Habib0x0/spec-driven-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
