---
trigger: always_on
description: > **For AI Agents**: This document provides structured reference data for AI coding assistants. For human-readable documentation, see [README.md](README.md).
---

# AI Engineering Harness

> **For AI Agents**: This document provides structured reference data for AI coding assistants. For human-readable documentation, see [README.md](README.md).

## Quick Reference

**Purpose**: Configuration harness for AI coding agents with reusable prompts, agents, and workflows.

**Supported Tools**: OpenCode, Claude Code, Gemini CLI, Pi

**Installation**: `./setup.sh <tool>` where tool is `opencode`, `claude`, `gemini`, `pi`, or `all`

## Repository Structure

```
ai-engineering-harness/
├── opencode/           → ~/.config/opencode/
│   ├── agents/         # 6 agents (snake_case)
│   ├── commands/       # 11 slash commands
│   ├── skills/         # 15 skills (auto-triggered)
│   └── opencode.json   # MCP configuration
├── claude/             → ~/.claude/
│   ├── agents/         # 6 agents (snake_case)
│   ├── skills/         # 26 skills (13 manual + 13 auto)
│   ├── .mcp.json       # MCP configuration
│   └── settings.json   # Settings schema
├── gemini/             → ~/.gemini/
│   ├── agents/         # 6 agents (snake_case)
│   ├── commands/       # 14 commands (TOML format)
│   └── skills/         # 23 skills (auto-triggered)
├── pi/                 → ~/.pi/agent/
│   ├── agents/         # 6 agents (kebab-case)
│   ├── prompts/        # 11 prompt templates (Pi's commands)
│   ├── skills/         # 21 skills (auto-triggered)
│   └── extensions/     # subagent extension (multi-agent workflows)
└── thoughts/           # Context engineering artifacts
    ├── shared/tickets/ # Work items
    ├── shared/plans/   # Implementation plans
    ├── shared/research/# Research documents
    └── global/         # Cross-repo concerns
```

## Commands & Skills

| Command | OpenCode | Claude | Gemini | Pi | Type | Description |
|---------|:--------:|:------:|:------:|:--:|------|-------------|
| `/init_harness` | ✓ | ✓ | ✓ | ✓ | Manual | Initialize harness (creates AGENTS.md/CLAUDE.md/GEMINI.md + thoughts/) |
| `/create_plan` | ✓ | ✓ | ✓ | ✓ | Manual | Generate implementation plan from ticket |
| `/implement_plan` | ✓ | ✓ | ✓ | ✓ | Manual | Execute approved plan phase-by-phase |
| `/validate_plan` | ✓ | ✓ | ✓ | ✓ | Manual | Verify implementation against plan |
| `/commit` | ✓ | ✓ | ✓ | ✓ | Manual | Create well-structured git commits |
| `/debug` | ✓ | ✓ | ✓ | ✓ | Manual | Investigate issues during testing |
| `/debug_k8s` | ✓ | ✓ | ✓ | ✓ | Manual | Debug Kubernetes (prefers MCP, falls back to kubectl) |
| `/research_codebase` | ✓ | ✓ | ✓ | ✓ | Manual | Comprehensive codebase research |
| `/validate_telemetry` | ✓ | ✓ | ✓ | ✓ | Manual | Validate local telemetry against a narrative spec |
| `observability_driven_development` | ✓ | ✓ | ✓ | ✓ | Auto | Design the trace before the feature; local OTel feedback loop |
| `git_commit_helper` | ✓ | ✓ | ✓ | ✓ | Auto | Triggers on "commit" keywords |
| `pr_description_generator` | ✓ | ✓ | ✓ | ✓ | Auto | Triggers when creating PRs |
| `experimental_pr_workflow` | ✓ | ✓ | ✓ | ✓ | Auto | Formalizes experimental work |
| `interview` | ✓ | ✓ | ✓ | ✓ | Auto | Stress-test plans via relentless user interview |
| `improve_codebase_architecture` | ✓ | ✓ | ✓ | ✓ | Auto | Find architectural friction, propose deep-module refactors |
| `prd_to_issues` | ✓ | ✓ | ✓ | ✓ | Auto | Break a PRD into vertical-slice issue files |
| `tdd` | ✓ | ✓ | ✓ | ✓ | Auto | Red-green-refactor TDD discipline |
| `write_a_prd` | ✓ | ✓ | ✓ | ✓ | Auto | Generate a PRD from a client brief |

**Naming**: OpenCode, Claude, and Gemini use snake_case. Pi uses kebab-case (its native convention).

## Agents

All agents are shared across all four tools:

| Agent | OpenCode | Claude | Gemini | Pi | Purpose |
|-------|:--------:|:------:|:------:|:--:|--------|
| `codebase_analyzer` | ✓ | ✓ | ✓ | ✓ | Analyze implementation details, trace data flow |
| `codebase_locator` | ✓ | ✓ | ✓ | ✓ | Find files/directories by feature or task |
| `codebase_pattern_finder` | ✓ | ✓ | ✓ | ✓ | Discover similar implementations and patterns |
| `thoughts_analyzer` | ✓ | ✓ | ✓ | ✓ | Extract insights from research documents |
| `thoughts_locator` | ✓ | ✓ | ✓ | ✓ | Discover documents in thoughts/ directory |
| `web_search_researcher` | ✓ | ✓ | ✓ | ✓ | Research information from web sources |

## Workflow

```
Ticket → /create_plan → /implement_plan → /validate_plan → [/validate_telemetry] → /commit
```

1. Create ticket in `thoughts/shared/tickets/` (use ticket-template.md)
2. Run `/create_plan <ticket-path>` to generate plan
3. Run `/implement_plan <plan-path>` to execute
4. Run `/validate_plan` to verify
5. (Optional, telemetry-bearing features only) Run `/validate_telemetry [<spec>]` to verify the trace narrative
6. Run `/commit` to commit changes

## MCP Configuration

| Tool | File | Disable Syntax |
|------|------|----------------|
| OpenCode | `opencode.json` | `"enabled": false` |
| Claude Code | `.mcp.json` | `"disabled": true` |
| Gemini CLI | TBD | TBD |
| Pi | N/A | N/A |

Available MCP servers: `kubernetes` (disabled by default), `aspire-dashboard` (disabled by default; see [microsoft/aspire#14733](https://github.com/microsoft/aspire/issues/14733) for the standalone-Docker MCP caveat)

## Tool-Specific Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adrielp/ai-engineering-harness](https://github.com/adrielp/ai-engineering-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
