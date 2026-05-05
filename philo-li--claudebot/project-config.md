---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**ClaudeBot** is an Electron tray application that provides two interfaces for controlling Claude Code CLI:

1. **Telegram Bot** — Send prompts to Claude Code via Telegram messages
2. **Dopamind Client** — HTTP polling client that receives prompts from the Dopamind API queue

The app runs as a system tray icon with start/stop controls and a settings window.

## Repository Structure

```
ClaudeBot/
├── main.cjs              # Electron main process (tray app, config window, IPC)
├── bot.js                # Telegram bot (ESM, polling mode)
├── claude-runner.js      # Shared module: spawn claude CLI, session management
├── dopamind-client.cjs   # Dopamind HTTP polling client (CJS)
├── config.html           # Settings UI (Electron BrowserWindow)
├── preload-config.cjs    # Electron preload script for config window
├── sessions.json         # Session persistence (auto-generated)
├── assets/               # Tray icons (tray-running.png, tray-stopped.png, icon.ico)
├── electron-builder.yml  # Electron Builder config
├── package.json          # Dependencies and scripts
└── dist/                 # Build output
```

## Quick Start Commands

```bash
# Run Telegram bot directly (without Electron)
npm start

# Run as Electron tray app (development)
npm run electron:dev

# Build installers
npm run build:win      # Windows (NSIS + portable)
npm run build:mac      # macOS (DMG)
npm run build:linux    # Linux (AppImage + deb)
```

## Technology Stack

| Component       | Technology                                                           |
| --------------- | -------------------------------------------------------------------- |
| Desktop Shell   | Electron                                                             |
| Telegram API    | node-telegram-bot-api                                                |
| Module System   | ESM (bot.js, claude-runner.js) + CJS (main.cjs, dopamind-client.cjs) |
| CLI Integration | Claude Code CLI via child_process.spawn                              |
| Build           | electron-builder                                                     |

## Architecture Notes

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Electron Shell                           │
│                         (main.cjs)                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ System Tray │  │ Config IPC  │  │ Bot Lifecycle Manager   │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌──────────────────────────┐    ┌──────────────────────────┐
│      Telegram Bot        │    │    Dopamind Client       │
│        (bot.js)          │    │  (dopamind-client.cjs)   │
│                          │    │                          │
│  • /ask, /run, /new      │    │  • HTTP polling (3s)     │
│  • /stop, /status        │    │  • Progress reporting    │
│  • /dir, /setdir         │    │  • Result posting        │
└──────────────────────────┘    └──────────────────────────┘
              │                               │
              └───────────────┬───────────────┘
                              ▼
              ┌──────────────────────────────┐
              │      Claude Runner           │
              │    (claude-runner.js)        │
              │                              │
              │  • Spawn claude -p           │
              │  • Stream JSON parsing       │
              │  • Session management        │
              │  • Progress callbacks        │
              └──────────────────────────────┘
                              │
                              ▼
              ┌──────────────────────────────┐
              │       Claude Code CLI        │
              │  (child_process.spawn)       │
              │                              │
              │  --output-format stream-json │
              │  --resume (session ID)       │
              └──────────────────────────────┘
```

### Module Descriptions

- **main.cjs** — Electron main process. Creates system tray, manages bot lifecycle, settings window via IPC
- **bot.js** — Telegram bot. Registers command handlers (/ask, /run, /new, /stop, /dir, /setdir, /status). Plain messages also sent to Claude
- **claude-runner.js** — Core module. Spawns `claude -p` with `--output-format stream-json`, manages sessions (resume via session ID), parses streaming progress
- **dopamind-client.cjs** — Polls Dopamind API `/api/desktop-queue/poll` every 3s, processes messages via claude-runner, posts progress and results back
- **Config** is stored in `%APPDATA%/ClaudeBot/.env` (Electron userData path), not in the repo

### Data Flow

1. **Input**: User sends message via Telegram or Dopamind queues a prompt
2. **Processing**: Bot/Client receives message → calls `claude-runner.js`
3. **Execution**: Runner spawns Claude CLI with streaming output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Philo-Li/claudebot](https://github.com/Philo-Li/claudebot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
