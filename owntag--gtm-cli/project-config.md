---
trigger: always_on
description: **GTM CLI** is a command-line interface for Google Tag Manager, built with Deno and Cliffy. It provides full access to the GTM API v2, enabling users to manage all GTM resources from the terminal.
---

# CLAUDE.md - GTM CLI Project Documentation

## Project Overview

**GTM CLI** is a command-line interface for Google Tag Manager, built with Deno and Cliffy. It provides full access to the GTM API v2, enabling users to manage all GTM resources from the terminal.

**Maintained by**: owntag GmbH

**Supported Platforms**: macOS, Linux (no Windows support)

## Architecture

### Runtime Environment
- **Deno** - TypeScript/JavaScript runtime
- **Cliffy** - CLI framework (command parsing, prompts, tables)
- **Google APIs** - Official googleapis npm package for GTM API

### Key Technologies
- `@cliffy/command` - Command-line parsing and subcommands
- `@cliffy/prompt` - Interactive prompts
- `@cliffy/table` - Table output formatting
- `@cliffy/ansi` - Colored terminal output
- `googleapis` - Google Tag Manager API v2 client

## Project Structure

```
src/
├── main.ts                 # CLI entry point, command registration
├── auth/
│   ├── mod.ts             # Auth module exports
│   ├── credentials.ts     # Local credential storage
│   ├── oauth.ts           # OAuth 2.0 flow implementation
│   └── service-account.ts # Service account auth
├── api/
│   ├── mod.ts             # API module exports
│   └── client.ts          # GTM API client wrapper, type exports
├── commands/
│   ├── mod.ts             # Command exports
│   ├── auth.ts            # Login, logout, status
│   ├── config.ts          # Configuration management
│   ├── upgrade.ts         # Self-update command
│   ├── accounts.ts        # Account operations
│   ├── containers.ts      # Container CRUD
│   ├── workspaces.ts      # Workspace management
│   ├── tags.ts            # Tag CRUD + revert
│   ├── triggers.ts        # Trigger CRUD + revert
│   ├── variables.ts       # Variable CRUD + revert
│   ├── folders.ts         # Folder management
│   ├── versions.ts        # Version operations
│   ├── version-headers.ts # Version listing (lightweight)
│   ├── environments.ts    # Environment management
│   ├── built-in-variables.ts # Built-in variable enable/disable
│   ├── clients.ts         # sGTM client operations
│   ├── templates.ts       # Custom template management
│   ├── transformations.ts # Transformation operations
│   ├── zones.ts           # Zone operations
│   ├── destinations.ts    # Destination operations
│   ├── gtag-configs.ts    # Gtag config operations
│   └── user-permissions.ts # User permission management
├── config/
│   ├── mod.ts             # Config module exports
│   ├── constants.ts       # App constants, OAuth config
│   └── store.ts           # User config persistence
└── utils/
    ├── mod.ts             # Utils module exports
    ├── output.ts          # Output formatting (JSON, table, compact)
    ├── errors.ts          # Error handling utilities
    ├── pagination.ts      # Pagination helpers
    ├── banner.ts          # ASCII banner display
    └── update-checker.ts  # Version checking and self-update
```

## Development Commands

```bash
deno task dev        # Run with watch mode
deno task start      # Run the CLI
deno task compile    # Build binary for current platform
deno task lint       # Run Deno linter
deno task fmt        # Format code
deno task check      # Type check
```

## Authentication Flow

1. User runs `gtm auth login`
2. CLI starts local HTTP server on port 8085
3. Browser opens to Google OAuth consent screen
4. User authenticates and grants permissions
5. Google redirects to `localhost:8085/callback` with auth code
6. CLI exchanges code for access token and refresh token
7. Tokens stored in `~/.config/gtm-cli/credentials.json`
8. Subsequent commands use stored tokens (auto-refresh when expired)

### Alternative Auth Method
- **Service Account**: `gtm auth login --service-account /path/to/key.json`

## Configuration Files

| File | Location | Purpose |
|------|----------|---------|
| `credentials.json` | `~/.config/gtm-cli/` | OAuth tokens (access, refresh) |
| `config.json` | `~/.config/gtm-cli/` | User preferences, defaults |
| `auth-method.json` | `~/.config/gtm-cli/` | Selected auth method (OAuth/SA) |
| `update-check.json` | `~/.config/gtm-cli/` | Last update check time & version |

## Available Commands

### Core Commands
- `gtm auth` - Authentication (login, logout, status)
- `gtm config` - Configuration (get, set, unset, setup)
- `gtm upgrade` - Self-update to latest version
- `gtm accounts` - Account management
- `gtm containers` - Container CRUD
- `gtm workspaces` - Workspace management

### Resource Commands (Tags, Triggers, Variables)
All support: list, get, create, update, delete, revert

### Version Commands
- `gtm versions` - Create, get, publish, update, delete versions
- `gtm version-headers` - List version headers (lightweight)

### Additional Commands
- `gtm folders` - Folder management
- `gtm environments` - Environment management
- `gtm built-in-variables` - Built-in variable enable/disable

### Server-Side GTM Commands
- `gtm clients` - sGTM client management
- `gtm templates` - Custom template management
- `gtm transformations` - Transformation operations
- `gtm zones` - Zone management
- `gtm destinations` - Destination linking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owntag/gtm-cli](https://github.com/owntag/gtm-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
