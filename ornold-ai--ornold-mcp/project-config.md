---
trigger: always_on
description: You are an AI agent. A user has asked you to install Ornold MCP. Follow these steps.
---

# Instructions for AI Agents

You are an AI agent. A user has asked you to install Ornold MCP. Follow these steps.

## What is Ornold MCP?

Ornold MCP (`mcp-ornold-browser`) is an MCP server for browser automation: control antidetect browsers (Linken Sphere, Dolphin Anty, Wade X/Wadex, Vision Browser, AdsPower, Octo Browser, Multilogin, MoreLogin, 0DETECT, GoLogin, Undetectable, Incogniton, Indigo), solve CAPTCHAs, record and replay automation flows, and interact with web pages through vision or DOM.

npm package: `mcp-ornold-browser`

## Prerequisites

- **Node.js ≥ 18** — check with `node -v`. If missing, tell the user to install from https://nodejs.org
- **Ornold token** — starts with `orn_sk_`. Ask the user for it, or tell them to get one at https://mcp.ornold.com (sign up → Tokens page → Create token)

## Installation by Agent Type

Ask the user which antidetect browser and mode they need before installing. See "Recommended Setup" below for splitting into multiple servers.

### Claude Code
```bash
claude mcp add --transport stdio ornold-browser -- npx mcp-ornold-browser --token TOKEN --mode vision
npx mcp-ornold-browser --install-skill
```

### Cursor
Write to `~/.cursor/mcp.json` (create if missing, merge if exists):
```json
{
  "mcpServers": {
    "ornold-browser": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN"]
    }
  }
}
```
Tell user to restart Cursor.

### VS Code / Copilot
```bash
code --add-mcp '{"name":"ornold-browser","command":"npx","args":["mcp-ornold-browser","--token","TOKEN"]}'
```

### Codex
Write to `~/.codex/config.toml` (create if missing):
```toml
[mcp_servers.ornold-browser]
command = "npx"
args = ["-y", "mcp-ornold-browser", "--token", "TOKEN"]
```
Desktop apps don't inherit terminal PATH. If MCP fails, use absolute path from `which npx`.

### Windsurf
Tell user: Settings → Tools → Add MCP Server, paste:
```json
{
  "mcpServers": {
    "ornold-browser": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN"]
    }
  }
}
```

### Cline
Tell user: Cline sidebar → MCP → Add, paste:
```json
{
  "mcpServers": {
    "ornold-browser": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN"],
      "disabled": false
    }
  }
}
```

## Recommended Setup: Separate MCP Servers

**Important:** Create separate MCP server instances instead of loading everything into one. This keeps the tool list small and focused — the agent picks the right tool more reliably.

**Rule of thumb:** one MCP server = one interaction mode + one antidetect browser.

### Why split by mode?

- `dom` mode loads ~15 tools (snapshot, click, type, fill, hover, drag, select...)
- `vision` mode loads ~12 tools (screenshot, vision_analyze, click_normalized_box, flow recording...)
- `both` loads ~27 tools — the agent gets confused which click/type to use

Split them: one server for DOM work, one for vision work. If you only use one mode, one server is fine.

### Why split by antidetect browser?

- Each antidetect browser adds its own management tools (for example: Linken, Dolphin, Octo, AdsPower, Multilogin)
- If you use multiple antidetect browsers, create a separate MCP server per browser
- If you only use one antidetect browser, one server is enough

### Example: Claude Code with Linken Sphere (vision + dom)

```bash
# Vision mode — for stealth automation, flow recording
claude mcp add --transport stdio ornold-vision -- npx mcp-ornold-browser --token TOKEN --mode vision --linken-port 40080

# DOM mode — for fast scraping, form filling
claude mcp add --transport stdio ornold-dom -- npx mcp-ornold-browser --token TOKEN --mode dom --linken-port 40080
```

### Example: Cursor with two antidetect browsers

```json
{
  "mcpServers": {
    "ornold-linken": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN", "--mode", "vision", "--linken-port", "40080"]
    },
    "ornold-dolphin": {
      "command": "npx",
      "args": ["mcp-ornold-browser", "--token", "TOKEN", "--mode", "vision", "--dolphin-port", "3001", "--dolphin-token", "API_TOKEN"]
    }
  }
}
```

### Example: Single antidetect browser, single mode (simplest)

If you only use one browser and one mode, one server is fine:
```bash
claude mcp add --transport stdio ornold-browser -- npx mcp-ornold-browser --token TOKEN --mode vision --linken-port 40080
```

## Antidetect Browser Flags

Ask user which antidetect browser they use. Add flags to the args array:

| Browser | Flags |
|---------|-------|
| Linken Sphere | `"--linken-port", "40080"` |
| Dolphin Anty | `"--dolphin-port", "3001", "--dolphin-token", "API_TOKEN"` |
| Wade X / Wadex | `"--wadex-port", "8080"` |
| Vision Browser | `"--vision-token", "X_TOKEN", "--vision-port", "PORT"` |
| AdsPower | `"--adspower-url", "http://local.adspower.net:50325"` and optionally `"--adspower-key", "API_KEY"` |
| Octo Browser | `"--octo-token", "OCTO_API_TOKEN", "--octo-url", "http://127.0.0.1:58888"` |
| Multilogin | `"--multilogin-token", "API_TOKEN", "--multilogin-url", "http://127.0.0.1:35000"` |
| MoreLogin | `"--morelogin-port", "40000"` or `"--morelogin-url", "http://127.0.0.1:40000"` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ornold-ai/ornold-mcp](https://github.com/ornold-ai/ornold-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
