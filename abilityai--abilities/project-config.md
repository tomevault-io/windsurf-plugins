---
trigger: always_on
description: A skill that has never been individually edited carries a single honest `"1.0: Initial version — <summary>"` entry — do not fabricate history. Skills under `memory-templates/` are scaffolding copied into other agents, not invokable here, and are exempt. The generators (`create-playbook`, `create-wizard`) seed both parts; the modifiers (`adjust-playbook`, `update-wizard`) maintain them.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Abilities Plugin Marketplace** - a curated collection of Claude Code plugins from Ability.ai. The project is **documentation-driven**: plugins are defined through YAML frontmatter and procedural markdown instructions, not traditional code.

## Repository Structure

```
.claude-plugin/marketplace.json  # Central registry of all plugins
plugins/
  ├── create-agent/             # Create new agents (domain wizards + custom)
  ├── agent-dev/                # Develop agents (skills, memory, backlog, planning)
  ├── trinity/                  # Deploy to Trinity platform
  ├── dev-methodology/          # Documentation-driven development
  └── utilities/                # General-purpose utility skills
```

Each plugin follows this structure:
```
plugins/[name]/
  ├── .claude-plugin/plugin.json  # Plugin metadata
  ├── README.md                   # User documentation
  ├── skills/[skill-name]/        # Skill-based plugins
  │   ├── SKILL.md               # Skill definition with YAML frontmatter
  │   └── reference.md           # Technical reference
  └── commands/[cmd].md          # Command-based plugins
```

## Plugin Installation

```bash
# Add marketplace (one-time)
/plugin marketplace add abilityai/abilities

# Install plugins
/plugin install create-agent@abilityai    # Create new agents
/plugin install agent-dev@abilityai       # Add skills, memory, backlog
/plugin install trinity@abilityai         # Deploy to Trinity
/plugin install dev-methodology@abilityai # Documentation-driven dev
/plugin install utilities@abilityai       # General utilities

# Manual installation from local
/plugin add ./plugins/trinity
```

## Skill Definition Format

Skills use YAML frontmatter to declare metadata and permissions:

```yaml
---
name: skill-name
description: What the skill does
allowed-tools: Read, Write, Bash
user-invocable: true
argument-hint: "[optional args]"
---
# Procedural instructions follow in markdown
```

Key frontmatter fields:
- `allowed-tools`: Comma-separated list (NOT YAML array)
- `disable-model-invocation`: If true, Claude executes steps without additional reasoning
- `argument-hint`: Shows users expected arguments

## Adding a New Plugin

1. Create directory: `plugins/[plugin-name]/`
2. Add `.claude-plugin/plugin.json`:
   ```json
   {
     "name": "plugin-name",
     "description": "...",
     "version": "1.0.0",
     "author": { "name": "Ability.ai", "email": "support@ability.ai" },
     "license": "MIT"
   }
   ```
3. Add skills in `skills/[skill-name]/SKILL.md` or commands in `commands/[cmd].md`
4. Add `README.md` with usage documentation
5. Register in `marketplace.json` under the `plugins` array with explicit `skills` array (required for skill discovery)

## Marketplace Registration (Critical)

**Skills must be explicitly declared in `marketplace.json`.** Claude Code does NOT auto-discover skills from the `skills/` directory - they must be listed in a `skills` array for each plugin entry:

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin",
  "skills": [
    "./skills/skill-one",
    "./skills/skill-two"
  ]
}
```

**Never use `strict: false`** - it conflicts with plugins that have their own `plugin.json` files.

**When adding a new skill:**
1. Create the skill in `plugins/[name]/skills/[skill-name]/SKILL.md`
2. Add the skill path to the `skills` array in `marketplace.json`
3. Bump the plugin version in `plugin.json`

## Plugin Conventions

- **Report before action**: Generate audit/analysis first, then request approval for changes
- **Archive over delete**: Move files to `archive/` preserving structure instead of deleting
- **Safe artifacts are automatic**: `__pycache__`, `.pyc`, `.DS_Store` can be cleaned without approval
- **Templates use placeholders**: Files ending in `.example` or `.template` use `${VAR_NAME}` syntax
- **Always bump version on changes**: When modifying any plugin file (skills, commands, etc.), increment the patch version in `.claude-plugin/plugin.json`. This ensures client installations detect and sync the update.

## Version Bumping (Required)

**Every change to a plugin requires a version bump.** Without this, client installations won't receive updates when running `/plugin marketplace update`.

```bash
# Before: "version": "2.0.0"
# After:  "version": "2.0.1"
```

**Why:** Claude Code caches plugins locally at `~/.claude/plugins/cache/`. It uses the version number to detect changes. Same version = no update, even if files changed.

**Workflow:**
1. Make your changes to skills/commands/etc.
2. Bump the patch version in `plugins/[name]/.claude-plugin/plugin.json`
3. Commit both changes together

## Skill Changelog & What's-New Banner (Required)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Abilityai/abilities](https://github.com/Abilityai/abilities) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
