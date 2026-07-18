---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code plugin marketplace containing custom plugins that extend Claude Code functionality. The repository is structured as a collection of independent plugins, each providing agents, commands, skills, or hooks.

**Author Name**: Tobey Forsman (must be used for all plugin metadata)

## Repository Structure

```
rad-cc-plugins/
├── marketplace.json          # Registry of all plugins in this marketplace
├── plugins/
│   ├── git-github-operations/   # Git workflow automation
│   ├── agent-architect/         # Tools for designing subagents
│   ├── github-issues/           # GitHub Issues management via gh CLI
│   ├── backlog-md-cli/           # Task management with Backlog.md CLI
│   └── astro-content-author/    # Astro content creation and management
```

## Plugin Architecture

Each plugin follows this structure:

- `plugin.json` - Plugin metadata (name, version, author, components)
- `agents/*.md` - Subagent definitions (frontmatter + instructions)
- `commands/*.md` - Slash commands (prompts for specific workflows)
- `skills/*/SKILL.md` - Reusable skill modules
- `hooks/hooks.json` - Event-triggered shell commands
- `docs/*.md` - Plugin-specific documentation

### Plugin Metadata Requirements

All `plugin.json` files must include:

- `name`: kebab-case plugin identifier
- `version`: Semantic version (e.g., "1.0.0")
- `description`: Clear description of plugin purpose
- `author`: Object with `name` and `email` fields (see example below)
- Component arrays: `agents`, `commands`, `skills` (as applicable)

### Plugin.json Schema (CRITICAL)

**Adding invalid fields or structures causes "Invalid input" validation errors.** Always reference working plugins when unsure.

**Agents array** - each object must have:
```json
{
  "name": "agent-name",
  "source": "./agents/agent-name.md",
  "description": "Brief description"
}
```
- Use `source` (webapp-team style) OR `path` (simbl style) - both work
- All three fields are required

**Skills array** - each object must have:
```json
{
  "name": "skill-name",
  "source": "./skills/skill-name"
}
```
- Points to directory containing `SKILL.md`
- `description` field is optional for skills

**Commands array** - each object must have:
```json
{
  "name": "command-name",
  "source": "./commands/command-name.md",
  "description": "Brief description"
}
```

**Reference plugins for validation**:
- `plugins/webapp-team/plugin.json` - comprehensive example with agents, commands, skills
- `plugins/simbl/plugin.json` - example using `path` instead of `source`

### Agent Definition Format

Agents use YAML frontmatter followed by markdown instructions:

```yaml
---
name: agent-name
description: When to use this agent. Use PROACTIVELY after X.
tools: Read, Grep, Glob, Bash
model: inherit
color: blue
---
```

See `plugins/agent-architect/templates/claude-code-subagent-template.md` for the complete template.

### Agent Frontmatter Schema (CRITICAL)

**ONLY use these validated frontmatter fields** in agent `.md` files. Adding non-standard fields causes plugin validation errors ("agents: Invalid input").

**Standard Fields** (always safe):
- `name` - Agent identifier (required)
- `description` - When to use this agent (required)
- `tools` - Comma-separated tool list (required)
- `model` - Always use `inherit` (required)

**Optional Fields** (validated in working plugins):
- `color` - Display color (e.g., `"#2563eb"`, `blue`)
- `role` - Role title for display
- `expertise` - List of expertise areas
- `triggers` - List of trigger conditions

**INVALID Fields** (will cause validation errors):
- ❌ `skills` - Not a valid agent frontmatter field
- ❌ Any other custom fields not listed above

**When in doubt**: Compare your agent frontmatter against working agents in `plugins/webapp-team/agents/` or `plugins/simbl/agents/`.

### Command Definition Format

Commands use YAML frontmatter followed by markdown instructions:

```yaml
---
name: command-name
description: Brief description of what this command does
tools: Bash, Read, Write
model: inherit
---
```

### Model Specification Guidelines

**IMPORTANT**: Always use `model: inherit` in agent and command frontmatter.

**Valid Model Specifications**:
- `model: inherit` - **RECOMMENDED**: Inherits from parent/user settings
- `model: claude-sonnet-4-5-20250929` - Full model ID (rarely needed)
- Omit field entirely - Uses default model

**Invalid Model Specifications** (will cause 404 errors):
- ❌ `model: sonnet` - Not a valid model identifier
- ❌ `model: opus` - Not a valid model identifier
- ❌ `model: haiku` - Not a valid model identifier

**Why `inherit` is preferred**:
- Respects user's model preferences
- Easier to maintain across plugin updates
- Consistent with marketplace standards
- Flexible for different deployment contexts

## Development Workflow

### Creating New Plugins

1. Create directory under `plugins/{plugin-name}/`
2. Create `plugin.json` with required metadata (see Plugin Metadata Requirements)
3. Add components (agents/commands/skills/hooks)
4. Register in root `marketplace.json`

### Creating Agents


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yebot/rad-cc-plugins](https://github.com/yebot/rad-cc-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
