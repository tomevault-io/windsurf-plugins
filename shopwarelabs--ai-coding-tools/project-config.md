---
trigger: always_on
description: **Skills are executable code, not documentation.** Markdown files in `skills/[skill-name]/` directories (SKILL.md, references/*.md) are instruction files that Claude Code reads and executes. Modifying these files changes the skill's behavior directly - treat them as you would Python or JavaScript code.
---

@README.md

# Shopware AI Coding Tools Marketplace - Technical Reference

## Understanding Skills

**Skills are executable code, not documentation.** Markdown files in `skills/[skill-name]/` directories (SKILL.md, references/*.md) are instruction files that Claude Code reads and executes. Modifying these files changes the skill's behavior directly - treat them as you would Python or JavaScript code.

## Understanding Slash Commands

**Slash commands are executable code, not documentation.** Markdown files in `commands/` directories are instruction files that Claude Code reads and executes when users invoke the command. Modifying these files changes what happens when users run the slash command.

## Understanding Developer Documentation

**AGENTS.md, README.md, and CHANGELOG.md inside plugins are developer documentation, not runtime code.** These files are read by humans maintaining this repository. Claude Code does not load or execute them when the plugin is installed. Changes to these files affect documentation only, not plugin behavior.

Runtime files (executed by Claude Code):
- `skills/*/SKILL.md` and `skills/*/references/*.md`
- `agents/*.md`
- `commands/*.md`
- `hooks/` (hooks.json and scripts)
- `.mcp.json`

When modifying runtime behavior (e.g. MCP tool references, workflow instructions), edit only runtime files. When updating architectural descriptions or usage guides, edit the developer documentation.

## Marketplace Architecture

This marketplace uses a **distributed metadata pattern** where plugin metadata is stored in individual `plugin.json` files rather than centralized in `marketplace.json`.

### Structure
```
.claude-plugin/marketplace.json       # Minimal registry (name + source only)
plugins/
  [category]/
    [plugin-name]/
      .claude-plugin/plugin.json      # Full plugin metadata
      ...                             # Plugin components
```

### marketplace.json Schema (Minimal Registry)

The marketplace configuration acts as a lightweight registry pointing to plugins. Each plugin entry only needs `name` and `source`.

**Required fields:**
- `name` - Marketplace identifier in kebab-case
- `owner` - Object with at least `name` property (optionally `email`, `url`)
- `plugins` - Array of plugin definitions

**Plugin entry structure (minimal):**
- `name` (required) - Plugin identifier in kebab-case
- `source` (required) - Relative path starting with `./`

**Optional marketplace-level metadata:**
- `metadata.description` - Marketplace description
- `metadata.version` - Marketplace version
- `metadata.pluginRoot` - Root directory for plugins

### plugin.json Schema (Per-Plugin Metadata)

Each plugin has its own `.claude-plugin/plugin.json` containing full metadata:

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "Plugin description",
  "author": { "name": "Author Name", "email": "email@example.com" },
  "license": "MIT",
  "keywords": ["tag1", "tag2"],
  "homepage": "https://github.com/...",
  "repository": "https://github.com/..."
}
```

**Fields:**
- `name` (required) - Plugin identifier in kebab-case
- `version` (required) - Semantic version string
- `description` - Full description of functionality
- `author` - Object with `name`, optionally `email` and `url`
- `license` - SPDX license identifier (e.g., "MIT", "Apache-2.0")
- `keywords` - Array of tags for discovery
- `homepage` - Documentation URL
- `repository` - Source code repository URL

## Plugin Component Types

Claude Code plugins can include any combination of these components:

- **Commands** - Custom slash commands (markdown files in `commands/`)
- **Agents** - Specialized subagents (markdown files in `agents/`)
- **Skills** - Model-invoked capabilities (`skills/[skill-name]/SKILL.md`)
- **Hooks** - Event handlers (configured via `hooks/hooks.json`)
- **MCP Servers** - External tool integration (`.mcp.json` configuration)

### MCP Server Cross-Plugin Dependencies

When an MCP config plugin needs to reference server code from another plugin, **do not use relative paths** like `${CLAUDE_PLUGIN_ROOT}/../other-plugin/`. This fails because the plugin cache uses versioned subdirectories (`plugin-name/1.0.0/`).

**Solution**: Use a wrapper script that dynamically discovers the dependency:

```bash
#!/bin/bash
# run-server.sh
CACHE_ROOT="$(dirname "$(dirname "$(cd "$(dirname "$0")" && pwd)")")"
SERVER=$(find "$CACHE_ROOT/dependency-plugin" -name "server.sh" -path "*/mcp-server/*" 2>/dev/null | sort -V | tail -1)
[ -z "$SERVER" ] && echo '{"jsonrpc":"2.0","error":{"code":-32603,"message":"dependency-plugin not found"}}' >&2 && exit 1
exec "$SERVER" "$@"
```

Reference in `.mcp.json`: `"command": "${CLAUDE_PLUGIN_ROOT}/run-server.sh"`

See `plugins/dev-tooling/` for implementation.

### Skills Directory Structure

Skills follow this pattern:
```
plugin-root/
└── skills/
    └── skill-name/
        └── SKILL.md
```

Example: `plugins/adr-writing/skills/adr-creating/SKILL.md`

## Commit Messages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shopwareLabs/ai-coding-tools](https://github.com/shopwareLabs/ai-coding-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
