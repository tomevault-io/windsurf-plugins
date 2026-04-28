---
trigger: always_on
description: **Tokentop** is a terminal-based real-time token usage and cost monitoring application - "htop for AI API usage." It monitors model providers (Anthropic, OpenAI, etc.) used by coding agents (OpenCode, Claude Code, Cursor) and displays usage limits, costs, and budgets in a beautiful terminal UI.
---

# Tokentop (`ttop`) - AI Token Usage Monitor

## Project Overview

**Tokentop** is a terminal-based real-time token usage and cost monitoring application - "htop for AI API usage." It monitors model providers (Anthropic, OpenAI, etc.) used by coding agents (OpenCode, Claude Code, Cursor) and displays usage limits, costs, and budgets in a beautiful terminal UI.

**This is a public tool** intended for release to the community. Cross-platform support (macOS, Linux, Windows) is important.

## Tech Stack

- **Runtime**: Bun (required for OpenTUI native modules)
- **TUI Framework**: OpenTUI with React reconciler (`@opentui/react`)
- **Language**: TypeScript (strict mode)
- **Package Manager**: Bun

## Architecture Principles

1. **Plugin-Based Architecture**
   - Four plugin types: `provider`, `agent`, `theme`, `notification`
   - Built-in plugins for common use cases
   - npm-based distribution for community plugins (`@tokentop/*`)
   - Permission-based sandboxing for security

2. **OpenCode-First**
   - Primary focus on OpenCode users
   - Reuse OpenCode's existing auth credentials
   - Support for OpenCode's provider ecosystem

3. **Real-Time Focus**
   - API polling for provider usage data
   - Session parsing for token tracking
   - Cost estimation when actual data unavailable

4. **Modular & Extensible**
   - Each component is independently testable
   - New providers/agents can be added via plugins
   - Themes and notifications are pluggable

## Directory Structure

```
src/
├── cli.ts                    # CLI entry point (ttop binary)
├── plugins/
│   ├── types/                # Plugin interface definitions
│   │   ├── base.ts
│   │   ├── provider.ts
│   │   ├── agent.ts
│   │   ├── theme.ts
│   │   ├── notification.ts
│   │   └── index.ts
│   ├── loader.ts             # Plugin discovery & loading
│   ├── registry.ts           # Plugin management
│   ├── sandbox.ts            # Permission enforcement
│   ├── providers/            # Built-in provider plugins
│   ├── agents/               # Built-in agent plugins
│   ├── themes/               # Built-in theme plugins
│   └── notifications/        # Built-in notification plugins
├── pricing/
│   ├── estimator.ts          # Cost estimation engine
│   ├── models-dev.ts         # models.dev integration
│   └── fallback.ts           # Fallback pricing data
├── credentials/
│   ├── index.ts              # Credential discovery
│   ├── opencode.ts           # OpenCode auth reader
│   ├── env.ts                # Environment variables
│   └── external.ts           # External CLI auth files
├── tui/
│   ├── index.tsx             # TUI entry point
│   ├── App.tsx               # Main app component
│   ├── components/           # Reusable UI components
│   ├── views/                # Full-screen views
│   ├── hooks/                # React hooks
│   ├── context/              # React context providers
│   └── config/               # Settings and themes
├── sessions/                 # Coding agent session parsing
└── utils/                    # Utility functions
```

## Plugin System

### Plugin Types

| Type | Purpose | Examples |
|------|---------|----------|
| `provider` | Model provider API integration | Anthropic, OpenAI, Codex |
| `agent` | Coding agent session/auth reading | OpenCode, Claude Code, Cursor |
| `theme` | Visual themes | Dracula, Tokyo Night, Nord |
| `notification` | Alert delivery | Terminal bell, Slack, Discord |

### Plugin Permissions

All plugins must declare their permissions:

```typescript
permissions: {
  network?: { enabled: boolean; allowedDomains?: string[] };
  filesystem?: { read?: boolean; write?: boolean; paths?: string[] };
  env?: { read?: boolean; vars?: string[] };
  system?: { notifications?: boolean; clipboard?: boolean };
}
```

### Plugin Loading

Plugins are loaded from three sources (in order):

1. **Builtin plugins** — shipped with the app in `src/plugins/{providers,agents,themes,notifications}/`
2. **Local plugins** — discovered from `~/.config/tokentop/plugins/` (files and directories), plus paths from `config.plugins.local` and `--plugin` CLI flag
3. **npm plugins** — packages listed in `config.plugins.npm`

Plugins listed in `config.plugins.disabled` are removed after loading.

**CLI flag**: `ttop --plugin <path>` loads a local plugin for that run (repeatable).

**Config** (`~/.config/tokentop/config.json`):
```json
{
  "plugins": {
    "local": ["~/development/my-plugin"],
    "npm": ["tokentop-provider-replicate"],
    "disabled": ["perplexity"]
  }
}
```

**Directory-based plugins**: The loader resolves entry points by checking `package.json` main/exports, then `src/index.ts`, `index.ts`, `dist/index.js`.

### Plugin SDK

Community plugins depend on `@tokentop/plugin-sdk` for types and helpers. The SDK lives at `~/development/tokentop/plugin-sdk/`. See `docs/plugins.md` for the full guide.

### npm Plugin Naming

`@tokentop/*` is reserved for official plugins. Community plugins use the `tokentop-{type}-` prefix.

| Tier | Pattern | Example |
|------|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tokentopapp/tokentop](https://github.com/tokentopapp/tokentop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
