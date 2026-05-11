---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Code plugin marketplace** for Frappe Framework development tools. It contains the `frappe-apps-manager` plugin, which provides commands, agents, and skills for building Frappe applications.

## Architecture

### Marketplace Structure

```
frappe-marketplace/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace metadata
└── frappe-apps-manager/          # Main plugin package
    ├── .claude-plugin/
    │   └── plugin.json           # Plugin metadata
    ├── commands/                 # Slash commands (7 total)
    ├── agents/                   # Specialized agents (3 total)
    ├── skills/                   # Model-invoked skills (3 total)
    └── hooks/                    # Automated workflows
        └── hooks.json
```

### Plugin Components

**Commands** (`commands/*.md`): User-invoked workflows with guided steps
- `/frappe-new-app` - App creation
- `/frappe-new-doctype` - DocType creation
- `/frappe-install-app` - App installation
- `/frappe-bench-start` - Start bench
- `/frappe-migrate` - Run migrations
- `/frappe-backup` - Backup site
- `/frappe-deploy` - Production deployment

**Agents** (`agents/*.md`): Specialized AI assistants for specific domains
- `frappe-developer.md` - App and DocType development expert
- `frappe-debugger.md` - Troubleshooting and error analysis
- `frappe-architect.md` - System design and architecture planning

**Skills** (`skills/*/SKILL.md`): Autonomous capabilities invoked by Claude
- `frappe-doctype-builder` - Generate complete DocType JSON structures
- `frappe-api-handler` - Create whitelisted API methods
- `frappe-report-generator` - Build custom reports

**Hooks** (`hooks/hooks.json`): Automated workflows triggered by events
- `pre-tool-use`: Detect Frappe bench context before operations
- `post-tool-use`: Suggest migrations after DocType changes
- `user-prompt-submit`: Validate bench setup for Frappe operations

## Key Design Patterns

### Command Structure
Commands are markdown files with YAML frontmatter containing:
- `description`: Brief summary shown in `/help`
- Main content: Step-by-step instructions for Claude to follow

### Agent Structure
Agents are markdown files defining:
- Role and expertise areas
- Responsibilities and capabilities
- Communication style and best practices
- Common tasks and workflows

### Skill Structure
Skills have `SKILL.md` files with:
- `name` and `description` in YAML frontmatter
- "When to Use" section (Claude uses this to decide invocation)
- Capabilities with code examples
- Best practices and patterns
- Output format specifications

### Hook Configuration
Hooks in `hooks.json` define:
- `name`: Hook identifier
- `description`: Purpose
- `tool_names`: Which tools trigger it (Bash, Write, Edit, etc.)
- `keywords`: User prompt keywords that trigger it
- `command`: Shell command to execute

## Development Workflow

### Plugin Management Commands

```bash
# Add this marketplace
/plugin marketplace add ./frappe-marketplace

# Install the plugin
/plugin install frappe-apps-manager@frappe-marketplace

# Manage plugins
/plugin                           # Interactive plugin management UI
/plugin enable plugin-name        # Enable a disabled plugin
/plugin disable plugin-name       # Disable without uninstalling
/plugin uninstall plugin-name     # Completely remove a plugin

# Verify installation
/help                            # Should show all Frappe commands
```

### Testing the Plugin Locally

```bash
# From the parent directory containing frappe-marketplace/
claude

# In Claude Code session:
/plugin marketplace add ./frappe-marketplace
/plugin install frappe-apps-manager@frappe-marketplace

# After making changes to plugin code:
/plugin uninstall frappe-apps-manager@frappe-marketplace
/plugin install frappe-apps-manager@frappe-marketplace
# (Restart Claude Code to reload changes)
```

### Adding a New Command

1. Create `commands/command-name.md` with frontmatter:
   ```markdown
   ---
   description: Brief description (under 100 chars)
   ---

   # Command Name

   Step-by-step instructions for Claude to follow...
   ```

2. Plugin auto-discovers commands in the `commands/` directory

3. Test: Restart Claude Code, then `/command-name` should work

### Adding a New Agent

1. Create `agents/agent-name.md` with:
   ```markdown
   ---
   description: Brief description of agent's role
   ---

   # Agent Name

   You are a specialized [role] expert...

   ## Core Expertise
   - Area 1
   - Area 2

   ## Responsibilities
   [What the agent does]

   ## Communication Style
   [How the agent responds]
   ```

2. Users invoke agents by describing tasks naturally or explicitly requesting the agent

3. Agents appear in `/agents` command after installation

### Adding a New Skill

1. Create `skills/skill-name/SKILL.md`:
   ```markdown
   ---
   name: skill-name
   description: Clear description of when Claude should use this skill
   ---

   # Skill Name

   ## When to Use This Skill

   Claude should invoke this skill when:
   - Trigger condition 1
   - Trigger condition 2

   ## Capabilities

   ### Feature 1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Venkateshvenki404224/frappe-apps-manager](https://github.com/Venkateshvenki404224/frappe-apps-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
