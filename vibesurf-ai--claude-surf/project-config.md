---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude-Surf is a Claude Code plugin that integrates with VibeSurf, exposing browser automation, AI skills, workflows, and external app integrations through a **pure markdown skill system**.

**Key Dependency**: VibeSurf server must be running on `http://127.0.0.1:9335` for this plugin to function.

## Architecture

### Superpowers-Inspired Design

This plugin follows the **superpowers** architecture pattern:

- **No build system** - Pure markdown skills, no compilation
- **Hook-based injection** - SessionStart injects skill content
- **Frontmatter-driven** - `description` field defines triggers
- **Direct API access** - Claude calls VibeSurf HTTP API directly

### Project Structure

```
claude-surf/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata
│   └── marketplace.json     # Marketplace configuration
├── hooks/
│   ├── hooks.json           # Hook configuration
│   ├── session-start.sh     # Health check + skill injection
│   └── run-hook.cmd         # Cross-platform wrapper
├── skills/
│   ├── surf/                # Main entry skill (auto-injected)
│   ├── search/              # AI web search
│   ├── fetch/               # Fetch URL content as markdown
│   ├── js_code/             # Structured data extraction
│   ├── crawl/               # Page content extraction
│   ├── summary/             # Page summarization
│   ├── finance/             # Stock data
│   ├── trend/               # Trending news
│   ├── screenshot/          # Screenshots
│   ├── browser/             # Direct browser control (21 actions)
│   ├── browser-use/         # AI multi-step automation
│   ├── website-api/         # Platform APIs (XiaoHongShu/etc)
│   ├── workflows/           # Pre-built workflows
│   └── integrations/        # External apps (Gmail/GitHub/etc)
├── README.md
└── CLAUDE.md
```

## Skill System

### Auto-Injected Skill

**`surf`** - Main entry point, automatically injected via SessionStart hook
- Provides navigation to all other skills
- Contains VibeSurf API documentation
- No manual invocation needed

### On-Demand Skills

All other 13 skills are loaded via the **Skill tool** when needed:
- `search` - AI web search
- `fetch` - Fetch URL content as structured markdown
- `js_code` - Extract structured data with auto-generated JS
- `crawl` - Extract page content
- `summary` - Summarize webpages
- `finance` - Stock data from Yahoo Finance
- `trend` - Trending news
- `screenshot` - Page screenshots
- `browser` - Direct browser control
- `browser-use` - AI multi-step automation
- `website-api` - Social platform APIs
- `workflows` - Pre-built automations
- `integrations` - Gmail/GitHub/Slack/etc

### Skill Frontmatter

Each `SKILL.md` file has YAML frontmatter:

```yaml
---
name: skill-name
description: Use when [triggering conditions and symptoms]
---
```

**Critical**: The `description` field determines when Claude loads the skill. It should:
- Start with "Use when..."
- Describe triggering conditions, NOT workflow
- Include specific symptoms and examples

## VibeSurf API

VibeSurf exposes three core HTTP endpoints at `http://127.0.0.1:9335`:

### 1. List Available Actions
```bash
GET /api/tool/search?keyword={optional_keyword}
```
Returns all available VibeSurf actions.

### 2. Get Action Parameters
```bash
GET /api/tool/{action_name}/params
```
Returns JSON schema for the action's parameters.

### 3. Execute Action
```bash
POST /api/tool/execute
Content-Type: application/json

{
  "action_name": "action_name_here",
  "parameters": {
    // action-specific parameters
  }
}
```

### Standard Workflow

1. **Discover actions** → `GET /api/tool/search`
2. **Get parameters** → `GET /api/tool/{action_name}/params`
3. **Execute** → `POST /api/tool/execute`

## Plugin Configuration

### Claude Code Plugin Structure

**`.claude-plugin/plugin.json`** - Plugin metadata
```json
{
  "name": "surf",
  "description": "VibeSurf integration - Control browsers, automate workflows...",
  "version": "1.0.0"
}
```

**`.claude-plugin/marketplace.json`** - Marketplace configuration
```json
{
  "name": "claude-surf",
  "plugins": [{
    "name": "surf",
    "source": "./"
  }]
}
```

**`hooks/hooks.json`** - SessionStart hook configuration
```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|resume|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start.sh"
      }]
    }]
  }
}
```

### Health Check Hook

**`hooks/session-start.sh`** - Runs on session start:
1. Checks if VibeSurf is running (`curl http://127.0.0.1:9335/health`)
2. If running: Injects `surf` skill content to context
3. If not running: Displays warning with installation instructions

**Non-blocking**: Hook timeout is 3 seconds, won't prevent session start.

## Skill Reference

### AI Skills (8)

| Skill | Action | Purpose |
|-------|--------|---------|
| `search` | `skill_search` | AI-powered web search (Gemini) |
| `fetch` | `skill_fetch` | Fetch URL content as structured markdown |
| `js_code` | `skill_code` | Auto-generate JS to extract lists/tables |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibesurf-ai/claude-surf](https://github.com/vibesurf-ai/claude-surf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
