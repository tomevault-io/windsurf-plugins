---
trigger: always_on
description: Development guidelines for AI-assisted development on the Inspekt project.
---

# Claude Development Guide for Inspekt

Development guidelines for AI-assisted development on the Inspekt project.

---

## Running CLI Commands

**Always use `inspekt` directly, never `python -m inspekt`.**

```bash
# Correct
inspekt inspected css --help

# Wrong - don't do this
python -m inspekt inspected css --help
```

---

## Proactive Command Execution

When explaining shell commands to the user, **always offer to execute them**:

> "Would you like me to run these commands for you?"

This applies to documentation servers, file operations, build/test commands, git operations, etc.

**Exception**: Don't offer to run destructive commands without explicit user confirmation.

---

## Inspekt MCP Tools (CRITICAL)

**ALWAYS use Inspekt MCP tools for ANY browser interaction.** Never invent alternatives.

### Golden Rules

1. **NEVER use AppleScript, osascript, or Automator** for browser automation
2. **NEVER use `screencapture`, `screenshot`, or system screenshot tools** - use `mcp__inspekt__take_screenshot`
3. **NEVER use `curl` or `wget` to fetch web pages** - use Inspekt MCP tools
4. **NEVER try to control Safari, Chrome, or Firefox directly** via shell commands
5. **NEVER use WebFetch for pages that need JS rendering or authentication** - Inspekt has access to the user's actual browser session

### When to Use What

| Task | ✅ Use This | ❌ NOT This |
|------|------------|-------------|
| Screenshot | `mcp__inspekt__take_screenshot` | `screencapture`, AppleScript |
| Navigate | `mcp__inspekt__navigate_to_url` | `open -a Safari`, AppleScript |
| Click element | `mcp__inspekt__click_element` | AppleScript, osascript |
| Type text | `mcp__inspekt__type_text` | AppleScript keyboard events |
| Press keys | `mcp__inspekt__press_keys` | AppleScript, osascript |
| Get page content | `mcp__inspekt__extract_article` | curl, wget, WebFetch |
| Run accessibility audit | `mcp__inspekt__run_axe` | External tools |
| Execute JavaScript | `mcp__inspekt__execute_javascript` | Browser DevTools manually |

### Available MCP Tools (33 total)

**Navigation:**
- `navigate_to_url` - Go to a URL
- `go_back` - Browser back button
- `go_forward` - Browser forward button
- `reload_page` - Refresh the page
- `scroll_to_top`, `scroll_to_bottom` - Scroll to edges
- `page_up`, `page_down` - Scroll by viewport

**Interaction:**
- `click_element` - Click by CSS selector
- `type_text` - Type into focused element
- `paste_text` - Paste text (faster for long text)
- `press_keys` - Keyboard shortcuts (Tab, Enter, Ctrl+A, etc.)

**Inspection:**
- `get_page_info` - URL, title, viewport size
- `take_screenshot` - Capture viewport, full page, or element
- `get_selected_text` - Get user's text selection

**Extraction:**
- `extract_links` - All links on page
- `extract_outline` - Heading hierarchy
- `extract_page_info` - Meta tags, Open Graph
- `extract_article` - Main content (Readability)

**AI-Powered:**
- `summarize` - AI article summary
- `describe` - AI page description for screen readers
- `ask` - Ask questions about the page
- `do` - Natural language actions ("click the login button")
- `index_page` - Index page for AI queries
- `md_link` - Get markdown link for current page

**Accessibility:**
- `run_axe` - axe-core WCAG audit
- `check_autocomplete` - Form field autocomplete check

**Network:**
- `get_network_requests` - Performance API network data
- `get_har` - Full HAR data (requires DevTools open)

**Storage:**
- `get_cookies`, `set_cookie` - Cookie management

**Console:**
- `get_console_logs` - Browser console messages
- `clear_console_logs` - Clear console buffer

### Troubleshooting

If an MCP tool fails:
1. Check bridge connection: `mcp__inspekt__get_page_info`
2. Verify browser extension is installed and enabled
3. Make sure `inspekt start` is running (check with `inspekt status`)

**Never fall back to AppleScript or shell hacks - ask the user to check the connection instead.**

---

## Browser Debugging

Use `inspekt console` commands autonomously instead of asking users to check their browser console.

---

## Browser VM Development

For the full VM deployment workflow, invoke the `/vm` skill.

**Quick reference — what to do after file changes:**

| Changed | Command |
|---------|---------|
| Python code (`inspekt/`) | Nothing (hot-reload in dev mode) |
| `control-server.py` | `make vm-restart-control` |
| `terminal-server.py` | `make vm-restart-terminal` |
| `control-panel.html` | `make vm-restart` |
| Chrome extension | `make vm-restart-chromium` |
| `proxy-scripts/*` | `make vm-restart-proxy` |
| Dockerfile / supervisord / entrypoint | `make vm-rebuild` |

**Key ports:** Control server = **8888**, noVNC = 6080, Terminal WS = 8889, CDP = 9222, mitmproxy = 8080

- Always use `get_bridge_port()` from `inspekt/config.py`, never hardcode ports
- Dev mode auto-enables when running from source repo

See [docs/development/vm.md](docs/development/vm.md) for full documentation.

---

## Command Palette Output Modes

Commands launched from the VM command palette (Cmd+K) use one of three output modes:

| Mode | When to use | How it works |
|------|-------------|--------------|
| **toast** | Fire-and-forget (no output) | Calls API, shows brief notification |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/roelvangils) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
