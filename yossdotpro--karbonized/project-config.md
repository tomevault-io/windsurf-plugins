---
trigger: always_on
description: **Agent** is the design assistant built into Karbonized. You describe what you want and it edits the canvas with the same actions you use: it adds and updates blocks, aligns them, changes the background and exports images. It works with the model provider you choose.
---

# Agent and the MCP server

**Agent** is the design assistant built into Karbonized. You describe what you want and it edits the canvas with the same actions you use: it adds and updates blocks, aligns them, changes the background and exports images. It works with the model provider you choose.

The desktop app can also expose those actions as a local **MCP server**, so Claude Desktop, Claude Code, Cursor and other MCP clients can control Karbonized.

- [Using Agent](#using-agent)
- [Model providers](#model-providers)
- [API keys](#api-keys)
- [Undo](#undo)
- [Tools](#tools)
- [MCP server (desktop app)](#mcp-server-desktop-app)
- [Troubleshooting](#troubleshooting)
- [For contributors](#for-contributors)

## Using Agent

Open the panel with <kbd>Ctrl</kbd>/<kbd>⌘</kbd> + <kbd>L</kbd>, the **Agent** button in the status bar, the **AI** menu in the menu bar or **Show Agent** in the command palette. It docks on the left edge of the window, opposite the properties panel. The layout button at the left of the status bar switches between the canvas alone, the properties panel, the agent, or both.

- Type a request and press <kbd>Enter</kbd> (<kbd>Shift</kbd> + <kbd>Enter</kbd> for a new line).
- Answers stream in. Every action shows up as a card; open it to see its arguments, its result and how long it took.
- **Stop** ends the response at any point. **Retry** asks again from your last message.
- The model picker under the message box switches between your providers.
- **New chat** starts over; the history button reopens the last 30 chats. Chats are stored on your device.

When the model accepts images, Agent can look at a snapshot of the canvas to check its work.

## Model providers

Open **Agent settings** (gear in the panel, or the command palette) and add a provider:

| Provider          | Base URL                                    | Key      | Notes                                           |
| ----------------- | ------------------------------------------- | -------- | ----------------------------------------------- |
| Anthropic         | `https://api.anthropic.com`                 | Required |                                                 |
| OpenAI            | `https://api.openai.com/v1`                 | Required |                                                 |
| Google Gemini     | `https://generativelanguage.googleapis.com` | Required |                                                 |
| OpenRouter        | `https://openrouter.ai/api/v1`              | Required | Any model on OpenRouter                         |
| Ollama            | `http://localhost:11434/v1`                 | No       | Local models                                    |
| LM Studio         | `http://localhost:1234/v1`                  | No       | Local models                                    |
| OpenAI-compatible | your server                                 | Optional | Any server that speaks the Chat Completions API |

**Browse** lists the models of the provider, and **Test connection** checks the base URL and key. Pick a model that supports tool calling; small local models may struggle with multi-step edits.

### Web version and CORS

The web app calls providers straight from the browser. Anthropic, OpenAI, Gemini and OpenRouter allow it. For local servers:

- **Ollama**: allow the site with the `OLLAMA_ORIGINS` environment variable (set it to the address of the site, for example `OLLAMA_ORIGINS=https://example.github.io`) and restart Ollama.
- **LM Studio**: turn on **Enable CORS** in the server settings.
- **Other servers**: enable CORS on the server, or use the desktop app.

The desktop app sends provider requests from its main process, so CORS never applies there.

## API keys

Keys never leave your device except in requests to their provider, and they are never logged.

- **Desktop app**: keys are encrypted with the system keychain and only decrypted by the main process when it sends a request. The page cannot read them back, and a key only works with the base URL it was saved for. If you change the base URL, save the key again.
- **Web app**: keys are stored in this browser (IndexedDB).

## Undo

Everything Agent does in one response is **one undo step**: press <kbd>Ctrl</kbd> + <kbd>Z</kbd> on the canvas, or **Undo changes** under the response. Changes made by MCP clients undo one tool call at a time. Your own edits made while Agent works are never merged into its step.

## Tools

Agent and the MCP server share the same tools:

| Tool                                   | What it does                                                                                                                         |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `get_design_guide`                     | The design standards (sizes per platform, layout, type, color, HTML block rules, final checklist)                                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yossdotpro/karbonized](https://github.com/yossdotpro/karbonized) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
