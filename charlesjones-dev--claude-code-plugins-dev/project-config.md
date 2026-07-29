---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a marketplace for custom Claude Code plugins, skills, and other extensions. The repository follows a structured plugin system where each plugin is self-contained with its own metadata and skills.

## Official Documentation

For detailed information about Claude Code plugins, refer to these official documentation resources:

1. **[Plugins](https://docs.claude.com/en/docs/claude-code/plugins)**: Core concepts and guide for creating and using Claude Code plugins
2. **[Plugins Reference](https://docs.claude.com/en/docs/claude-code/plugins-reference)**: Technical reference for plugin structure, metadata schemas, and command formats
3. **[Plugin Marketplaces](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces)**: Guide for creating and managing plugin marketplaces like this repository

These documentation resources provide authoritative information on plugin development best practices, metadata schemas, and marketplace structure that should be consulted when developing new plugins.

## Architecture

### Marketplace Structure

The repository uses a two-level metadata system:

1. **Root marketplace manifest** (`.claude-plugin/marketplace.json`): Central registry that defines:
   - Marketplace owner and metadata
   - List of all available plugins with their source paths, descriptions, and categories
   - Plugin root directory location (`./plugins`)

2. **Individual plugin manifests** (`plugins/{plugin-name}/.claude-plugin/plugin.json`): Each plugin has its own metadata including name, version, description, author, and license.

### Plugin Organization

Plugins are organized under the `plugins/` directory:
```
plugins/
  {plugin-name}/
    .claude-plugin/
      plugin.json          # Plugin metadata
    skills/
      {skill-name}/
        SKILL.md            # Skill implementation
    agents/                 # Optional
      {agent-name}.md       # Subagent definition
```

### Plugin Types

Plugins can include:
- **Skills**: Markdown files in `skills/{skill-name}/SKILL.md` directories that define executable skills invoked as slash commands
- **Agents**: Markdown files in `agents/{agent-name}.md` that define subagents for automated/unattended runs (e.g., `security-auditor`, `modernize-auditor`). Agents should use `model: inherit` rather than hardcoding a model ID, and skills that delegate to an agent must reference one that actually ships with the plugin (`{plugin-name}:{agent-name}`)
- Each skill is a markdown file with instructions for Claude Code to execute

## Development Workflow

### Recommended Setup

Before developing plugins in this repository, it's recommended to:

1. **Initialize security settings** using the ai-security plugin:
   ```bash
   /security-init
   ```
   This configures `.claude/settings.json` to prevent Claude Code from reading sensitive files (credentials, secrets, build artifacts, etc.) based on your project's technology stack.

   **Important**: Restart Claude Code after running this command for the settings to take effect.

### Adding a New Plugin

1. Create the plugin directory structure under `plugins/{plugin-name}/`
2. Add plugin metadata in `.claude-plugin/plugin.json` with:
   - name, version, description
   - author information
   - keywords for discoverability
   - repository and license
3. Implement skills in the `skills/{skill-name}/SKILL.md` directory structure
4. Register the plugin in the root `.claude-plugin/marketplace.json`

### Skill Development

Skills are markdown files located at `skills/{skill-name}/SKILL.md` that provide instructions to Claude Code. They should:
- Have a clear title and description
- Include step-by-step instructions under an "## Instructions" section
- Specify any important constraints or requirements (e.g., what NOT to include in outputs)

Example from git-commit-push skill (plugins/ai-git/skills/git-commit-push/SKILL.md):
```
IMPORTANT: Do not include the following in commit messages:
- 🤖 Generated with [Claude Code](https://claude.com/claude-code)
- Co-Authored-By: Claude <noreply@anthropic.com>
```

## Key Conventions

### Git Commit Messages

This repository has specific requirements for git commit messages:
- Follow the repository's existing commit message style (check recent commits with `git log`)
- Use conventional commit prefixes if the repository uses them (fix:, feat:, docs:, etc.)
- Do NOT include Claude Code attribution or co-author tags

### Plugin Metadata

When creating or updating plugins, ensure consistency between:
1. The plugin's own `plugin.json` file
2. The marketplace's root `marketplace.json` registry entry

Both should have matching:
- name
- version
- description
- author information

### Plugin Versioning

This repository follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html):
- **MAJOR** (x.0.0): Breaking changes or incompatible API changes
- **MINOR** (0.x.0): New features added in a backward-compatible manner
- **PATCH** (0.0.x): Backward-compatible bug fixes

#### Updating Plugin Versions

When adding new features or fixing bugs in a plugin, follow this workflow:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [charlesjones-dev/claude-code-plugins-dev](https://github.com/charlesjones-dev/claude-code-plugins-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
