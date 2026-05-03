---
trigger: always_on
description: This file provides guidance for AI assistants (Codex and similar tools) working on this repository.
---

# AGENTS.md — Pocket Desk Agent

This file provides guidance for AI assistants (Codex and similar tools) working on this repository.

---

## Code Change Quality Standard

After **every** code change — no matter how small — you must:

1. **Re-read every file you touched**, end-to-end, in its final state.
2. **Run a gaps analysis**: check for ordering bugs, stale docstrings, broken execution paths, type mismatches, blocking calls in async contexts, magic strings, and incorrect output for each distinct code path.
3. **Fix every issue found** before reporting completion.
4. **Repeat steps 1–3** until a full re-read of all changed files produces zero issues.

Do not claim a task is complete after a single pass. Do not self-certify without evidence from the re-read. The loop ends only when you find nothing left to fix.

---

## Project Overview

**Pocket Desk Agent** is a Python Telegram bot that provides secure remote control of a Windows PC, powered by Google Gemini 2.0 Flash AI. It is distributed as a PyPI package (`pocket-desk-agent`) and runs as a local CLI daemon (`pdagent`).

Key capabilities: AI chat & agentic computer use (Gemini), file system browsing, desktop screenshots, keyboard/clipboard control, OCR-based UI automation, macro recording, Claude Desktop/VS Code integration, build automation (React Native APKs), and task scheduling.

**Platform target:** Windows (UI automation features). File system and AI features are cross-platform.

---

## Repository Layout

```
pocket-desk-agent/
├── pocket_desk_agent/          # Main Python package
│   ├── handlers/               # Bot command handlers (13 modules)
│   │   ├── _shared.py          # Singleton clients, safe_command decorator, global state
│   │   ├── auth.py             # /login, /authcode, /checkauth, /logout
│   │   ├── core.py             # /start, /help, /status, /new, /enhance, /sync, etc.
│   │   ├── filesystem.py       # /pwd, /cd, /ls, /cat, /find, /info
│   │   ├── system.py           # /screenshot, /hotkey, /clipboard, /battery, /shutdown, etc.
│   │   ├── automation.py       # /clicktext, /findtext, /smartclick, /findelements, etc.
│   │   ├── custom_commands.py  # /savecommand, /done, /listcommands, /deletecommand
│   │   ├── claude.py          # /claudeask, /clauderepo, /claudechat, /clauderemote, etc.
│   │   ├── antigravity.py      # /openantigravity, /antigravitychat, /claudecli, etc.
│   │   ├── build.py            # /build, /getapk
│   │   ├── scheduling.py       # /schedule, /claudeschedule, /listschedules, /cancelschedule
│   │   └── callbacks.py        # Inline keyboard button handlers
│   ├── cli.py                  # Entry point for `pdagent` CLI
│   ├── main.py                 # Application bootstrap, scheduler loop
│   ├── config.py               # Config class — reads from os.environ
│   ├── configure.py            # Interactive setup wizard + INI loader
│   ├── command_map.py          # Centralized list of (command, handler, description)
│   ├── command_registry.py     # User-defined macro storage
│   ├── file_manager.py         # Sandboxed file I/O (path traversal prevention)
│   ├── gemini_client.py        # Gemini API client with tool-calling
│   ├── antigravity_auth.py     # OAuth 2.0 PKCE implementation
│   ├── auth.py                 # User allowlist + multi-mode auth wrapper
│   ├── gemini_cli_auth.py      # Gemini CLI OAuth PKCE implementation
│   ├── scheduler_registry.py   # Persistent scheduled task storage
│   ├── startup_manager.py      # Windows logon-task startup management
│   ├── rate_limiter.py         # Token-bucket rate limiter
│   ├── updater.py              # Auto-update manager (git pull)
│   ├── automation_utils.py     # OCR/UI automation helpers
│   └── constants.py            # API endpoints and header constants
├── scripts/
│   ├── manage_auth.py          # Gemini authentication management script
│   └── manage_service.py       # Daemon lifecycle script
├── docs/                       # Feature documentation (markdown)
├── .github/workflows/
│   └── publish.yml             # PyPI publish on GitHub release
├── .env.example                # Config template
├── pyproject.toml              # PEP 621 metadata, dependencies, build config
├── requirements.txt            # Pinned dependency list
├── Makefile                    # Dev task automation
├── setup.sh / setup.bat        # Platform setup helpers
├── README.md
├── CONTRIBUTING.md
└── PROJECT_STRUCTURE.md
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Language | Python 3.11+ |
| Bot Framework | python-telegram-bot ≥ 21.0 (async) |
| AI | Google Gemini 2.0 Flash (via REST API) |
| Auth | Multi-mode auth: Antigravity OAuth PKCE, Gemini CLI OAuth PKCE, or API key |
| UI Automation | pywinauto, pyautogui, pygetwindow (Windows only) |
| Computer Vision | opencv-python, numpy (contour detection for /findelements) |
| OCR | pytesseract (Tesseract engine) |
| File Uploads | Dropbox SDK |
| Build Backend | hatchling (PEP 517) |
| Packaging | PyPI (`pocket-desk-agent`) |
| CI/CD | GitHub Actions, OIDC trusted publishing |

---

## Development Workflows

### Setup

```bash
git clone https://github.com/techgniouss/pocket-desk-agent.git
cd pocket-desk-agent
pip install -e ".[dev]"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techgniouss/pdagent](https://github.com/techgniouss/pdagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
