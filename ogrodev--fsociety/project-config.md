---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

fsociety is a multi-plugin marketplace for Claude Code offensive security plugins. Each plugin is a self-contained toolkit targeting a specific domain of penetration testing. The repo itself is NOT a Node.js application — there is no build step, no package.json, no test suite.

**Requirements**: Claude Code with MCP support, Kali Linux with Hexstrike MCP server, Node.js 18+.

## Repository Architecture

This is a **Claude Code plugin marketplace**. The root contains the marketplace registry and individual plugin directories:

- `.claude-plugin/marketplace.json` — Plugin registry listing all available plugins with their source directories
- Each plugin directory (e.g., `elliot/`) is a self-contained Claude Code plugin with its own `plugin.json`, `CLAUDE.md`, commands, skills, agents, scripts, and hooks

### Plugin Anatomy

Every plugin follows this structure:

```
plugin-name/
├── plugin.json           # Plugin definition (name, version, skills, agents)
├── CLAUDE.md             # Plugin-specific guidance for Claude Code
├── .gitignore            # Ignores runtime data files
├── commands/*.md         # Slash commands with YAML frontmatter
├── skills/*/SKILL.md     # Auto-activating skills with references/
├── agents/*.md           # Agent definitions with YAML frontmatter
├── scripts/*.js          # Node.js scripts (zero npm deps, stdlib only)
└── hooks/hooks.json      # Lifecycle hooks wiring scripts to events
```

### Marketplace Registration

`.claude-plugin/marketplace.json` is the index. To register a plugin, add an entry to the `plugins` array:

```json
{
  "name": "plugin-name",
  "description": "What it does.",
  "source": "./plugin-name/",
  "author": { "name": "author-name" }
}
```

### Plugin JSON Schema

Each plugin's `plugin.json`:

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "...",
  "author": { "name": "..." },
  "skills": ["./skills/", "./commands/"],
  "agents": ["./agents/agent-name.md"],
  "license": "MIT",
  "keywords": ["security", "pentest"]
}
```

## Current Plugins

| Plugin | Directory | Status | Domain |
|--------|-----------|--------|--------|
| **elliot** | `elliot/` | Available | Web & Application security — full offensive lifecycle |
| **romero** | `romero/` | Available | Reverse engineering — Windows binary analysis, decompilation, malware classification |
| **trenton** | `trenton/` | Available | Operational security — machine hardening, VPS security, anti-forensics, footprint elimination |
| **tyrell** | `tyrell/` | Available | Leak database hunting — exposed database discovery, data acquisition, cross-plugin pipeline to elliot |
| **fsociety** | `fsociety/` | Available | Engagement workspace setup — interactive wizard for targets, goals, scope, plugin selection, and cross-plugin orchestration |

See each plugin's `CLAUDE.md` for plugin-specific architecture, script CLI reference, data layer, and conventions.

Planned: **dom** (mobile/IoT).

## Engagement Setup

The `fsociety` plugin provides a `/setup` command to initialize engagement workspaces. This is the recommended starting point for any new operation.

### Quick Start

```bash
# Install the fsociety setup plugin
claude plugin install fsociety@fsociety

# Run the setup wizard
/setup my-operation
```

### What Setup Creates

| File | Purpose |
|------|---------|
| `engagement.json` | Central engagement config — targets, plugins, opsec, scope |
| `CLAUDE.md` | Tailored guidance with only active plugin commands/skills |
| `README.md` | Engagement overview and status |
| `scope.md` | Formal scope definition |
| `targets.jsonl` | Structured target list (append-only, SHA256-deduped) |
| `.gitignore` | Runtime data ignore patterns for all active plugins |

### OPSEC Profiles

| Profile | Speed | Anonymity | Use Case |
|---------|-------|-----------|----------|
| `surface` | Maximum | None | Lab/CTF |
| `standard` | Moderate | Basic | Authorized external tests |
| `paranoid` | Slow | Full (Tor/VPN) | Red team engagements |

## Conventions

- **Zero dependencies**: All scripts use only Node.js built-ins (`fs`, `path`, `crypto`, `child_process`)
- **Markdown-driven**: Commands, skills, and agents are defined in `.md` files with YAML frontmatter
- **Plugin isolation**: Each plugin is fully self-contained — no cross-plugin imports or shared scripts
- **Runtime data is gitignored**: Each plugin's `.gitignore` excludes generated data files (JSONL databases, reports, archives, session state)
- **Scripts reference `${CLAUDE_PLUGIN_ROOT}`**: Environment variable pointing to the plugin directory at runtime
- **JSONL for data**: Append-only JSONL files for findings, techniques, intel. JSON for session state
- **SHA256 deduplication**: All trackers deduplicate by hashing normalized key fields

## Command Frontmatter Schema

```yaml
---
description: What the command does
allowed-tools: Bash, Read, Write, Glob, Grep, Task, AskUserQuestion
argument-hint: <arguments>
---
```

## Agent Frontmatter Schema

```yaml
---
name: agent-name
description: |
  Trigger description with <example> blocks
color: cyan | red

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ogrodev/fsociety](https://github.com/ogrodev/fsociety) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
