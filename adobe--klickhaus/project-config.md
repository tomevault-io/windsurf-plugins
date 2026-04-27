---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

When opening pull requests, use the GitHub PR template in `.github/pull_request_template.md` and fill out the Summary, Testing Done, and Checklist sections.

## Naming Conventions

This project enforces strict naming conventions via ESLint rules. All contributions must follow these patterns:

### Variables and Functions
- **Style**: camelCase
- **Examples**: `userData`, `fetchLogs`, `handleClick`, `isValid`
- **Enforced by**: `camelcase` ESLint rule

### Constants
- **Style**: SCREAMING_SNAKE_CASE for module-level constants
- **Examples**: `TIME_RANGES`, `DEFAULT_TOP_N`, `API_BASE_URL`
- **Enforced by**: `camelcase` ESLint rule with SCREAMING_SNAKE_CASE allowed

### Classes and Constructors
- **Style**: PascalCase
- **Examples**: `DataProcessor`, `ChartRenderer`, `FilterManager`
- **Enforced by**: `new-cap` ESLint rule

### File Names
- **Style**: kebab-case for JavaScript files
- **Examples**: `url-state.js`, `facet-search.js`, `step-detection.js`
- **Exception**: Test files append `.test.js` suffix

### Private Members
- **Style**: Underscore prefix allowed only after `this`
- **Examples**: `this._internalState`, `this._cache`
- **Enforced by**: `no-underscore-dangle` ESLint rule

### DOM Element IDs and Classes
- **Style**: camelCase for IDs, kebab-case for CSS classes
- **Examples**: `id="loginForm"`, `class="dashboard-content"`

Run `npm run lint` to verify naming conventions are followed.

## Overview

This repository contains the analytics dashboard for CDN log data for Adobe Experience Manager (AEM) Edge Delivery Services (formerly Helix). CDN logs from Cloudflare and Fastly are ingested via GCS into four ClickHouse tables (`delivery`, `delivery_errors`, `admin`, `backend`) by the [helix-gcs2clickhouse-ingestor](https://github.com/adobe/helix-gcs2clickhouse-ingestor) Cloud Run service.

## Development

```bash
npm install
npm start
```

This starts a dev server with auto-reload. The port is deterministic per worktree and printed on startup. Use `node scripts/dev-server.mjs --dry-run` to get the port without starting the server.

## Browser Exploration with playwright-cli

This project includes the `playwright-cli` skill (`.claude/skills/playwright-cli/`) for interactive browser automation. **Use it as a first step when investigating bugs or exploring new features** before writing formalized tests.

### When to Use

- **Bug investigation**: Open the dashboard, reproduce the issue, inspect DOM state, check console errors and network requests
- **Feature exploration**: Navigate the UI to understand current behavior before implementing changes
- **Visual verification**: Take screenshots to confirm rendering after code changes
- **Ad-hoc testing**: Quickly validate a fix in a real browser before committing to a formal test

### Quick Start

```bash
# Start the dev server first
npm start

# Open the dashboard and explore
playwright-cli open http://localhost:$(node scripts/dev-server.mjs --dry-run)/dashboard.html
playwright-cli snapshot
playwright-cli fill e3 "<username>"
playwright-cli fill e5 "<password>"
playwright-cli click e7
playwright-cli snapshot
playwright-cli screenshot
```

### Workflow: Explore, Then Formalize

1. **Explore** the bug or feature interactively with `playwright-cli` (snapshot, click, inspect console/network)
2. **Understand** the root cause or behavior using `playwright-cli eval`, `playwright-cli console`, `playwright-cli network`
3. **Fix** the code
4. **Verify** the fix with `playwright-cli` (screenshot, re-test the flow)
5. **Formalize** the findings into a unit test in `js/**/*.test.js` using `@web/test-runner`

### Key Commands

```bash
playwright-cli snapshot              # Capture page structure with element refs
playwright-cli click e3              # Click element by ref from snapshot
playwright-cli fill e5 "text"        # Fill input field
playwright-cli eval "document.title" # Run JS on page
playwright-cli console               # View console messages
playwright-cli network               # View network requests
playwright-cli screenshot            # Capture screenshot
playwright-cli screenshot e12        # Screenshot specific element
```

See `.claude/skills/playwright-cli/SKILL.md` for full command reference and `.claude/skills/playwright-cli/references/` for advanced topics (request mocking, tracing, session management).

### Credentials

Dashboard login credentials are in `README.local.md` under the Users table. Use a read-only user (e.g., `lars` or `david_query`) for testing.

## Database Connection

```bash
clickhouse client --host s2p5b8wmt5.eastus2.azure.clickhouse.cloud \
  --user default --password '<see README.local.md>' --secure
```

Database: `helix_logs_production`

## User Management

Scripts in `scripts/` manage read-only ClickHouse users:

```bash
# Add a new read-only user (generates password if not provided)
node scripts/add-user.mjs <admin-user> <admin-password> <new-username> [password]

# Rotate a user's password
node scripts/roll-user.mjs <admin-user> <admin-password> <username>

# Remove a user
node scripts/drop-user.mjs <admin-user> <admin-password> <username>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adobe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
