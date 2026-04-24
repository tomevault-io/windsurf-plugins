---
trigger: always_on
description: > **Scalable Requirements Engineering for AI-Assisted Development**
---

# syspilot - Copilot Instructions

> **Scalable Requirements Engineering for AI-Assisted Development**

## Project Overview

syspilot is a requirements engineering toolkit that uses **sphinx-needs traceability links** to provide focused context to AI agents. Instead of scanning entire codebases, syspilot follows links from User Stories → Requirements → Design Specs to find only the affected elements.

**Key Insight**: AI agents need focused context, not the entire codebase. syspilot achieves O(affected) not O(total) complexity.

**Principle**: Spec-driven development for everything — not just the product, but also processes, methods, and tools. Every decision is traceable through User Stories → Requirements → Design Specs.

## Tech Stack

- **Documentation**: Sphinx with sphinx-needs extension
- **Markup**: reStructuredText (RST) + Markdown (MyST)
- **Diagrams**: Mermaid via sphinxcontrib-mermaid (client-side rendering)
- **Python Runner**: uv (Astral's fast Python package manager)
- **Theme**: Furo
- **Process Alignment**: Optional (see `docs/syspilot/process/`)

## Project Structure

```
syspilot/                            # Repository root (also the product dir)
├── .github/
│   ├── agents/                 # Installed agent files (*.agent.md)
│   ├── prompts/                # Prompt configuration files (*.prompt.md)
│   ├── skills/                 # Shared skill files (*/SKILL.md)
│   └── copilot-instructions.md # This file
├── projects/                        # Jarvis project folders (role-based)
│   ├── project-manager/        # PM: planning, backlog, delegation
│   │   ├── project.yaml
│   │   └── context.md
│   ├── change-manager/         # CM: E2E change orchestration
│   │   ├── project.yaml
│   │   └── context.md
│   └── quality-manager/        # QM: independent quality checks
│       ├── project.yaml
│       └── context.md
├── scripts/
│   └── python/
│       └── get_need_links.py   # Link discovery utility (consumer copy)
├── syspilot/                    # syspilot family product artifacts
│   ├── version.json            # Release version
│   ├── agents/                 # Distributable agents (product)
│   ├── prompts/                # Distributable prompts
│   ├── skills/                 # Distributable skills
│   ├── scripts/python/         # Distributable scripts
│   ├── sphinx/                 # Sphinx build script templates
│   └── templates/              # Document templates
│       └── change-document.md  # Change Document template
├── docs/
│   ├── methodology.md          # Framework methodology
│   ├── architecture.md         # Product/Instance concept
│   ├── workflows.md            # Workflows & branching strategy
│   ├── namingconventions.md    # Framework naming conventions
│   ├── releasenotes.md         # Release notes (newest first)
│   ├── syspilot/               # syspilot family specs (product-level)
│   │   ├── userstories/        # Level 0: WHY (User Stories)
│   │   ├── requirements/       # Level 1: WHAT (Requirements)
│   │   ├── design/             # Level 2: HOW (Design Specs)
│   │   ├── process/            # A-SPICE process alignment
│   │   ├── methodology.md      # Family-specific methodology
│   │   └── namingconventions.md # Family-specific naming
│   ├── traceability/           # Cross-family traceability matrices
│   ├── changes/                # Change Documents
│   │   └── archive/            # Archived by version after release
│   ├── conf.py                 # Sphinx configuration
│   └── requirements.txt        # Python dependencies for Sphinx
└── README.md                   # Installation instructions
```

## Specification Hierarchy

```
Level 0: User Stories (WHY)     docs/syspilot/userstories/    SYSP_US_*
         │ :links:
         ▼
Level 1: Requirements (WHAT)    docs/syspilot/requirements/   SYSP_REQ_*
         │ :links:
         ▼
Level 2: Design Specs (HOW)     docs/syspilot/design/         SYSP_SPEC_*
```

## Agent System

### Managers (user-invocable)

| Agent | Persona | Purpose |
|-------|---------|--------|
| `@syspilot.pm` | Project Manager | Feature discussion, backlog, research, delegation |
| `@syspilot.cm` | Change Manager | E2E change orchestration, quality gates |
| `@syspilot.qm` | Quality Manager | Independent quality checks, MECE/Trace dispatch |

### Engineers (subagents)

| Agent | Persona | Purpose |
|-------|---------|--------|
| `@syspilot.design` | System Designer | Level-by-level change analysis, RST writing |
| `@syspilot.implement` | Dev Engineer | Code implementation from specs |
| `@syspilot.uat` | Test Engineer | UAT artifact generation |
| `@syspilot.docu` | Documentation Engineer | Internal + external doc maintenance |
| `@syspilot.mece` | Quality Engineer MECE | Horizontal consistency checks |
| `@syspilot.trace` | Quality Engineer Trace | Vertical traceability checks |
| `@syspilot.release` | Release Engineer | Version, tag, release notes |
| `@syspilot.setup` | Setup Engineer | Installation and updates |

## Role-Based Development (Jarvis)

syspilot uses Jarvis as project management tool. Three manager roles, each with their own `projects/<role>/context.md`:

| Role | Folder | Responsibility |
|------|--------|---------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enthali/syspilot](https://github.com/enthali/syspilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
