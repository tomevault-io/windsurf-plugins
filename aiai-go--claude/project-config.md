---
trigger: always_on
description: - **Product**: @aiai-go/claude — Multilingual AI Coding CLI
---

# @aiai-go/claude — Project Handbook

## Project Identity
- **Product**: @aiai-go/claude — Multilingual AI Coding CLI
- **npm**: `@aiai-go/claude` (org: `@aiai-go`, account: `aiaigo`)
- **GitHub**: https://github.com/aiai-go/claude
- **Website**: https://aiaigo.org
- **Telegram**: https://t.me/aiai_go
- **Email**: hi@aiaigo.org
- **License**: MIT
- **Author**: Claudechinese (GitHub: whaleaicode)
- **Git author**: `Claudechinese <239421346+whaleaicode@users.noreply.github.com>`

## Brand Rules
- Product name: `@aiai-go/claude` (NOT claudezh, except as CLI command name)
- Ecosystem: `aiai-go` (claude/gemini/gpt/codex)
- CLI commands: `aiai-claude` (primary), `claudezh` (alias)
- Config directory: `~/.claudezh/`
- Python package: `aicodezh` (internal, don't rename)

## Tech Stack
- **Python 3.10+**: Core logic (aicodezh/ package)
- **Node.js 16+**: npm wrapper (bin/claudezh-wrapper.js)
- **claude-agent-sdk**: SDK mode (subscription, free)
- **anthropic SDK**: API mode (standalone, paid)
- **Rich**: Terminal UI
- **No database, no external services** (all local files)

## Architecture

```
CLI Layer (cli.py)
├── Commands (20+ slash commands, Chinese + English aliases)
├── Banner (print_banner, modern Rich UI)
├── REPL loop (input → dispatch → chat)
└── i18n (i18n.py, 290+ keys, zh-CN/zh-TW/en)

Backend Layer (backend.py)
├── SDKBackend (claude-agent-sdk → Claude CLI process)
│   ├── Persistent connection (ClaudeSDKClient)
│   ├── Built-in tools: Read, Edit, Write, Bash, Glob, Grep
│   └── get_account_info() via get_server_info()
├── APIBackend (anthropic SDK → direct API)
│   └── Custom tools from tools.py
└── detect_backend() auto-selects best available

Feature Modules:
├── skills.py — 10 AI personas, 4 categories
├── hooks.py — 13 safety rules, audit logging
├── checkpoint.py — file undo/rollback
├── mcp_tools.py — 12 custom MCP tools
├── quota.py — real Claude usage API + local tracking
├── agent.py — ChineseAgent wrapper, preset agents
├── tools.py — 8 file/code tools (API mode)
├── templates.py — 7 preset prompts
├── history.py — conversation persistence
└── config.py — ~/.claudezh/config.json

Plugin Layer (plugin/)
├── DXT manifest (manifest.json)
├── MCP server (server.py, 12 tools standalone)
└── Commands (7 Chinese slash commands for Claude Code)
```

## File Map
```
aicodezh/
├── __init__.py      — Package exports
├── __main__.py      — python -m aicodezh entry
├── cli.py           — Main CLI (REPL, commands, banner) ~1200 lines
├── backend.py       — Dual backend abstraction ~800 lines
├── agent.py         — SDK wrapper, ChineseAgent class
├── skills.py        — 10 skill personas with system prompts
├── hooks.py         — Safety hooks (PreToolUse, PostToolUse, etc.)
├── checkpoint.py    — File checkpointing for undo
├── mcp_tools.py     — 12 custom MCP tools
├── quota.py         — Usage tracking (real API + local)
├── tools.py         — File/code tools for API mode
├── templates.py     — 7 preset prompt templates
├── history.py       — Conversation history management
├── config.py        — Config load/save (~/.claudezh/)
├── i18n.py          — 290+ translation keys (zh-CN/zh-TW/en)
└── version.py       — VERSION, APP_NAME, BRAND_NAME

bin/
├── claudezh           — Shell entry point
└── claudezh-wrapper.js — Node.js wrapper for npm

scripts/
└── postinstall.js     — npm postinstall (pip dependencies)

.github/
├── FUNDING.yml
├── PULL_REQUEST_TEMPLATE.md
└── ISSUE_TEMPLATE/
    ├── bug_report.md
    └── feature_request.md

plugin/
├── manifest.json      — DXT plugin manifest
├── server.py          — Standalone MCP server (12 tools)
├── README.md          — Plugin documentation
└── commands/
    ├── zh.md          — /zh 简体中文
    ├── zht.md         — /zht 繁体中文
    ├── en.md          — /en English
    ├── review-zh.md   — /review-zh 中文审查
    ├── explain-zh.md  — /explain-zh 中文解释
    ├── test-zh.md     — /test-zh 中文测试
    └── fix-zh.md      — /fix-zh 中文修复

assets/
└── demo.svg           — Terminal screenshot for README

docs/
└── COMMUNITY_GUIDE.md — Community management handbook
```

## Development Commands
```bash
# Install for development
pip install -e . --break-system-packages

# Run
env CLAUDECODE= claudezh

# Test imports
python3 -c "from aicodezh.cli import run; print('OK')"

# Check all modules
python3 -c "
from aicodezh import VERSION
from aicodezh.skills import SKILLS
from aicodezh.tools import list_tools
from aicodezh.hooks import DANGEROUS_PATTERNS
print(f'v{VERSION} | {len(SKILLS)} skills | {len(list_tools())} tools')
"

# npm pack test
npm pack --dry-run
```

## Git Rules
- Author: `Claudechinese <239421346+whaleaicode@users.noreply.github.com>`
- Always use this author for commits
- Keep commits clean and descriptive
- Squash before release (single commit per version)

## i18n Rules
- Every user-facing string MUST go through `t()` function
- Every key needs 3 locales: zh-CN, zh-TW, en
- Chinese skill/template system prompts are intentionally Chinese (domain expertise)
- CLI command aliases (e.g., /帮助) stay as-is
- Default locale: `en` for non-Chinese systems

## Safety Rules
- NEVER modify ~/.claude/ directory
- NEVER delete user files without confirmation
- NEVER store user credentials in our files
- Credentials are READ-ONLY

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiai-go/claude](https://github.com/aiai-go/claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
