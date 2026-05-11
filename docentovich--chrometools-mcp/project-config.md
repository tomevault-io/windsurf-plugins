---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**chrometools-mcp** - MCP (Model Context Protocol) server for Chrome automation using Puppeteer.

Cross-platform MCP server (Windows/WSL, Linux, macOS) that provides Chrome browser automation capabilities.

## Key Architecture Concept

**Persistent Browser Sessions**: Browser tabs remain open after command execution, allowing users to:
- Prepare pages manually between AI iterations
- Continue work from previous state
- Interact with pages between MCP server calls

This enables iterative workflows where the user and AI collaborate within the same browser session.

## Installation & Usage

Users install and run via npx (single command, no setup):

```bash
npx chrometools-mcp
```

This design makes installation automatic and simple for end users.

## Development Commands

```bash
# Install dependencies
npm install

# Run tests
npm test

# Build
npm run build

# Development/testing
npm start
# or
node index.js
```

## Technology Stack

- **Puppeteer**: Chrome automation library
- **MCP SDK**: `@modelcontextprotocol/sdk` for server implementation
- **Transport**: stdio (standard for MCP servers)

## Core Workflow

1. User runs `npx chrometools-mcp` (or adds to MCP config)
2. MCP server launches Puppeteer with `headless: false`
3. AI sends commands via MCP tools → Server executes in Chrome
4. Browser tabs **stay open** after execution
5. User can manually interact with pages
6. Next AI request continues from current browser state

## Implementation Guidelines

### Puppeteer Configuration

```javascript
// Browser should persist between commands
const browser = await puppeteer.launch({
  headless: false,  // User needs to see and interact
  // Keep browser alive between MCP calls
});
```

### Tab Management

- **DO NOT close tabs** after command execution
- Reuse existing tabs when possible
- Allow user to open/navigate tabs manually
- Track tab state between MCP requests

### MCP Tools Design

Expected tools might include:
- Navigate to URL
- Click elements
- Fill forms
- Get page content
- Execute JavaScript
- Take screenshots
- Wait for elements

All tools should work with **current browser state**, not create new sessions.

## Important: Applying Code Changes

**⚠️ All code changes require MCP server restart to take effect!**

After modifying any code files (index.js, tool-schemas.js, tool-definitions.js, etc.):

1. **User must restart the MCP server** (or restart the MCP client like Claude Desktop/Telegram bot)
2. Only then will changes be applied
3. Testing with old server instance will show old behavior

**DO NOT** test changes without confirming the server was restarted. If testing fails unexpectedly, first verify the server was restarted.

```bash
# For development: restart the MCP client or run
npm start
```

## Testing

When testing locally:

```bash
# Test as users would install
npx .

# Or during development
npm test
```

## Cross-Platform Considerations

- Works in pure Linux/macOS
- Works in Windows WSL
- Works in native Windows
- Puppeteer handles Chrome/Chromium installation automatically
- No platform-specific code needed (Puppeteer abstracts this)

## Package Configuration

Ensure `package.json` includes:
- `bin` field for npx execution
- Proper entry point
- MCP server metadata
- Puppeteer as dependency

## MCP Configuration for Development

MCP servers are configured in `/home/user/.claude.json` file (NOT `.claude/mcp_config.json`).

For WSL development with GUI support, use xvfb-run:

```json
"mcpServers": {
  "chrometools": {
    "type": "stdio",
    "command": "xvfb-run",
    "args": ["-a", "node", "/mnt/c/prj/chrometools-mcp/index.js"],
    "env": {}
  }
}
```

xvfb provides virtual X server, allowing Chrome GUI to run without external X server (VcXsrv).

## Documentation Guidelines

When making changes to the codebase:

### What to Update (ALWAYS)

1. **README.md** - ALWAYS update when adding new tools or changing user-facing behavior

**IMPORTANT**: For new tools/features, ALWAYS update README.md to document them. This is NOT optional.

### Version Bump and CHANGELOG (ASK USER FIRST)

**DO NOT automatically bump version or update CHANGELOG after each atomic change!**

Instead, at the end of a work session (or when user requests):
1. **Ask user**: "Do you want me to bump version and update CHANGELOG for this session?"
2. If YES:
   - Increment version in `package.json` appropriately (patch/minor/major)
   - Add **single CHANGELOG entry** summarizing ALL changes from the session
   - Update `extension/manifest.json` version if extension was modified
3. If NO: Leave version and CHANGELOG unchanged

This prevents:
- Multiple tiny version bumps for related changes
- Fragmented CHANGELOG entries
- Version noise in git history

### What NOT to Create

**DO NOT create separate documentation files** for individual changes, such as:
- ❌ `PERFORMANCE_IMPROVEMENTS.md`
- ❌ `NEW_FEATURES.md`
- ❌ `MIGRATION_GUIDE.md`
- ❌ Individual feature documentation files

### Documentation Rules

1. **CHANGELOG.md** - All change documentation goes here (version history)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docentovich/chrometools-mcp](https://github.com/docentovich/chrometools-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
