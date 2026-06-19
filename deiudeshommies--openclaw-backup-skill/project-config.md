---
trigger: always_on
description: Universal OpenClaw configuration backup and restore tool with automatic detection. Intelligently discovers and backs up all OpenClaw components regardless of installation location, agent configuration, or enabled integrations. Works across different OpenClaw versions, platforms (macOS/Linux/Windows), and deployment types. Use when backing up any OpenClaw instance, migrating between machines, creating disaster recovery snapshots, or preserving configuration state. Trigger on "backup", "backup ope
---


# Universal OpenClaw Configuration Backup

## Overview
OpenClaw is highly customizable, which means every instance is unique. This skill automatically detects your specific configuration and creates a complete backup without requiring manual configuration. Whether you have 3 agents or 30, use Telegram or Slack, run on macOS or Linux, this skill adapts to your setup.

**Key Features:**
- 🔍 **Auto-detection** - Discovers all components automatically
- 🌍 **Cross-platform** - Works on macOS, Linux, Windows
- 🎯 **Version-agnostic** - Compatible with all OpenClaw versions
- 🔒 **Safe** - Never assumes files exist, handles missing components gracefully
- 📦 **Complete** - Backs up everything that matters

---

## How It Works: Discovery-Based Backup

Unlike traditional backup tools that assume a fixed structure, this skill:

1. **Discovers** your OpenClaw installation location
2. **Scans** for all configuration files and directories
3. **Detects** enabled integrations and agents
4. **Backs up** only what exists in your instance
5. **Documents** what was backed up in a manifest

**No hardcoded paths. No assumptions. Just intelligent detection.**

---

## What Gets Backed Up (Auto-Detected)

### Core Configuration (if present)
- **openclaw.json** - Main configuration file
- **.env** - Environment variables and API keys
- **BOOT.md** - Startup configuration (if exists)
- **RECOVERY.md** - Recovery instructions (if exists)
- **config.yaml** / **config.toml** - Alternative config formats
- All backup files matching patterns: `*.bak`, `*.backup-*`, `*.old`

### Agent Configurations (all discovered agents)
The skill automatically scans your agents directory and backs up:
- All agent subdirectories (regardless of names)
- Agent-specific configuration files
- Agent prompts and instructions
- Model caches (if present)
- Session histories (optional, can be excluded)

**Example:** If you have agents named `coder`, `writer`, `analyst`, all will be backed up automatically.

### Authentication & Credentials (if configured)
Detects and backs up credentials for:
- Messaging platforms (Telegram, Slack, Discord, Feishu, etc.)
- OAuth tokens and API keys
- Bot configurations and pairing info
- Channel authorization settings
- Any files in `credentials/` or `auth/` directories

### Data & Knowledge (if present)
- **cron/** or **scheduler/** - Scheduled jobs and execution history
- **shared-learnings/** or **knowledge/** - Knowledge base
- **memory/** - Persistent memory and knowledge graphs
- **subagents/** - Subagent execution records
- **workspace/** - Workspace configurations and state
- **logs/** - Recent logs (optional)

### External Integrations (auto-detected)
If you have Claude, Cursor, or other IDE integrations:
- Searches common config locations (`~/.claude/`, `~/.cursor/`, etc.)
- Backs up integration configs if found
- Skips if not present

---

## Step 1: Quick Backup (Default Mode)

The fastest way to create a complete backup.

**Command:**
```
"backup my openclaw config"
"create a backup of my setup"
"backup everything"
```

**What happens:**
1. **Detects** OpenClaw installation directory
2. **Scans** for all configuration files
3. **Discovers** all agents (regardless of names)
4. **Identifies** enabled integrations
5. **Creates** timestamped backup directory
6. **Copies** all discovered components
7. **Generates** backup manifest and restore guide
8. **Verifies** backup integrity

**Output structure (example):**
```
<backup-location>/openclaw-backup-YYYYMMDD-HHMMSS/
├── openclaw-config/          # Core configs (detected)
│   ├── openclaw.json
│   ├── .env
│   ├── BOOT.md              # (if exists)
│   └── *.bak                # (if exists)
├── openclaw-data/            # All data (detected)
│   ├── agents/              # All discovered agents
│   │   ├── <agent-1>/
│   │   ├── <agent-2>/
│   │   └── ...
│   ├── cron/                # (if exists)
│   ├── credentials/         # (if exists)
│   ├── shared-learnings/    # (if exists)
│   ├── memory/              # (if exists)
│   └── workspace/           # (if exists)
├── integrations/             # External integrations (if detected)
│   └── claude/              # (if ~/.claude/ exists)
├── backup-manifest.json      # What was backed up
└── README.md                 # Restore guide
```

**Backup location priority:**
1. User-specified location (if provided)
2. Desktop directory (if exists and writable)
3. User's home directory under `~/openclaw-backups/`
4. Current directory as fallback

**Time estimate:** 30-90 seconds depending on instance size

---

## Step 2: Selective Backup

Backup only specific components.

**Available components (auto-detected):**

| Component | Command Trigger | What's Included |
|---|---|---|
| **Config only** | "backup just config files" | Core configuration files only |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeiuDesHommies/openclaw-backup-skill](https://github.com/DeiuDesHommies/openclaw-backup-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
