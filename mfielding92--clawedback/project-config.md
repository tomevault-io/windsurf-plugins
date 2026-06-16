---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

clawed-back is a personal AI assistant platform that runs inside Claude Code. It's a white-room reimplementation of the OpenClaw concept — a multi-channel AI assistant with tool execution, automation, and approval workflows — built entirely with Claude Code skills and minimal Python.

## Architecture

Claude Code IS the agent runtime. Python handles only persistent I/O (web server, message queue, voice transcription). Everything else is skills.

### Skill Compatibility
OpenClaw and Claude Code both use the **AgentSkills spec** (Markdown + YAML frontmatter in SKILL.md). OpenClaw/ClawHub skills can be imported via `/oc-hub import <slug>`. The importer strips OpenClaw-specific `metadata.openclaw` fields and adds Claude Code `allowed-tools`. The markdown body (actual instructions) works unchanged.

```
Web UI → FastAPI Server → SQLite Queue → Claude Code (polling) → Response → SSE → Web UI
```

### Hybrid Polling
- **Idle**: CronCreate checks queue every 1 minute
- **Active**: /loop checks every 10 seconds (escalates when user messages within 5 min)
- **De-escalation**: After 5 min idle, drops back to CronCreate

## Tech Stack

- **Python 3.11+** with FastAPI, uvicorn, sse-starlette
- **SQLite** for message queue (WAL mode)
- **Whisper** (turbo model) for voice transcription
- **Claude Code** skills for all intelligence and workflows

## Commands

```bash
# Start server (HTTP)
cd .claude/skills/oc-poll/scripts && source ../.venv/bin/activate && python main.py

# Start server (HTTPS — needs fullchain.pem + privkey.pem in server/)
cd .claude/skills/oc-poll/scripts && source ../.venv/bin/activate && python main.py --ssl

# Start server (HTTPS — custom cert paths)
cd .claude/skills/oc-poll/scripts && source ../.venv/bin/activate && python main.py --public /path/to/cert.pem --private /path/to/key.pem

# Or use the setup wizard
/oc-setup

# Token management
python .claude/skills/oc-poll/scripts/token_manager.py show          # Display current token
python .claude/skills/oc-poll/scripts/token_manager.py regenerate    # Generate new random token
python .claude/skills/oc-poll/scripts/token_manager.py set <token>   # Set custom token (restart server after)

# File management
python .claude/skills/oc-poll/scripts/file_manager.py store <path> --name file.pdf --type generated  # Store a file
python .claude/skills/oc-poll/scripts/file_manager.py get <file_id>       # Get file path by ID
python .claude/skills/oc-poll/scripts/file_manager.py share <file_id>     # Create temp download link (60 min default)
python .claude/skills/oc-poll/scripts/file_manager.py shares              # List active shares
python .claude/skills/oc-poll/scripts/file_manager.py cleanup             # Remove expired shares

# Queue operations (used by skills)
python .claude/skills/oc-poll/scripts/queue_manager.py peek          # Check for messages
python .claude/skills/oc-poll/scripts/queue_manager.py read          # Read next message
python .claude/skills/oc-poll/scripts/queue_manager.py write '<json>' # Send response
python .claude/skills/oc-poll/scripts/queue_manager.py ack <id>      # Mark processed
python .claude/skills/oc-poll/scripts/queue_manager.py history <n>   # Message history
python .claude/skills/oc-poll/scripts/queue_manager.py activity      # Last user activity timestamp
```

## Project Structure

```
data/                # Runtime data (gitignored)
  messages.db        # SQLite queue
  uploads/           # User-uploaded files
  sessions/          # Session state, poll state, automations
  logs/              # Server logs

.claude/skills/      # All skills
  oc-poll/           # Hybrid polling controller (heartbeat)
  oc-router/         # Message routing and dispatch
  oc-session/        # Conversation state management
  oc-respond/        # Response formatting and delivery
  oc-approve/        # Approval gate for dangerous operations
  oc-tools/          # Tool execution coordinator
  oc-voice/          # Voice message processing
  oc-automate/       # Scheduled task management
  oc-webhook/        # Webhook handler
  oc-channel/        # Channel adapter registry
  oc-hub/            # Skill marketplace
  oc-setup/          # First-run setup wizard
  (+ inherited toolkit skills)

.claude/agents/      # Subagents
```

## Available Skills

| Skill | Purpose |
|-------|---------|
| `/start` | Start ClawedBack — ensures server is up, creates polling cron, runs one poll. **Run this whenever restarting Claude Code.** |
| `/oc-update` | Pull latest ClawedBack updates from GitHub, preserving local config/data |
| `/oc-resume` | Resume after restart — starts server, polling, rebuilds config if missing (use if config is lost) |
| `/oc-setup` | First-run setup wizard |
| `/oc-automate` | Schedule tasks and reminders |
| `/oc-hub` | Browse and install skills |
| `/oc-files` | Store, organize, and share files via temporary download links |
| `/oc-token` | View, regenerate, or set a custom auth token |
| `/oc-ssl` | Set up Let's Encrypt SSL certificates via certbot |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mfielding92/ClawedBack](https://github.com/mfielding92/ClawedBack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
