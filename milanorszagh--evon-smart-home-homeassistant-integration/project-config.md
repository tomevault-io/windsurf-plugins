---
trigger: always_on
description: **IMPORTANT FOR CLAUDE:** This is the primary reference document. Always check this file first when working with this codebase. The Evon MCP server is available for direct API queries - prefer using it over writing scripts.
---

# AGENTS.md - AI Agent Guidelines for Evon Smart Home

**IMPORTANT FOR CLAUDE:** This is the primary reference document. Always check this file first when working with this codebase. The Evon MCP server is available for direct API queries - prefer using it over writing scripts.

**Commit messages:** Do not add Co-Authored-By lines to commit messages.

**Before committing:** Always run `ruff check custom_components/evon/ tests/ && ruff format --check custom_components/evon/ tests/` to catch linting/formatting issues before they fail CI. Use `--fix` and `ruff format` to auto-fix.

**GitHub Releases:** Do NOT include the version number in the release title. HACS automatically prepends the tag name (e.g., `v1.19.2 - `), so including it in the title creates duplication like `v1.19.2 - v1.19.2 — Fix ...`. Use only a descriptive title, e.g., `--title "Fix double-click detection"`.

This document provides critical information for AI agents working with this codebase.

## Project Overview

This repository contains two integrations for Evon Smart Home systems:
- **MCP Server** (`src/index.ts`) - TypeScript-based Model Context Protocol server
- **Home Assistant Integration** (`custom_components/evon/`) - Python-based HA custom component

## Documentation Structure

**IMPORTANT**: When updating documentation, put content in the correct file based on audience:

| File | Audience | Content |
|------|----------|---------|
| **README.md** | End users | Installation, features, configuration, platform descriptions (user-friendly, no internal details), version history |
| **DEVELOPMENT.md** | Developers | Architecture, API reference, device classes, methods, code patterns, testing, MCP server setup |
| **AGENTS.md** | AI agents | Critical API knowledge, debugging tips, gotchas, implementation patterns, version history (detailed) |

### What Goes Where

| Content Type | File |
|--------------|------|
| How to install/configure | README.md |
| What features are supported | README.md |
| API endpoints and methods | DEVELOPMENT.md |
| Internal property values (e.g., ModeSaved) | DEVELOPMENT.md, AGENTS.md |
| Password encoding details | DEVELOPMENT.md |
| Debugging tips and gotchas | AGENTS.md |
| Code patterns and examples | DEVELOPMENT.md, AGENTS.md |
| MCP tools/resources tables | DEVELOPMENT.md |
| Version history (brief) | README.md |
| Version history (detailed) | AGENTS.md |

### Guidelines

1. **README.md should be user-friendly** - No internal implementation details, no API property names, no code examples
2. **DEVELOPMENT.md is for developers** - Technical details, API reference, code patterns
3. **AGENTS.md is for AI agents** - Critical knowledge that prevents mistakes, debugging tips, gotchas
4. **Keep formatting consistent** - Use tables for structured data, code blocks for examples
5. **Update version history** - Brief in README.md, detailed in AGENTS.md
6. **Don't duplicate** - Link to other docs instead of copying content

## Remote Access API - CRITICAL

The Evon API supports both local and remote access. **Remote access has different authentication requirements.**

### Connection Types

| Type | Base URL | Use Case |
|------|----------|----------|
| **Local** | `http://{local-ip}` | Direct LAN connection (faster, recommended) |
| **Remote** | `https://my.evon-smarthome.com` | Internet access via relay server |

### Remote Login - IMPORTANT DIFFERENCES

**Local login:**
```
POST http://{local-ip}/login
Headers:
  x-elocs-username: <username>
  x-elocs-password: <encoded-password>
```

**Remote login:**
```
POST https://my.evon-smarthome.com/login   ← Note: /login at ROOT, NOT /{engine-id}/login
Headers:
  x-elocs-username: <username>
  x-elocs-password: <encoded-password>
  x-elocs-relayid: <engine-id>              ← REQUIRED for remote
  x-elocs-sessionlogin: true                ← REQUIRED for remote
  X-Requested-With: XMLHttpRequest          ← REQUIRED for remote
```

**Critical gotcha:** The remote login URL is `https://my.evon-smarthome.com/login` (at root), NOT `https://my.evon-smarthome.com/{engine-id}/login`. The engine ID goes in the `x-elocs-relayid` header only.

### Remote API Calls

After login, API calls use different base URLs:

| Type | API Base URL |
|------|--------------|
| Local | `http://{local-ip}/api/instances/...` |
| Remote | `https://my.evon-smarthome.com/{engine-id}/api/instances/...` |

The actual API methods (get instances, turn on/off, etc.) are identical - the relay server proxies requests to the local Evon system.

### Engine ID

The Engine ID is found in your Evon system settings. It identifies your installation on the relay server (e.g., `985315`).

---

## Critical API Knowledge

### Brightness Control - IMPORTANT

**DO NOT use `BrightnessSetInternal`** - this sets an internal value but does not change the physical light brightness.

**USE `BrightnessSetScaled`** (canonical name) - this is the correct method that controls the actual physical brightness. The HTTP fallback translates this to `AmznSetBrightness` automatically.

Similarly, when **reading** brightness:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [milanorszagh/evon-smart-home-homeassistant-integration](https://github.com/milanorszagh/evon-smart-home-homeassistant-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
