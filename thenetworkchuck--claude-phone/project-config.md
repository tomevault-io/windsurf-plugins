---
trigger: always_on
description: Voice interface for Claude Code via SIP/3CX. Call your AI, and your AI can call you.
---

# Claude Phone

Voice interface for Claude Code via SIP/3CX. Call your AI, and your AI can call you.

## Project Overview

Claude Phone gives your Claude Code installation a phone number through 3CX PBX integration:
- **Inbound**: Call an extension and talk to Claude - run commands, check status, ask questions
- **Outbound**: Your server can call YOU with alerts, then have a conversation about what to do

## Tech Stack

| Component | Technology |
|-----------|------------|
| Language | Node.js (ES modules for CLI, CommonJS for voice-app) |
| SIP Server | drachtio-srf |
| Media Server | FreeSWITCH (via drachtio-fsmrf) |
| STT | OpenAI Whisper API |
| TTS | ElevenLabs API |
| AI Backend | Claude Code CLI (via HTTP wrapper) |
| PBX | 3CX (any SIP-compatible works) |
| Container | Docker Compose |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Phone Call                                                  │
│      │                                                       │
│      ↓ Call extension 9000                                  │
│  ┌─────────────┐                                            │
│  │     3CX     │  ← PBX routes the call                    │
│  └──────┬──────┘                                            │
│         │ SIP                                               │
│         ↓                                                    │
│  ┌─────────────────────────────────────────────────┐       │
│  │           voice-app (Docker)                     │       │
│  │  ┌─────────────────────────────────────────┐   │       │
│  │  │ drachtio  │  FreeSWITCH  │  Node.js     │   │       │
│  │  │ (SIP)     │  (Media)     │  (Logic)     │   │       │
│  │  └─────────────────────────────────────────┘   │       │
│  └────────────────────┬────────────────────────────┘       │
│                       │ HTTP                                │
│                       ↓                                      │
│  ┌─────────────────────────────────────────────────┐       │
│  │   claude-api-server                              │       │
│  │   Wraps Claude Code CLI with session management │       │
│  └─────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
claude-phone/
├── CLAUDE.md                 # This file
├── CONSTITUTION.md           # DevFlow 2.0 development principles
├── README.md                 # User-facing documentation
├── install.sh                # One-command installer
├── package.json              # Root package (hooks, linting, tests)
├── eslint.config.js          # ESLint configuration
├── docker-compose.yml        # Multi-container orchestration
├── .env.example              # Environment template
│
├── .claude/commands/         # DevFlow slash commands
│   ├── feature.md            # /feature spec|start|ship
│   ├── test.md               # /test
│   ├── fix.md                # /fix [N]
│   ├── issues.md             # /issues
│   ├── investigate.md        # /investigate
│   ├── project.md            # /project
│   ├── batch.md              # /batch
│   └── design.md             # /design
│
├── cli/                      # Unified CLI tool
│   ├── package.json
│   ├── README.md
│   ├── bin/
│   │   ├── claude-phone.js   # CLI entry point
│   │   └── cli-main.js       # Command definitions
│   ├── lib/
│   │   ├── commands/         # Command implementations
│   │   │   ├── setup.js      # Interactive setup wizard
│   │   │   ├── start.js      # Start services
│   │   │   ├── stop.js       # Stop services
│   │   │   ├── status.js     # Service status
│   │   │   ├── doctor.js     # Health checks
│   │   │   ├── api-server.js # Start API server standalone
│   │   │   ├── logs.js       # Tail service logs
│   │   │   ├── backup.js     # Create backups
│   │   │   ├── restore.js    # Restore backups
│   │   │   ├── update.js     # Self-update
│   │   │   ├── uninstall.js  # Clean removal
│   │   │   ├── config/       # Config subcommands
│   │   │   │   ├── show.js
│   │   │   │   ├── path.js
│   │   │   │   └── reset.js
│   │   │   └── device/       # Device subcommands
│   │   │       ├── add.js
│   │   │       ├── list.js
│   │   │       └── remove.js
│   │   ├── config.js         # Config read/write
│   │   ├── docker.js         # Docker compose wrapper
│   │   ├── network.js        # Network utilities
│   │   ├── platform.js       # Platform detection
│   │   ├── port-check.js     # Port availability checks
│   │   ├── prereqs.js        # Prerequisite checks
│   │   ├── prerequisites.js  # Pi-specific prereqs
│   │   ├── process-manager.js# PID-based process management
│   │   ├── utils.js          # Shared utilities
│   │   └── validators.js     # API key validation
│   └── test/                 # Test suite
│
├── voice-app/                # Docker container for voice handling
│   ├── Dockerfile
│   ├── package.json
│   ├── index.js              # Main entry point
│   ├── config/
│   │   └── devices.json      # Device configurations
│   ├── lib/
│   │   ├── audio-fork.js     # WebSocket audio streaming
│   │   ├── claude-bridge.js  # HTTP client for Claude API
│   │   ├── connection-retry.js # Connection retry logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theNetworkChuck/claude-phone](https://github.com/theNetworkChuck/claude-phone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
