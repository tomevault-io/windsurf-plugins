---
trigger: always_on
description: <!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
---

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# GitHub Copilot Instructions for AL Development

## Overview

This workspace contains AL (Application Language) code for Microsoft Dynamics 365 Business Central. It uses the **ALDC Core v1.1** skills-based architecture: **4 agents + 11 skills + 6 workflows + 9 instructions**.

## Core Principles

These principles apply to ALL work in this repository:

- **Extension-only development** — Never modify base application objects. Use tableextensions, pageextensions, event subscribers.
- **Human-in-the-Loop (HITL)** — All critical decisions require user confirmation before proceeding.
- **TDD / spec-driven** — Features follow the flow: `spec.create → architecture → test-plan → implementation → review`.
- **Least privilege** — Generate only the minimum permissions required. Use XLIFF for all user-facing strings.

## Agent Routing

Choose the right agent for your task:

| Intent | Agent | What it does |
|--------|-------|-------------|
| Designing, analyzing architecture, strategic decisions? | `@AL Architecture & Design Specialist` | Solution design, data modeling, integration strategy |
| Implementing, coding, debugging, fixing? | `@AL Implementation Specialist` | Tactical implementation with full AL MCP tools |
| Building a feature with TDD orchestration (plan → implement → review → commit)? | `@AL Development Conductor` | Orchestrates planning, implementation, and review subagents |
| Estimating a project, sizing, proposals? | `@AL Pre-Sales & Project Estimation Specialist` | PERT estimation, SWOT analysis, cost breakdown |

### Quick routing guide

```
New feature (MEDIUM/HIGH)? → @AL Architecture & Design Specialist → al-spec.create → @AL Development Conductor
New feature (LOW)?         → al-spec.create → @AL Implementation Specialist
Bug fix / debugging?       → @AL Implementation Specialist
Architecture review?       → @AL Architecture & Design Specialist
Full TDD cycle?            → @AL Development Conductor
Project estimation?        → @AL Pre-Sales & Project Estimation Specialist
```

## Workflows

6 workflows available via `@workspace use [name]`:

| Workflow | When to use |
|----------|-------------|
| `al-spec.create` | Create functional-technical specifications before development |
| `al-build` | Build, package, and deploy extensions |
| `al-pr-prepare` | Prepare pull requests with documentation and validation |
| `al-memory.create` | Generate/update memory.md for session continuity |
| `al-context.create` | Generate project context.md for AI assistants |
| `al-initialize` | Complete environment and workspace setup |

### Usage

```
@workspace use al-spec.create    # Create specification
@workspace use al-build          # Build & deploy
@workspace use al-pr-prepare     # Prepare PR
@workspace use al-initialize     # Setup project
```

## Skills

11 composable knowledge modules loaded on-demand by agents. You don't invoke skills directly — agents load them automatically when the task requires domain-specific knowledge.

| Skill | Domain | Loaded by |
|-------|--------|-----------|
| `skill-debug` | Debugging, diagnosis, snapshot debugging | al-developer |
| `skill-api` | API pages, OData, REST endpoints | al-developer, al-architect |
| `skill-copilot` | AI features, PromptDialog, AI Test Toolkit | al-developer, al-architect |
| `skill-events` | Event subscribers, publishers, handled pattern | al-developer, al-architect |
| `skill-permissions` | Permission sets, XLIFF, security | al-developer |
| `skill-pages` | Page types, FastTabs, actions, dynamic UI | al-developer |
| `skill-migrate` | BC version migration, upgrade codeunits, rollback | al-developer |
| `skill-translate` | XLF translation, NAB AL Tools, quality review | al-developer |
| `skill-performance` | CPU profiling, FlowField optimization, set-based ops | al-developer, al-architect |
| `skill-testing` | TDD, test strategy, AL Test Toolkit | al-architect, al-conductor |
| `skill-estimation` | PERT estimation, complexity scoring, SWOT | al-presales |

## Skills Evidencing

Agents MUST declare which skills they loaded and which patterns they applied:

- **al-architect** → `> **Skills applied**: skill-api, skill-events` at top of architecture.md
- **al-developer** → `> **Skills loaded**: skill-debug (root cause analysis)` at start of response
- **AL Implementation Subagent** → `### Skills Loaded` section in Phase Summary returned to Conductor
- **AL Code Review Subagent** → `Skills Compliance Check` checklist verifying patterns were applied
- **al-conductor** → `Skills Applied in This Phase` table in phase-complete.md; `Skills Utilization Summary` in plan-complete.md

This traceability chain ensures every skill application is auditable end-to-end.

## Auto-Applied Instructions

These instruction files activate automatically based on file type — no invocation needed:

### Always active on `*.al` files

- **al-guidelines.instructions.md** — Master hub referencing all guidelines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [javiarmesto/ALDC-AL-Development-Collection](https://github.com/javiarmesto/ALDC-AL-Development-Collection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
