---
trigger: always_on
description: PTY-based client-server system for managing AI CLI sessions (Claude Code, OpenAI Codex, Cursor Agent, Gemini CLI) with message queueing, image support, and native IDE scrolling.
---

# Leap

PTY-based client-server system for managing AI CLI sessions (Claude Code, OpenAI Codex, Cursor Agent, Gemini CLI) with message queueing, image support, and native IDE scrolling.

## Quick Start

```bash
make install                # Install core
make install-monitor        # Install GUI (optional)
source ~/.zshrc             # Reload shell

leap mytag                       # Terminal 1: Select CLI + start server
leap mytag                       # Terminal 2: Connect client
leap                             # Interactive: choose CLI + session name
```

## Project Structure

```
src/
├── scripts/                     # Entry point scripts
│   ├── leap-main.sh          # Main launcher (called by 'leap' command)
│   ├── leap-resume.py        # `leap --resume` picker — CLI-agnostic; each row is shown as `[cli] tag` and resumes via the provider's `resume_args(id)`
│   ├── leap-hook-process.py  # Hook processor invoked by `leap-hook.sh`; shared across all CLIs. Handles stdin parsing, session recording via `provider.extract_session_id()`, and last-assistant-message extraction for Slack
│   ├── leap-cleanup.sh       # Dead session cleanup
│   ├── _leap                 # zsh completion for user-facing flags (conditional --slack)
│   ├── leap-server.py        # Thin launcher → LeapServer
│   ├── leap-client.py        # Thin launcher → LeapClient
│   ├── leap-monitor.py       # Thin launcher → MonitorWindow
│   ├── leap-slack.py         # Thin launcher → SlackBot
│   ├── leap_monitor_launcher.py  # py2app entry point
│   ├── setup-slack-app.sh       # Interactive Slack app setup wizard
│   ├── configure_jetbrains_xml.py   # JetBrains IDE auto-configuration
│   ├── configure_hooks.py           # Unified hook config (delegates to provider.configure_hooks())
│   ├── configure_claude_hooks.py    # Legacy Claude hook config (kept for reference)
│   ├── configure_codex_hooks.py     # Legacy Codex hook config (kept for reference)
│   └── leap-hook.sh             # CLI hook script (writes state to signal file)
│
└── leap/                     # Main Python package
    ├── __init__.py              # Version, exports
    ├── main.py                  # Package entry point
    │
    ├── cli_providers/           # CLI backend abstraction (Strategy pattern)
    │   ├── __init__.py          # Package exports, get_provider(), list_providers()
    │   ├── base.py              # CLIProvider ABC (patterns, timings, hooks, input)
    │   ├── claude.py            # Claude Code provider (Ink TUI, numbered menus)
    │   ├── codex.py             # OpenAI Codex provider (Ratatui TUI, y/n approval)
    │   ├── cursor_agent.py     # Cursor Agent provider (Ink TUI, menu approval)
    │   ├── gemini.py            # Gemini CLI provider (Ink TUI, radio-button approval)
    │   ├── registry.py          # Provider registry (name → class lookup)
    │   └── states.py            # CLIState enum + state groupings (WAITING/SIGNAL/PROMPT)
    │
    ├── utils/                   # Shared utilities
    │   ├── constants.py         # QUEUE_DIR, SOCKET_DIR, timing, colors, is_valid_tag()
    │   ├── terminal.py          # Terminal title, banner
    │   ├── ide_detection.py     # IDE detection, git branch
    │   ├── socket_utils.py      # Shared Unix socket send/recv helper
    │   └── resume_store.py      # Shared read/write/prune of `cli_sessions/<cli>/<tag>.json` (used by hook + picker)
    │
    ├── server/                  # PTY Server
    │   ├── server.py            # LeapServer - main orchestrator
    │   ├── pty_handler.py       # CLI PTY (pexpect, provider-driven)
    │   ├── socket_handler.py    # Unix socket server
    │   ├── queue_manager.py     # Message queue persistence
    │   └── metadata.py          # Session metadata (IDE, project, branch, cli_provider)
    │
    ├── client/                  # Interactive Client
    │   ├── client.py            # LeapClient - main class
    │   ├── socket_client.py     # Unix socket client
    │   ├── input_handler.py     # Prompt toolkit / readline
    │   └── image_handler.py     # Clipboard image handling
    │
    ├── monitor/                 # GUI Monitor (PyQt5)
    │   ├── app.py               # MonitorWindow (core window + UI init + lifecycle)
    │   ├── server_launcher.py   # PR server clone/checkout/start flow
    │   ├── session_manager.py   # Session discovery + read_client_pid()
    │   ├── scm_polling.py       # SCM poller + background workers
    │   ├── leap_sender.py         # Socket sender for /leap commands + message bundles
    │   ├── navigation.py        # IDE terminal navigation
    │   ├── monitor_utils.py     # Utilities (icon finder, lock removal)
    │   ├── themes.py            # Visual theme definitions (9 built-in themes, manager API)
    │   ├── permissions.py       # macOS Accessibility + Notifications checks; live state via AXIsProcessTrusted and ncprefs.plist bit 25
    │   │
    │   ├── _mixins/             # MonitorWindow mixin classes
    │   │   ├── actions_menu_mixin.py  # Git menu (branch col) + Path menu (Open Terminal/IDE)
    │   │   ├── scm_config_mixin.py    # SCM provider init, setup dialogs, toggles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nevo24/leap](https://github.com/Nevo24/leap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
