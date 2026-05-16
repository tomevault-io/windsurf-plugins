---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WebDriverIO MCP Server is a Model Context Protocol (MCP) server that enables Claude Desktop to interact with web browsers and mobile applications using WebDriverIO for automation. The server supports:
- **Browser automation**: Chrome browser control (headed/headless)
- **Mobile app automation**: iOS and Android native app testing via Appium
- **Cross-platform**: Unified API for web, iOS, and Android automation

The server is published as an npm package (`webdriverio-mcp`) and runs via stdio transport.

## Development Commands

### Build and Package
```bash
npm run bundle           # Clean, build with tsup, make executable, and create .tgz package
npm run prebundle        # Clean lib directory and .tgz files
npm run postbundle       # Create npm package tarball
```

### Run Server
```bash
npm run dev              # Run development server with tsx (no build)
npm start                # Run built server from lib/server.js
```

## Architecture

### Core Components

**Server Entry Point** (`src/server.ts`)
- Initializes MCP server using `@modelcontextprotocol/sdk`
- Redirects console output to stderr to avoid interfering with MCP protocol (Chrome writes to stdout)
- Registers all tool handlers with the MCP server
- Uses StdioServerTransport for communication with Claude Desktop

**Session State Management** (`src/tools/browser.tool.ts` and `src/tools/app-session.tool.ts`)
- Maintains global state with three Maps:
  - `browsers`: Map<sessionId, WebdriverIO.Browser> - stores all browser/app instances
  - `currentSession`: string | null - tracks the single active session
  - `sessionMetadata`: Map<sessionId, {type, capabilities}> - tracks session type and config
- `getBrowser()` helper retrieves the current active browser/app instance
- `startBrowserTool` creates Chrome browser session with configurable options:
  - Headless mode support
  - Custom window dimensions (400-3840 width, 400-2160 height)
  - Chrome-specific arguments (sandbox, security, media stream, etc.)
- `startAppTool` creates iOS/Android app session via Appium with platform-specific capabilities:
  - `noReset`: Controls whether to preserve app state between sessions (default: false)
  - `fullReset`: Controls whether to uninstall app before/after session (default: true)
  - Sessions created with `noReset: true` will automatically detach on close (preserves session state)
- `closeSessionTool` properly cleans up browser/app sessions and metadata:
  - `detach: false` (default): Calls `deleteSession()` to terminate on server
  - `detach: true`: Disconnects without terminating (preserves session for manual testing)
  - Sessions created without `appPath` or with `noReset: true` automatically detach

**Tool Pattern**
All tools follow a consistent pattern:
1. Export Zod schema for arguments validation (e.g., `navigateToolArguments`)
2. Export ToolCallback function (e.g., `navigateTool`)
3. Use `getBrowser()` to access current session
4. Return `CallToolResult` with text content
5. Wrap operations in try-catch and return errors as text content

**Browser Script Execution** (`src/scripts/get-interactable-elements.ts`)
- Returns a function that executes in the browser context (not Node.js)
- `getInteractableElements()` finds all visible, interactable elements on the page
- Uses modern `element.checkVisibility()` API with fallback for older browsers
- Generates CSS selectors using IDs, classes, or nth-child path-based selectors
- Returns element metadata: tagName, type, id, className, textContent, value, placeholder, href, ariaLabel, role, cssSelector, isInViewport

**Mobile Element Detection** (`src/locators/` and `src/utils/mobile-elements.ts`)
- Uses XML-based page source parsing to extract all element attributes
- Platform-specific element classification:
  - `ANDROID_INTERACTABLE_TAGS`: Button, EditText, CheckBox, RadioButton, Switch, Spinner, etc.
  - `ANDROID_LAYOUT_CONTAINERS`: ViewGroup, LinearLayout, RelativeLayout, FrameLayout, ScrollView, etc.
  - `IOS_INTERACTABLE_TAGS`: Button, TextField, SecureTextField, Switch, Picker, etc.
  - `IOS_LAYOUT_CONTAINERS`: View, ScrollView, StackView, CollectionView, etc.
- Generates multiple locator strategies per element:
  - Accessibility ID (cross-platform)
  - Resource ID / Name
  - Text / Label matching
  - XPath (full and simplified)
  - UiAutomator (Android) / Predicates (iOS)
- Smart filtering with `inViewportOnly` and `includeContainers` parameters

### Build Configuration

**TypeScript** (`tsconfig.json`)
- Target: ES2022, Module: ESNext
- Source: `src/`, Output: `build/` (but not used for distribution)
- Strict mode disabled
- Includes types for Node.js and `@wdio/types`

**Bundler** (`tsup.config.ts`)
- Entry: `src/server.ts`
- Output: `lib/` directory (ESM format only)
- Generates declaration files and sourcemaps
- Externalizes `zod` dependency
- The shebang `#!/usr/bin/env node` in server.ts is preserved for CLI execution

### Selector Syntax

**Web Browsers:**
- CSS selectors: `button.my-class`, `#element-id`
- XPath: `//button[@class='my-class']`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Winify/webdriverio-mcp](https://github.com/Winify/webdriverio-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
