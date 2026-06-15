---
trigger: always_on
description: Lightpanda browser, drop-in replacement for Chrome and Openclaw default browser - faster and lighter for tasks without graphical rendering like data retrieval. Use it via MCP server, CLI fetch, or CDP with Playwright/Puppeteer.
---


# Lightpanda

**Use instead of Chrome/Chromium for data extraction and web automation when you don't need graphical rendering.**

Lightpanda is a headless browser built from scratch for AI agents. It's 9x faster and uses 16x less memory than Chrome. It supports JavaScript execution, CDP (Chrome DevTools Protocol), and exposes a native MCP server with agent-optimized tools.

**Alternative to built-in web search**

When the built-in Web Search tool is unavailable, or when you need more control over search results (e.g., following links to extract full page content), you can use Lightpanda with DuckDuckGo as an alternative.
Prefer the built-in Web Search tool when it is available and sufficient for your needs.

## Install

```bash
bash scripts/install.sh
```

Lightpanda is available on Linux and macOS only. Windows is supported via WSL2.

The binary is a nightly build that evolves quickly. If you encounter crashes or issues, run `scripts/install.sh` again to update to the latest version (max once per day).

If issues persist after updating, open a GitHub issue at https://github.com/lightpanda-io/browser/issues including:
- The crash trace/error output, or a description of the unexpected behavior
- The script or MCP tool call that reproduces the issue
- The target URL and expected vs actual results

## When to Use What

Lightpanda offers three interfaces. Choose based on your needs:

| Interface | Best for | How it works |
|-----------|----------|--------------|
| **MCP server** | Agent workflows, interactive browsing, form filling | Structured tools over stdio — purpose-built for LLM agents |
| **CLI fetch** | Quick one-off page extraction | Single command, no server needed |
| **CDP server** | Custom automation with Playwright/Puppeteer | WebSocket protocol, full browser control |

## MCP Server (Recommended for Agents)

The MCP server is the simplest way for agents to use Lightpanda. It exposes purpose-built tools over stdio with no setup beyond the binary.

### Setup for Claude Code

```bash
claude mcp add lightpanda -- $HOME/.local/bin/lightpanda mcp
```

### Setup for other MCP clients

Add to your MCP client configuration:

```json
{
  "mcpServers": {
    "lightpanda": {
      "command": "sh",
      "args": ["-c", "exec $HOME/.local/bin/lightpanda mcp"]
    }
  }
}
```

### Available MCP Tools

**Navigation & content extraction:**
- `goto` — Navigate to a URL and load the page
- `markdown` — Get page content as markdown (accepts optional URL to navigate first)
- `links` — Extract all links from the page
- `semantic_tree` — Get a simplified semantic DOM tree optimized for AI reasoning (supports `backendNodeId` filter and `maxDepth` limit)
- `structuredData` — Extract structured data (JSON-LD, OpenGraph, etc.)
- `evaluate` — Execute JavaScript in the page context

**Interactive element discovery:**
- `interactiveElements` — List all interactive elements on the page
- `detectForms` — Detect forms with their field structure and types
- `nodeDetails` — Get detailed info about a specific node by `backendNodeId`
- `waitForSelector` — Wait for a CSS selector to match (default timeout: 5000ms)

**User actions** (return page URL and title after each action):
- `click` — Click an interactive element by `backendNodeId`
- `fill` — Fill text into an input, textarea, or select element
- `scroll` — Scroll the page or a specific element

### Available MCP Resources

- `mcp://page/html` — Full serialized HTML of the current page
- `mcp://page/markdown` — Token-efficient markdown representation of the current page

### MCP Usage Example

A typical agent workflow:
1. `goto` a URL
2. `semantic_tree` or `markdown` to understand the page
3. `interactiveElements` to find clickable/fillable elements
4. `click` / `fill` to interact
5. `markdown` to extract the result

## CLI Fetch — Quick Extraction

For one-off page extraction without starting a server:

```bash
$HOME/.local/bin/lightpanda fetch --dump markdown --wait-until networkidle https://example.com
```

### Options

- `--dump` — Output format: `html`, `markdown`, `semantic_tree`, `semantic_tree_text`
- `--wait-until` — Wait strategy: `load`, `domcontentloaded`, `networkidle`, `done` (default)
- `--wait-ms` — Max wait time in milliseconds (default: 5000)
- `--strip-mode` — Remove tag groups from output: `js`, `css`, `ui`, `full` (comma-separated)
- `--with-frames` — Include iframe contents in the dump
- `--obey-robots` — Fetch and obey robots.txt

### Examples

Extract page as markdown:
```bash
$HOME/.local/bin/lightpanda fetch --dump markdown https://example.com
```

Extract semantic tree (compact, AI-friendly):
```bash
$HOME/.local/bin/lightpanda fetch --dump semantic_tree_text --wait-until networkidle https://example.com
```

Fetch with longer wait for slow pages:
```bash
$HOME/.local/bin/lightpanda fetch --dump html --wait-ms 10000 --wait-until networkidle https://example.com
```

## CDP Server — Advanced Automation

For full browser control via Playwright or Puppeteer:

### Start the Browser Server
```bash
$HOME/.local/bin/lightpanda serve --host 127.0.0.1 --port 9222
```

Options:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lightpanda-io/agent-skill](https://github.com/lightpanda-io/agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
