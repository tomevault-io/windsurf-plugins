---
trigger: always_on
description: > **Hermes → Hermes equivalents** (all paths and commands below work on both platforms)
---

# SKILL.md - October Swarm Architecture

## PLATFORM_MAPPING

> **Hermes → Hermes equivalents** (all paths and commands below work on both platforms)

| Hermes Path / Command | Hermes Equivalent |
|-------------------------|-------------------|
| `~/.openclaw/workspace` | `~/.hermes/skills/autonomous-ai-agents/hermes-agent` |
| `~/.hermes/projects/october-swarm-skills/agents/octoberxin` | `~/.hermes/projects/october-swarm-skills/agents/octoberxin` |
| `~/.hermes/projects/october-swarm-skills/agents/halloween` | `~/.hermes/projects/october-swarm-skills/agents/halloween` |
| `~/.hermes/projects/october-swarm-skills/agents/octavia` | `~/.hermes/projects/october-swarm-skills/agents/octavia` |
| `~/.hermes/projects/october-swarm-skills/agents/octane` | `~/.hermes/projects/october-swarm-skills/agents/octane` |
| `~/.hermes/projects/october-swarm-skills/agents/octopus` | `~/.hermes/projects/october-swarm-skills/agents/octopus` |
| `~/.hermes/projects/october-swarm-skills/agents/bee` | `~/.hermes/projects/october-swarm-skills/agents/bee` |
| `~/.hermes/vault/credentials.json` | `~/.hermes/vault/credentials.json` |
| `~/.hermes/memory/daily/YYYY-MM-DD.md` | `~/.hermes/memory/daily/YYYY-MM-DD.md` |
| `~/.hermes/memory/user/MEMORY.md` | `~/.hermes/memory/user/MEMORY.md` |
| `~/.hermes/logs/agent-comm/` | `~/.hermes/logs/agent-comm/` |
| `~/obsidian-vault/` | `~/obsidian-vault/` |
| `hermes chat` | `hermes chat` |
| `openclaw october spawn <agent>` | `delegate_task` with context + skill loading |
| `openclaw october task init` | `/todo` or `todo` tool |
| `openclaw october workflow <stage>` | `delegate_task` with goal + toolsets |
| `cronjob or delegate_task for background tasks` | `cronjob` or `delegate_task` for background tasks |
| `delegate_task with classification goal` | Manual task classification (or `delegate_task` with classification goal) |
| `openclaw octorch route` | `delegate_task` with per-task model/provider |
| `openclaw octorch workflow start` | `delegate_task` batch with workflow stages |
| `openclaw octorch critique` | `delegate_task` with critique skill + adversarial-stress-test |
| `openclaw octorch topology` | `skills_list` + `skill_view` |
| `openclaw sprint start` | `todo` tool with sprint items |
| `openclaw bee task submit` | `cronjob` create for background tasks |
| `openclaw skill <name>` | `skill_view(name)` + follow skill instructions |

---
**October Swarm** — A multi-agent orchestration framework with tiered architecture, 4-stage workflows, and bee swarm administrative capabilities.

## Overview

October is a production-ready multi-agent orchestration framework designed for complex, autonomous task execution. It implements a tiered model (T1-T4) with specialized agents that collaborate through defined workflows.

**Key Features:**
- Tiered collaboration (T1-T4 models based on task complexity)
- 4-stage workflow: Planning → Review → QA → Ship
- Bee swarm administrative pool for stateless workers
- Agent topology and handoff protocols
- Sprint integration for time-boxed work cycles

**Version:** 1.0.0
**Status:** Alpha
**Channel:** Hermes / OpenClaw

---

## Installation

### Prerequisites

```bash
# Python 3.10+ or Node.js 18+
# ollama or LLM service configured
# Hermes installed and running
```

### Install the October Swarm

```bash
# Install via ClawHub
clawhub install october-swarm-skills

# Or clone manually
git clone <repository-url> ~/.hermes/skills/autonomous-ai-agents/hermes-agent
cd ~/.hermes/skills/autonomous-ai-agents/hermes-agent
```

### Install Individual Skills

```bash
clawhub install october-swarm-skills/swarm-orchestration
clawhub install october-swarm-skills/4-stage-workflow
clawhub install october-swarm-skills/bee-tasks
clawhub install october-swarm-skills/octoberxin-critique
clawhub install october-swarm-skills/agent-topology
clawhub install october-swarm-skills/sprint-cycles
```

### Setup Agent Workspaces

```bash
# Create agent directories
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/halloween
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octoberxin
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octavia
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octane
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/octopus
mkdir -p ~/.hermes/projects/october-swarm-skills/agents/bee/{pending,completed}

# Copy agent profiles
cp -r october-swarm-skills/agents/* ~/.hermes/projects/october-swarm-skills/agents/bee/
```

### Configure Credentials

```bash
cp examples/credentials.example ~/.hermes/vault/credentials.json
# Edit with actual credentials (Moltbook, Notion, X/GitHub, etc.)
```

---

## Quick Start

### Initialize October

```bash
# Start the orchestrator
hermes chat

# Check swarm status
hermes status

# Spawn an agent
delegate_task with context + skill loading for halloween
```

### Run a 4-Stage Workflow

```bash
# Create a new task
todo add "feature-auth"

# Execute workflow stages
delegate_task with goal + toolsets for planning  # Halloween
delegate_task with goal + toolsets for review    # Octavia
delegate_task with goal + toolsets for qa        # Octane
delegate_task with goal + toolsets for ship      # Octopus

# Complete the workflow
todo complete feature-auth
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0x-wzw/october-swarm-skills](https://github.com/0x-wzw/october-swarm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
