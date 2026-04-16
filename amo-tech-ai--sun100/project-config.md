---
trigger: always_on
description: Guidelines for using Chrome DevTools MCP for browser automation, debugging, and performance testing
---


# Chrome DevTools MCP - Cursor Rule

## Overview

Chrome DevTools MCP (`chrome-devtools-mcp`) provides full access to Chrome DevTools Protocol for browser automation, debugging, performance analysis, and testing. It acts as a Model-Context-Protocol (MCP) server, giving AI assistants access to Chrome DevTools capabilities.

**Repository:** [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)  
**NPM Package:** [chrome-devtools-mcp](https://npmjs.org/package/chrome-devtools-mcp)

## Key Features

- **Performance Insights**: Record traces and extract actionable performance insights using Chrome DevTools
- **Advanced Browser Debugging**: Analyze network requests, take screenshots, check browser console
- **Reliable Automation**: Uses Puppeteer to automate Chrome actions with automatic wait for results
- **Full DevTools Access**: Complete access to Chrome DevTools Protocol capabilities

## Requirements

- **Node.js**: v20.19 or newer (latest maintenance LTS)
- **Chrome**: Current stable version or newer
- **npm**: For package management

## Configuration

### Basic Setup for Cursor

Add to `.cursor/mcp.json` or Cursor Settings → MCP → New MCP Server:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

> **Note:** Using `chrome-devtools-mcp@latest` ensures you always use the latest version.

### Advanced Configuration Options

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "chrome-devtools-mcp@latest",
        "--channel=canary",           // stable, canary, beta, dev
        "--headless=true",            // Run without UI
        "--isolated=true",            // Temporary user data dir (auto-cleanup)
        "--viewport=1920x1080",       // Initial viewport size
        "--browser-url=http://127.0.0.1:9222"  // Connect to running Chrome
      ]
    }
  }
}
```

### Configuration Options Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--browserUrl`, `-u` | string | - | Connect to running Chrome via port forwarding |
| `--wsEndpoint`, `-w` | string | - | WebSocket endpoint (e.g., `ws://127.0.0.1:9222/devtools/browser/<id>`) |
| `--wsHeaders` | string | - | Custom headers for WebSocket (JSON format) |
| `--headless` | boolean | `false` | Run in headless (no UI) mode |
| `--executablePath`, `-e` | string | - | Path to custom Chrome executable |
| `--isolated` | boolean | `false` | Use temporary user-data-dir (auto-cleanup) |
| `--userDataDir` | string | `$HOME/.cache/chrome-devtools-mcp/chrome-profile-$CHANNEL` | Custom user data directory |
| `--channel` | string | `stable` | Chrome channel: `stable`, `canary`, `beta`, `dev` |
| `--logFile` | string | - | Path to debug log file |
| `--viewport` | string | - | Initial viewport size (e.g., `1280x720`) |
| `--proxyServer` | string | - | Proxy server configuration |
| `--acceptInsecureCerts` | boolean | `false` | Ignore self-signed/expired certificates |
| `--chromeArg` | array | - | Additional Chrome arguments |
| `--categoryEmulation` | boolean | `true` | Include emulation tools |
| `--categoryPerformance` | boolean | `true` | Include performance tools |
| `--categoryNetwork` | boolean | `true` | Include network tools |

## Available Tools

### Input Automation (8 tools)
- `click` - Click elements on the page
- `drag` - Drag and drop elements
- `fill` - Fill input fields
- `fill_form` - Fill entire forms
- `handle_dialog` - Handle browser dialogs (alert, confirm, prompt)
- `hover` - Hover over elements
- `press_key` - Press keyboard keys
- `upload_file` - Upload files to file inputs

### Navigation Automation (6 tools)
- `close_page` - Close a page
- `list_pages` - List all open pages
- `navigate_page` - Navigate to a URL
- `new_page` - Open a new page
- `select_page` - Switch to a specific page
- `wait_for` - Wait for conditions (element, text, time)

### Emulation (2 tools)
- `emulate` - Emulate devices (mobile, tablet, etc.)
- `resize_page` - Resize the viewport

### Performance (3 tools)
- `performance_analyze_insight` - Analyze performance and get insights
- `performance_start_trace` - Start performance tracing
- `performance_stop_trace` - Stop performance tracing

### Network (2 tools)
- `get_network_request` - Get details of a specific network request
- `list_network_requests` - List all network requests

### Debugging (5 tools)
- `evaluate_script` - Execute JavaScript in the page context
- `get_console_message` - Get a specific console message
- `list_console_messages` - List all console messages
- `take_screenshot` - Capture a screenshot
- `take_snapshot` - Capture accessibility snapshot

## When to Use Chrome DevTools MCP

### ✅ DO: Use For

1. **Performance Testing**
   - Record performance traces
   - Analyze page load times
   - Identify performance bottlenecks
   - Get actionable performance insights

2. **Browser Automation**
   - End-to-end testing workflows
   - Form filling and submission
   - Multi-step user journeys
   - Cross-browser testing (via emulation)

3. **Debugging**
   - Console error investigation
   - Network request analysis

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amo-tech-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
