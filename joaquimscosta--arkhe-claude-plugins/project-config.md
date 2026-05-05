---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Arkhe Claude Plugins** is a collection of Claude Code plugins providing specialized agents, commands, and skills for documentation, AI engineering, code review, UI/UX design, git workflows, Google Stitch prompting, Design Intent, Domain-Driven Design, and language-specific programming.

## Plugin Architecture

This repository uses a **marketplace-based plugin system** where each plugin is independently installable and provides:
- **Agents** - Specialized AI subagents with focused expertise
- **Commands** - Slash commands for specific workflows
- **Skills** - Auto-invoked capabilities triggered by context

### Marketplace Structure

```
arkhe-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog (13 plugins)
├── plugins/                       # All 13 plugins
│   ├── core/                      # Quality control and workflow orchestration
│   ├── ai/                        # AI engineering and LLM development
│   ├── doc/                       # Documentation generation
│   ├── review/                    # Code review and quality
│   ├── google-stitch/             # Google Stitch prompting toolkit
│   ├── git/                       # Git workflow automation
│   ├── design-intent/             # Design Intent for UI development
│   ├── lang/                      # Language-specific programming skills
│   ├── playwright/                # Browser automation via Playwright CLI
│   ├── spring-boot/               # Domain-Driven Design with Spring Boot 4
│   ├── ralph/                     # Autonomous development loop
│   ├── roadmap/                   # PM, roadmap analysis, solution architecture
│   ├── devtools/                  # Developer tooling and environment setup
│   └── startup/                   # Startup idea validation pipeline
├── docs/                          # Developer documentation
├── templates/                     # Plugin templates
└── assets/                        # Project assets
```

### Plugin Structure Pattern

Each plugin follows this structure:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json               # Plugin metadata (name, version, author)
├── agents/                        # Specialized AI subagents (*.md)
│   └── agent-name.md
├── commands/                      # Slash commands (*.md)
│   └── command-name.md
├── skills/                        # Auto-invoke skills
│   └── skill-name/
│       ├── SKILL.md              # Skill metadata and instructions
│       ├── WORKFLOW.md           # Detailed steps
│       ├── EXAMPLES.md           # Usage examples
│       ├── TROUBLESHOOTING.md    # Error handling
│       └── scripts/              # Executable Python scripts
└── README.md                     # Plugin documentation
```

## Available Plugins

### Core Plugin
Quality control and workflow orchestration utilities.
- **Agents**: `deep-think-partner`, `deep-researcher`, `code-explorer`, `code-architect`, `code-reviewer`, `systematic-debugger`
- **Commands**: `/discuss`, `/double-check` (`--deep` for multi-agent review), `/develop`, `/debug` (`--deep` for agent-assisted), `/think`, `/research`
- **Skills**: `sdlc-develop` (command-invoke), `deep-research` (auto-invoke), `workflow-orchestration` (auto-invoke)

### AI Plugin
AI engineering toolkit for production-ready LLM applications.
- **Agents**: `ai-engineer`, `prompt-engineer`, `context-manager`
- **Commands**: `/improve-agent`, `/multi-agent-optimize`
- **Skills**: `lyra` (auto-invoked for AI prompt engineering)

### Doc Plugin
Multi-purpose documentation toolkit with RFC management and documentation health.
- **Agents**: `rfc-critic`, `adr-critic`
- **Skills**: `doc-coauthoring`, `diagramming`, `code-explanation`, `jd-docs`, `diataxis`, `adr`, `rfc`, `doc-freshness`, `research-frontmatter` (all auto-invoke)
- **Commands**: `/code-explain`, `/diagram`, `/rfc`, `/health`
- **Use**: Documentation generation, code explanation, Mermaid diagrams, Johnny.Decimal management, Diataxis framework (audit, classify, validate, scaffold), ADR management, RFC lifecycle (create, review, list, update), documentation health (freshness, links, drift, cross-doc consistency), research frontmatter validation (JD-aware path resolution, RD-rules)

### Review Plugin
Code quality review tools for development teams.
- **Agents**: `pragmatic-code-review`, `design-review`, `false-positive-verifier`
- **Skills**: `code-review` (command-invoke), `security-review` (command-invoke), `verify-findings` (auto-invoke, context: fork), `design-review` (command-invoke)

### Google Stitch Plugin
Claude + Google Stitch workflow toolkit with MCP integration.
- **Commands**: `/prompt`, `/stitch-generate`, `/stitch-setup`
- **Skills**: `authoring-stitch-prompts`, `generating-stitch-screens`
- **Use**: Generate Stitch-ready prompts, automate screen generation via MCP

### Git Plugin
Git workflow automation for commits, pull requests, branching, changelog generation, releases, and Dependabot triage.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joaquimscosta/arkhe-claude-plugins](https://github.com/joaquimscosta/arkhe-claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
