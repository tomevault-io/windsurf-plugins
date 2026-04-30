---
trigger: always_on
description: This is the single source of truth for AI agents working on the Harness Engineering project. It provides essential context about our repository structure, architecture, conventions, and where to find information.
---

# Harness Engineering: AI Agent Knowledge Map

This is the single source of truth for AI agents working on the Harness Engineering project. It provides essential context about our repository structure, architecture, conventions, and where to find information.

## Project Overview

**Harness Engineering** is a comprehensive toolkit for transitioning from manual coding to **agent-first development**. We help teams architect software in ways that enable AI agents to execute, maintain, and improve code reliably and autonomously.

### Purpose and Goals

- Create a reusable library and toolkit for agent-first development
- Establish patterns that make AI-driven development predictable and scalable
- Document architectural decisions as the single source of truth
- Enforce constraints mechanically rather than through code review
- Measure and improve agent autonomy over time

### Current Phase

**Complete** — All core packages (types, core, cli, eslint-plugin, linter-gen, graph, orchestrator), 737 skills (claude-code and gemini-cli), 12 personas, 19 templates, and 3 progressive examples are implemented. The project is in adoption and refinement mode. See `examples/` for progressive tutorials.

## Repository Structure

This is a **monorepo** using pnpm workspaces and Turborepo for orchestration.

```
harness-engineering/
├── packages/                  # Core application packages
│   ├── types/                # Shared TypeScript types and interfaces
│   ├── core/                 # Core runtime library and utilities
│   ├── cli/                  # CLI tool (harness validate, check-deps, skill, state, etc.)
│   ├── eslint-plugin/        # ESLint rules for constraint enforcement
│   ├── linter-gen/           # YAML-to-ESLint rule generator
│   ├── graph/                # Unified Knowledge Graph: LokiJS store, ContextQL queries, code/git/knowledge ingestion, FusionLayer search, 4 external connectors (Jira, Slack, Confluence, CI)
│   ├── intelligence/         # Intelligence pipeline for spec enrichment, complexity modeling, and pre-execution simulation
│   ├── dashboard/            # Local web dashboard for project health and roadmap visualization
│   └── orchestrator/         # Agent orchestration daemon for dispatching coding agents to issues
├── agents/                    # Agent configuration
│   ├── skills/claude-code/   # 737 skills (skill.yaml + SKILL.md each)
│   ├── skills/gemini-cli/    # 737 skills (symlinked to claude-code for platform parity)
│   ├── skills/codex/         # 737 skills (symlinked to claude-code for platform parity)
│   ├── skills/cursor/        # 737 skills (symlinked to claude-code for platform parity)
│   ├── skills/templates/     # Shared discipline template (Evidence Requirements, Red Flags, Rationalizations to Reject)
│   └── personas/             # 12 personas (architecture-enforcer, code-reviewer, codebase-health-analyst, documentation-maintainer, entropy-cleaner, graph-maintainer, parallel-coordinator, performance-guardian, planner, security-reviewer, task-executor, verifier)
├── templates/                 # 19 project scaffolding templates (language bases + framework overlays: Express, NestJS, Next.js, FastAPI, Django, Gin, Axum, Spring Boot, React Vite, Vue, and more)
├── examples/                  # Progressive tutorial examples
│   ├── hello-world/          # Basic adoption level
│   ├── task-api/             # Intermediate adoption level
│   └── multi-tenant-api/     # Advanced adoption level
├── docs/                     # Complete documentation suite
│   ├── standard/            # Harness Engineering principles and standard
│   ├── guides/              # How-to guides and tutorials
│   ├── reference/           # Configuration and API reference
│   ├── changes/             # Design change proposals and technical specifications
│   ├── plans/               # Implementation and execution plans
│   ├── research/            # Framework research and analysis
│   └── conventions/          # Format conventions (markdown interaction patterns)
├── package.json             # Root package metadata and scripts
├── tsconfig.json            # Root TypeScript configuration
├── pnpm-workspace.yaml      # pnpm workspace definition
├── turbo.json               # Turborepo configuration
└── AGENTS.md                # This file - AI agent knowledge map
```

### Package Relationships

- **types** → Shared type definitions (no dependencies)
- **core** → Runtime library (depends on types)
- **graph** → Knowledge graph for codebase relationships and entropy detection (depends on types)
- **orchestrator** → Agent orchestration and multi-agent coordination (depends on core)
- All packages follow strict dependency rules: no circular dependencies, no upward dependencies

## Architecture

### Layered Architecture

We follow a strict, one-way dependency model:

```
Types (bottom layer - no dependencies)
  ↓
Configuration & Constants
  ↓
Repository & Data Access
  ↓
Services & Business Logic
  ↓
Agents & External Interfaces (top layer)
```

**Key Rule**: Layers can only depend on lower layers, never upward.

### Design Decisions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intense-Visions/harness-engineering](https://github.com/Intense-Visions/harness-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
