---
trigger: always_on
description: This file provides critical guidance to Claude Code (claude.ai/code) when working with the KIRA project.
---

# CLAUDE.md - Developer Guide for Claude Code

This file provides critical guidance to Claude Code (claude.ai/code) when working with the KIRA project.

## 🎯 Core Concept: "Install App = Hire AI Coworker"

**CRITICAL UNDERSTANDING**: This project's revolutionary concept is that installing the desktop app instantly transforms any computer into an AI coworker. No servers, no cloud setup, no technical expertise required - just like installing Microsoft Office.

### The Vision
- **End Users**: Non-technical people who need an AI assistant
- **Installation**: As simple as installing any desktop application
- **Configuration**: GUI-based, no terminal or coding required
- **Result**: A fully functional AI coworker working 24/7

### Development Philosophy
When developing ANY feature, always ask:
1. Can a non-developer understand and use this?
2. Does it maintain the "zero configuration" principle?
3. Is the GUI self-explanatory?
4. Will it work immediately after installation?

---

## 📁 Project Structure

```
kira/                           # Root directory
│
├── electron-app/               # 🎯 THE CORE - Desktop app
│   ├── main.js                # Electron main process - manages Python server lifecycle
│   ├── preload.js             # Secure IPC bridge
│   ├── renderer/              # User interface
│   │   ├── index.html         # Main UI structure (Environment Variables Settings UI)
│   │   ├── main.css           # Dark theme styling
│   │   └── main.js            # UI logic, server control, config management
│   ├── package.json           # Node dependencies and build config
│   └── dist/                  # Built installers (.dmg, .exe, .AppImage)
│
├── app/                       # Python AI server (runs invisibly in background)
│   ├── main.py               # Server entry point, worker/scheduler setup
│   ├── cc_agents/            # AI agent modules
│   │   ├── bot_call_detector/        # Bot call detection (Haiku)
│   │   ├── simple_chat/              # Simple conversation (Haiku)
│   │   ├── operator/                 # Complex task execution (Sonnet)
│   │   ├── memory_retriever/         # Memory search (Haiku)
│   │   ├── memory_manager/           # Memory storage (Haiku)
│   │   ├── answer_aggregator/        # Answer collection (Haiku)
│   │   ├── proactive_suggester/      # Proactive suggestions (Sonnet)
│   │   └── proactive_confirm/        # Suggestion approval request (Haiku)
│   ├── cc_checkers/          # Proactive Receiver Channels (Proactive monitors)
│   │   ├── outlook/          # Outlook email checker
│   │   └── atlassian/        # Confluence/Jira checker (Rovo MCP)
│   ├── cc_tools/             # MCP tool implementations
│   │   ├── slack/            # Slack tools (11)
│   │   ├── outlook/          # Outlook tools (7)
│   │   ├── scheduler/        # Message scheduling
│   │   ├── waiting_answer/   # Answer waiting management
│   │   ├── confirm/          # User approval management
│   │   ├── email_tasks/      # Email tasks DB
│   │   ├── jira_tasks/       # Jira tasks DB
│   │   └── x/                # X (Twitter) tools
│   ├── cc_web_interface/     # Web Server / Voice Input Channel
│   │   ├── server.py         # FastAPI server (port 8000, HTTPS)
│   │   ├── auth_handler.py   # Auth router
│   │   ├── auth_azure.py     # MS365 OAuth
│   │   └── auth_slack.py     # Slack OAuth
│   ├── cc_slack_handlers.py  # Slack event handlers
│   ├── queueing_extended.py  # 3-tier queue system
│   ├── scheduler.py          # APScheduler management
│   └── config/               # Configuration management
│       ├── settings.py       # Pydantic settings (environment variable definitions)
│       └── env/
│           ├── dev.env       # Development environment variables
│           └── credential.json  # GCP service account
│
└── docs/                     # User-facing documentation
```

---

## 🚨 Critical Development Rules

### 1. Electron App is Sacred
- **NEVER** break the Electron app - it's the user's only interface
- **ALWAYS** test GUI changes on multiple screen sizes
- **ENSURE** error messages are user-friendly (no stack traces!)
- **MAINTAIN** backward compatibility with saved configs

### 2. Zero-Setup Principle
- **NO** manual file editing should ever be required
- **NO** terminal commands for end users
- **ALL** configuration through GUI
- **DEFAULT** values for everything optional

### 3. Environment Variable Synchronization Required
**CRITICAL**: When adding/modifying environment variables, you **MUST update all 5 locations**:

1. **app/config/settings.py** - Pydantic model definition
2. **app/config/env/dev.env** - Development default values
3. **electron-app/renderer/index.html** - UI input fields
4. **electron-app/renderer/main.js** - `fields` array (list of fields to save/load)
5. **electron-app/main.js** - config.env save section (`sections` object)

**Be especially careful not to forget the `fields` array in `renderer/main.js`!**
Fields not in this array are ignored by the save/load logic.

The order must also match:
```
1. Slack Integration
2. Bot Information
3. MCP Settings (Perplexity, DeepL, GitLab, Atlassian, Outlook, X, Clova)
4. Computer Use
5. Web Server / Voice Input Channel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krafton-ai/KIRA](https://github.com/krafton-ai/KIRA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
