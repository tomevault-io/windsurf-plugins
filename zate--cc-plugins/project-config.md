---
trigger: always_on
description: This repository is a marketplace for Claude Code plugins. This document provides guidance for agents working in this codebase on how to build, structure, and contribute plugins.
---

# Claude Code Plugin Marketplace - Developer Guide

This repository is a marketplace for Claude Code plugins. This document provides guidance for agents working in this codebase on how to build, structure, and contribute plugins.

## Table of Contents

- [Repository Structure](#repository-structure)
- [Official Plugin Structure](#official-plugin-structure)
- [Plugin Development Guidelines](#plugin-development-guidelines)
- [Development Workflow](#development-workflow)
- [Recommended Workflow Pattern (devloop)](#recommended-workflow-pattern-devloop)
- [Command Orchestration Pattern](#command-orchestration-pattern)
- [.devloop/ Directory Structure](#devloop-directory-structure)
- [Key Principles](#key-principles)
- [Marketplace Structure](#marketplace-structure)
- [Versioning Guidelines](#versioning-guidelines-abc)

---

## Repository Structure

```
cc-plugins/
├── .claude-plugin/
│   └── marketplace.json    # Marketplace configuration (required)
├── plugins/                # Individual plugin directories
├── templates/              # Templates for creating new plugins
├── docs/                   # Additional documentation
├── CLAUDE.md              # This file - guidance for agents
├── CONTRIBUTING.md        # Contribution guidelines
└── README.md              # Public-facing marketplace documentation
```

## Official Plugin Structure

Each plugin MUST follow this structure:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json        # Required manifest file
├── agents/                # Optional: Agent definitions (.md files)
├── skills/                # Optional: Skills and commands (subdirs with SKILL.md)
├── hooks/                 # Optional: Event handlers
├── .mcp.json             # Optional: MCP server configuration
├── .lsp.json             # Optional: LSP server configuration
├── settings.json         # Optional: Plugin default settings
└── README.md             # Plugin documentation
```

**Note**: `commands/` is legacy. Both `commands/deploy.md` and `skills/deploy/SKILL.md` create `/deploy`. Use `skills/` for new development.

**Critical**: Component directories (agents/, skills/, hooks/) MUST be at plugin root, NOT inside .claude-plugin/

## Plugin Development Guidelines

### Plugin Manifest (plugin.json)

Required location: `.claude-plugin/plugin.json`

**Required field:**
- `name`: Unique identifier in kebab-case format

**Common metadata:**
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Brief plugin purpose",
  "author": "Your Name",
  "homepage": "https://github.com/...",
  "repository": "https://github.com/...",
  "license": "MIT"
}
```

### Component Types

1. **Skills** (recommended): Model-invoked capabilities in `skills/` subdirectories
   - Each skill has its own directory with `SKILL.md`
   - Claude autonomously determines when to apply
   - Include "when to use" and "when NOT to use" sections
   - Frontmatter fields: `name`, `description`, `argument-hint`, `allowed-tools`, `model`, `context` (`fork`), `agent`, `hooks`, `user-invocable`, `disable-model-invocation`
   - Supports `$ARGUMENTS`, `$N` (positional), `${CLAUDE_SKILL_DIR}`, `${CLAUDE_SESSION_ID}`
   - Dynamic context injection: `` !`command` `` runs shell before content is sent to Claude

2. **Commands** (legacy, still works): Custom slash commands in `commands/` directory
   - Markdown files with frontmatter
   - Commands have been merged into skills -- both `commands/deploy.md` and `skills/deploy/SKILL.md` create `/deploy`
   - Use `skills/` for new development

3. **Agents**: Specialized subagents in `agents/` directory
   - Claude invokes automatically based on context
   - Frontmatter fields: `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory` (`user`/`project`/`local`), `background`, `isolation` (`worktree`)
   - The `Task` tool was renamed to `Agent` (v2.1.63) -- `Task(...)` still works as alias

4. **Hooks**: Event handlers responding to lifecycle events
   - Configure via `hooks.json` or inline in `plugin.json`
   - Four handler types: `command` (shell), `http` (POST), `prompt` (LLM eval), `agent` (multi-turn)
   - Events: `SessionStart`, `SessionEnd`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PermissionRequest`, `Stop`, `SubagentStart`, `SubagentStop`, `Notification`, `PreCompact`, `InstructionsLoaded`, `ConfigChange`, `WorktreeCreate`, `WorktreeRemove`, `TeammateIdle`, `TaskCompleted`, `Setup`
   - PreToolUse: use `hookSpecificOutput.permissionDecision` (not top-level `decision`/`reason` -- deprecated)
   - `async: true` for background hooks, `once: true` for run-once (skills only)

5. **MCP Servers**: External tool integrations
   - Configure in `.mcp.json`
   - Start automatically when plugin activates

6. **LSP Servers** (new): Language Server Protocol integration
   - Configure in `.lsp.json`
   - Provides code intelligence (go-to-definition, diagnostics)

### Environment Variables

Use `${CLAUDE_PLUGIN_ROOT}` for absolute plugin directory paths in:
- Hook scripts
- MCP configurations
- Any file references


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zate/cc-plugins](https://github.com/Zate/cc-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
