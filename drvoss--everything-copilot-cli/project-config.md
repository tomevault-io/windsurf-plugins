---
trigger: always_on
description: > The equivalent of `CLAUDE.md` for the GitHub Copilot CLI ecosystem.
---

# Copilot Instructions — everything-copilot-cli

> The equivalent of `CLAUDE.md` for the GitHub Copilot CLI ecosystem.
> This file tells Copilot CLI how to understand, navigate, and contribute to this repository.

---

## Project Overview

**everything-copilot-cli** is the definitive guide and configuration system for GitHub Copilot CLI.
It provides a structured collection of agents, skills, rules, orchestration patterns,
MCP configurations, and multi-AI workflows.

This is a **reference repository** — it defines conventions, patterns, and reusable configurations
that projects can adopt. When working in this repo, Copilot CLI should treat content as
**configuration and documentation**, not application code.

## Response Voice

Write like a teammate handing off to another builder:

- lead with the outcome, not process narration
- stay concrete and repository-specific
- prefer short, dense explanations over marketing language
- avoid AI self-reference, consultant filler, and vague encouragement
- when a next step matters, name it plainly instead of hedging

### What This Repo Contains

| Directory | Purpose |
|-----------|---------|
| `agents/` | Agent definitions (persona, tools, model, behavior) |
| `skills/` | Composable skill modules organized by domain |
| `rules/` | Behavioral rules (common + language-specific) |
| `orchestration/` | Multi-agent coordination patterns |
| `contexts/` | Context definitions for scoped execution |
| `mcp-configs/` | Model Context Protocol server configurations |
| `guides/` | Documentation and usage guides |
| `examples/` | Complete example projects (Next.js, Python, .NET, monorepo) |
| `scripts/` | Validation, setup, and utility scripts |
| `tests/` | Configuration validation tests |

---

## Key Concepts

### Agents

Agents are personas with defined responsibilities, tool permissions, and behavioral guardrails. Each agent has:

- **Identity**: Name, role description, expertise areas
- **Tools**: Which tools the agent can use (e.g., `grep`, `edit`, `powershell`)
- **Model**: Recommended AI model (e.g., `claude-sonnet-4.6`, `gpt-5-mini`)
- **Behavioral Rules**: How the agent should approach tasks
- **Escalation Policy**: When to hand off to another agent or human

### Skills

Skills are composable capabilities that agents can invoke. They are organized by domain:

- `skills/copilot-exclusive/` — GitHub-specific integrations (PRs, Issues, Actions, fleet, background agents)
- `skills/development/` — Code generation, refactoring, debugging
- `skills/documentation/` — Doc generation, README updates, API docs
- `skills/security/` — Vulnerability scanning, secret detection
- `skills/testing/` — Test generation, coverage analysis, TDD workflows
- `skills/workflow/` — End-to-end development workflows (sprint, security audit, retrospective)
- `skills/product/` — Product management (OST framework, feature prioritization, launch strategy)
- `skills/content/` — Content strategy and AI visibility (GEO, llms.txt, SEO)

Skills follow the [agentskills.io](https://agentskills.io) spec: each skill lives in a
`skill-name/SKILL.md` directory, not a flat `.md` file. This ensures compatibility with
`agy skills install`, Codex CLI, and other skill-compatible tools.

See [`skills/README.md`](skills/README.md) for the full catalog and installation instructions.

Each skill file should include a **"When to Use"** section describing trigger conditions.

### Rules

Rules define behavioral constraints and coding standards:

- `rules/common/` — Universal rules (commit formats, file naming, error handling)
- `rules/languages/` — Language-specific rules (TypeScript conventions, Python style, etc.)

### Orchestration Patterns

Orchestration defines how multiple agents collaborate:

- `orchestration/patterns/` — Reusable coordination patterns (pipeline, fan-out, review-chain, sub-agent-sandboxing)
- `orchestration/configs/` — Configuration for specific orchestration setups
- `orchestration/examples/` — Worked examples of multi-agent workflows
- `orchestration/skills/` — Orchestration-specific skill combinations

---

## Architecture

```text
┌─────────────────────────────────────────────────┐
│                   User Request                   │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│              Orchestration Layer                  │
│  (patterns/ configs/ — decides agent routing)    │
└──────┬──────────┬──────────┬──────────┬─────────┘
       │          │          │          │
       ▼          ▼          ▼          ▼
┌──────────┐┌──────────┐┌──────────┐┌──────────┐
│  Agent 1 ││  Agent 2 ││  Agent 3 ││  Agent N │
│(planner) ││(architect)││(reviewer)││  (...)   │
└────┬─────┘└────┬─────┘└────┬─────┘└────┬─────┘
     │           │           │           │
     ▼           ▼           ▼           ▼
┌─────────────────────────────────────────────────┐
│                  Skills Layer                    │
│  (composable capabilities agents can invoke)     │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│                  Rules Layer                     │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drvoss/everything-copilot-cli](https://github.com/drvoss/everything-copilot-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
