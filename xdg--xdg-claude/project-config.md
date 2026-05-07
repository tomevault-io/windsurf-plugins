---
trigger: always_on
description: Use when analyzing code for security vulnerabilities.
---

# Claude Plugin Development Repository

This repository contains Claude Code plugins (agents, skills, commands, hooks) for extending Claude's capabilities. Use this guide as reference when developing, testing, and distributing plugins.

---

# Claude Code Plugins Reference

## What are Plugins?

Claude Code plugins are custom collections of commands, agents, skills, hooks, and MCP servers packaged in a Git repository. Plugins:

- Install with a single command (`/plugin`)
- Work across terminal and VS Code environments
- Enable sharing reusable workflows across projects and teams
- Automate development tasks and standardize practices
- Extend Claude's capabilities with specialized knowledge

## Plugin Architecture

### Component Types

**Skills** - User-invoked or model-invoked capabilities (replaces commands as of Claude Code 2.1.3)
- Defined in `skills/skill-name/SKILL.md` files with supporting resources
- Each skill isolated in its own directory
- Lazily loaded only when Claude determines they're needed
- Can route to agents via `context: fork` + `agent` frontmatter
- Use `disable-model-invocation: true` for manual-only skills (slash command only)
- Best for: Domain expertise, complex workflows, agent routing, specialized knowledge

**Commands** (legacy) - Custom slash commands; use skills instead
- Defined in `commands/*.md` files
- As of Claude Code 2.1.3, commands and skills are merged; prefer `skills/`
- Existing commands still work but `skills/` is the recommended path

**Agents (Sub-agents)** - Specialized AI assistants with isolated context
- Defined in `agents/*.md` files
- Have their own system prompt and tool permissions
- Routed to via skill frontmatter (`agent` field) or Task tool (`subagent_type`)
- Best for: Intelligent analysis requiring reasoning and adaptation

**Hooks** - Event handlers that run shell commands at lifecycle points
- Defined in `hooks/hooks.json` files with optional `hooks/references/` for content
- Can validate, block, or enhance tool usage
- SessionStart hooks can inject context from reference files
- Best for: Automation, validation, code formatting, injecting session context

**MCP Servers** - Model Context Protocol integrations
- Defined in `.mcp.json` file
- External tool and service integrations
- Best for: Database access, API integrations, system tools

### Standard Directory Structure

```
plugin-name/
├── .claude-plugin/
│   ├── plugin.json          # Required: plugin manifest
│   └── marketplace.json     # Optional: for distribution
├── skills/                  # Skills (user-invoked and model-invoked)
│   └── skill-name/         # Each skill in its own directory
│       ├── SKILL.md
│       ├── scripts/        # Executable code
│       ├── reference/      # Documentation loaded as needed
│       └── assets/         # Files used in output
├── agents/                  # Specialized sub-agents
│   └── helper.md
├── commands/                # (legacy) Use skills/ instead
│   └── example.md
├── hooks/                   # Hook configurations
│   ├── hooks.json
│   ├── session-start.sh    # Hook scripts
│   └── references/         # Content files for hooks to load
└── .mcp.json               # MCP server definitions
```

**Critical:** All component directories must be at plugin root, NOT inside `.claude-plugin/`.

---

# Deep Dive: Agent Skills

## What Makes Skills Special

Skills transform Claude from general-purpose to specialized agent through **progressive disclosure** - a three-level loading system that minimizes context usage while maximizing capability:

**Level 1: Metadata (always loaded, ~100 words)**
- Name and description in SKILL.md frontmatter
- Pre-loaded into system prompt at startup
- Gives Claude just enough info to know when to use each skill

**Level 2: SKILL.md body (loaded when triggered, <5k words)**
- Procedural instructions and workflow guidance
- Loaded only when Claude determines skill is relevant
- Keeps core instructions lean and focused

**Level 3: Bundled resources (loaded on-demand, unlimited)**
- `scripts/` - Executed without reading into context
- `reference/` - Documentation loaded only when needed
- `assets/` - Files used in output, never pollute context

## Skills vs Traditional Approaches

**Traditional:** All instructions loaded upfront, consuming context regardless of relevance

**Skills approach:**
- **Discoverable** - Claude can browse available capabilities via metadata
- **Lazy-loaded** - Content loaded only when needed
- **Executable** - Scripts run deterministically without token generation
- **Composable** - Multiple skills combine for complex workflows
- **Specialized** - Each skill focuses on specific domain

## Skill Design Principles

### Content Organization

**SKILL.md frontmatter fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill identifier (used in slash commands) |
| `description` | Yes | What skill does and when to use it (third-person) |
| `context` | No | `fork` to run skill body as a task prompt in a subagent |
| `agent` | No | Agent name to route to (requires `context: fork`) |
| `disable-model-invocation` | No | `true` for manual-only skills (slash command only) |

**SKILL.md structure (knowledge skill):**
```markdown
---
name: skill-name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xdg/xdg-claude](https://github.com/xdg/xdg-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
