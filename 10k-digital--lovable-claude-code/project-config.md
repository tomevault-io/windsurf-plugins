---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** for integrating with Lovable.dev projects. It's distributed as a plugin, not a typical software project with build steps or test suites.

- **Repository**: https://github.com/10K-Digital/lovable-claude-code
- **Current Version**: 1.9.0
- **Type**: Claude Code plugin marketplace (supports multiple plugins)
- **Distribution**: Via Claude Code plugin marketplace (10K-Digital/lovable-claude-code)

## Architecture

### Repository Structure (Multi-Plugin)

This repository is a **plugin marketplace** that can host multiple plugins. Currently contains the `lovable` plugin:

```
.claude-plugin/              # Marketplace metadata (publisher-level)
└── marketplace.json         # Lists all plugins in this marketplace

plugins/                     # Plugin directory (one folder per plugin)
└── lovable/                 # Lovable.dev integration plugin
    ├── plugin.json          # Plugin definition (version, description)
    ├── commands/            # Slash commands (/lovable:* commands)
    │   ├── init-lovable.md  # Initialize project context
    │   ├── map-codebase.md  # Generate Project Structure Map (NEW in v1.7.0)
    │   ├── deploy-edge.md   # Deploy edge functions
    │   ├── apply-migration.md # Apply database migrations
    │   ├── sync-lovable.md  # Sync with Lovable Cloud
    │   ├── yolo.md          # Toggle automation mode
    │   ├── test-init.md     # Preview testing wizard (NEW in v1.9.0)
    │   ├── test-run.md      # Run test plans in Preview (NEW in v1.9.0)
    │   ├── test-sync.md     # Resync test coverage (NEW in v1.9.0)
    │   └── [others].md
    ├── hooks/               # Claude Code hooks
    │   ├── hooks.json       # Hook configuration (Start and Stop events)
    │   ├── auto-sync.sh     # Auto-sync (Start event)
    │   └── auto-push.sh     # Auto-push (Stop event)
    ├── skills/              # Contextual skills
    │   ├── lovable/
    │   │   ├── SKILL.md     # Core Lovable integration patterns
    │   │   └── references/  # Supporting documentation
    │   │       ├── CLAUDE-template.md  # Template for generated CLAUDE.md
    │   │       ├── codebase-map.md     # Project Structure Map patterns (NEW)
    │   │       ├── prompts.md          # Lovable prompt library
    │   │       └── secret-detection.md # Secret scanning patterns
    │   ├── yolo/
    │   │   ├── SKILL.md     # Browser automation orchestration
    │   │   └── references/  # Automation workflows
    │   └── testing/         # Preview testing (NEW in v1.9.0)
    │       ├── SKILL.md     # Preview testing orchestration
    │       └── references/  # preview-access, test-plan-format,
    │                        # test-wizard, test-execution
    └── agents/              # Autonomous agents
        └── sync-agent.md    # Multi-phase sync agent

.claude/                     # Repository-level settings
└── settings.local.json      # Pre-approved git operations

CLAUDE.md                    # This file (repo documentation)
README.md                    # User-facing documentation
CHANGELOG.md                 # Version history
```

**Adding a new plugin**: Create a new folder under `plugins/` with its own `plugin.json` and add it to `marketplace.json`.

### Key Architectural Concepts

#### 1. Two-Tier Documentation System

**Commands** (`/lovable:*`) contain:
- User-facing instructions for Claude to execute
- Step-by-step procedural workflows
- User interaction patterns (questions, confirmations)
- CLAUDE.md generation logic

**Skills** (auto-activate) contain:
- Conceptual knowledge about Lovable integration
- When to use what approach
- Reference materials for complex operations
- Browser automation workflows

**Reference files** (`skills/*/references/`) contain:
- Detailed implementation procedures
- Edge case handling
- UI automation selectors and wait conditions
- Testing procedures

#### 2. CLAUDE.md Generation Pattern

This plugin's core feature is **generating CLAUDE.md files** for user projects (not this repo). The flow:

1. User runs `/lovable:init` in their Lovable project
2. Plugin scans their codebase (edge functions, migrations, secrets)
3. Asks 12-15 questions about their setup
4. Generates `CLAUDE.md` in their project root using `plugins/lovable/skills/lovable/references/CLAUDE-template.md`
5. This CLAUDE.md gives future Claude instances context about their Lovable project

**Critical distinction**: This repo contains the plugin code. User projects get a generated CLAUDE.md.

#### 3. Auto-Sync, Auto-Push, and Yolo Mode Features

**Auto-Sync** (NEW in v1.5.0):
- **Hook-based implementation** - uses Claude Code's Start event hook for 100% reliability
- Automatically pulls latest changes from GitHub when Claude starts working
- **Always-on feature** - runs automatically on every Start event (no configuration needed)
- Implemented in `plugins/lovable/hooks/auto-sync.sh` (triggered by `plugins/lovable/hooks/hooks.json`)
- Safety checks: only on main branch, no uncommitted changes, local is behind remote
- Uses `git pull --rebase` to maintain clean history

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [10K-Digital/lovable-claude-code](https://github.com/10K-Digital/lovable-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
