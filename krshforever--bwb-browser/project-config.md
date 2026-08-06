---
trigger: always_on
description: > **Author:** Krish Tiwari ([@krshforever](https://github.com/krshforever))
---

# bwb-browser — Agent Integration Guide

> **Author:** Krish Tiwari ([@krshforever](https://github.com/krshforever))
> **Package:** [`bwb-browser`](https://www.npmjs.com/package/bwb-browser) · 76KB source · 25 tools
> **Last updated:** 2026-07-28

## What is bwb?

**Browser Without Bloat** — a lightweight MCP server that gives any AI agent browser superpowers. 76KB. 25 tools. Zero heavy dependencies.

While other MCP browser tools ship a full browser binary (Playwright MCP = ~250MB, Puppeteer MCP = ~400MB), bwb uses **raw Chrome DevTools Protocol (CDP)** — the protocol Chrome speaks natively. It auto-detects the browser already on your system.

Built on Termux/Android. Runs everywhere. Weighs nothing. **Browser automation from your phone.**

---

## Compatible Agents

| Agent | Integration | Verified |
|-------|-----------|----------|
| **Claude Code** (Anthropic) | MCP config → `~/.claude/settings.json` | ✅ |
| **OpenCode** | MCP config → `~/.config/opencode/opencode.json` | ✅ |
| **Antigravity CLI** | MCP config → `~/.gemini/antigravity-cli/mcp_config.json` | ✅ |
| **Cline** (VS Code) | MCP config → VS Code settings or `~/.cline/mcp.json` | ✅ |
| **Continue.dev** | MCP config → `~/.continue/config.json` | ✅ |
| **Aider** | Custom tool integration | ✅ |
| **Codex CLI** | MCP config → `~/.codex/mcp.json` | ✅ |
| **Cody** (Sourcegraph) | MCP config | ✅ |
| **Windsurf** | MCP config | ✅ |
| **Cursor** | MCP config → `.cursor/mcp.json` | ✅ |
| **Any MCP-compatible agent** | MCP stdio transport | ✅ |

---

## Quick Install (60 seconds)

```bash
# 1. Install globally
npm install -g bwb-browser

# 2. Verify it works
bwb --version
# → bwb-browser 3.0.0

# 3. Add to your AI agent's MCP config
```

---

## 🔥 Copy-Paste Prompt for Your AI Agent

Give this to any AI agent (Claude, OpenCode, Gemini, etc.) to auto-install and configure bwb:

```
I want you to install and use bwb-browser — a 76KB MCP browser automation server with 25 tools.

bwb is Browser Without Bloat. It uses raw Chrome DevTools Protocol (no Playwright, no Puppeteer).
It has 25 tools including:

- `browser_act` — natural language interaction ("search for X", "click the button", "what's on this page")
- `browser_watch` — live capture of console messages, network requests, JS exceptions, page navigations
- `browser_diagnose` — full page health check (perf, errors, broken images, score)
- `browser_fingerprint` — realistic browser profile for testing
- Multi-tab management (create, close, switch, list)
- Session persistence (save/load cookies across restarts)

Created by Krish Tiwari (@krshforever) from a phone on a train in India. npm: bwb-browser

## Auto-Install Steps:

1. Check if bwb-browser is already installed:
   ```bash
   npm list -g bwb-browser 2>/dev/null || echo "not installed"
   ```

2. If not installed, install globally:
   ```bash
   npm install -g bwb-browser
   ```

3. Detect which MCP config file to update based on the current agent:
   - Claude Code → ~/.claude/settings.json
   - OpenCode → ~/.config/opencode/opencode.json
   - Antigravity → ~/.gemini/antigravity-cli/mcp_config.json
   - Cline → ~/.cline/mcp.json
   - Continue → ~/.continue/config.json
   - Cursor → .cursor/mcp.json (project root)

4. Add this MCP server entry (adjust for your config format):
   ```json
   "bwb": {
     "type": "local",
     "command": ["node", "/path/to/bwb/server.mjs"]
   }
   ```
   
   Or if installed globally:
   ```json
   "bwb": {
     "type": "local",
     "command": ["bwb"]
   }
   ```

5. Verify the config is valid JSON and the file has proper syntax.

6. Test the integration by calling `browser_status` tool — it should return `connected: true` 
   (it starts Chromium automatically on first call).

## Skill: I Know How to Use bwb

When using bwb for web tasks, follow these patterns:

### Basic Browsing
- `browser_goto({url: "https://example.com"})` — navigate to a page
- `browser_title()` — check page title
- `browser_screenshot()` — take a screenshot (also saves to /storage/emulated/0/Download/bwb-screenshots/ on Android or ~/bwb-screenshots/ on desktop)
- `browser_text()` — get page text content
- `browser_html()` — get page HTML
- `browser_elements({kind: "links"|"buttons"|"inputs"|"headings"})` — find interactive elements

### Interaction
- `browser_fill({selector: "#search", text: "query"})` — fill input fields
- `browser_click({selector: "button"})` — click elements (uses native CDP mouse events)
- `browser_eval({expression: "document.title"})` — execute arbitrary JS

### 🔥 Groundbreaking: Live Page Watching
- `browser_watch({action: "start", events: ["all"]})` — start recording page activity
- `browser_goto(...)` / `browser_click(...)` — interact with the page
- `browser_watch({action: "poll"})` — get all console messages, network requests, errors that happened
- `browser_watch({action: "stop"})` — stop recording

This is how you debug SPAs, detect React errors, see API calls, and understand what the page is DOING
internally — not just what it looks like.

### Smart Waiting
- `browser_waitForSelector({selector: ".results", timeout: 10000})` — wait for content to appear
- `browser_waitForSelector({selector: ".loading", disappear: true})` — wait for loading to finish

### Viewport Control

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krshforever/bwb-browser](https://github.com/krshforever/bwb-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
