---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP server enabling AI assistants to interact with Anki via AnkiConnect. Built with NestJS and `@rekog/mcp-nest`.

- **Package**: `@ankimcp/anki-mcp-server` (npm)
- **License**: MIT
- **Status**: Beta (0.x.x) - breaking changes allowed

## Quick Reference

```bash
# Build & Run
npm run build                    # Build → dist/ (both entry points)
npm run start:dev:stdio          # STDIO mode with watch
npm run start:dev:http           # HTTP mode with watch

# Testing
npm test                         # All tests
npm test -- path/to/file.spec.ts # Single test file
npm run test:tools               # Tool unit tests only
npm run test:workflows           # Multi-tool workflow scenarios (mocked)
npm run test:cov                 # With coverage (70% threshold)
npm run e2e:full:local           # One-shot E2E: up → test → down

# Quality
npm run lint && npm run type-check   # Pre-push checks (also runs via Husky)

# Debugging
npm run inspector:stdio          # MCP Inspector UI for testing tools
npm run inspector:stdio:debug    # With debugger on port 9229
npm run inspector:http           # Inspector against HTTP transport
```

## Architecture

### Entry Points

Two entry points compiled in a single build:

| Mode | Entry | Use Case | Logging |
|------|-------|----------|---------|
| STDIO | `dist/main-stdio.js` | Claude Desktop, MCP clients | stderr |
| HTTP | `dist/main-http.js` | Web-based AI (ChatGPT, claude.ai) | stdout |

### Core Files

```
src/
├── main-stdio.ts            # STDIO bootstrap: NestFactory.createApplicationContext()
├── main-http.ts             # HTTP bootstrap: NestFactory.create() + guards
├── app.module.ts            # Root module with forStdio()/forHttp() factories
├── bootstrap.ts             # Shared logger setup (pino → NestJS LoggerService)
├── cli.ts                   # Commander CLI (--port, --host, --anki-connect, --ngrok, --read-only)
├── anki-config.service.ts   # IAnkiConfig implementation (reads env vars via ConfigService)
├── http/guards/             # Origin validation guard (DNS rebinding protection)
├── services/ngrok.service.ts # Ngrok tunnel management (spawns ngrok, polls for URL)
└── mcp/
    ├── clients/anki-connect.client.ts  # HTTP client using ky (retries, error handling, read-only guard)
    ├── config/anki-config.interface.ts # ANKI_CONFIG injection token + IAnkiConfig interface
    ├── types/anki.types.ts             # Shared Anki types (cards, notes, ratings)
    ├── utils/                          # Shared utilities (anki.utils, markdown.utils, stats.utils)
    ├── primitives/essential/           # Core tools, prompts, resources
    └── primitives/gui/                 # GUI-specific tools (require user approval)
```

### Module System

```
AppModule.forStdio()/forHttp()
  → McpModule.forRoot()           # STDIO or STREAMABLE_HTTP transport
  → McpPrimitivesAnkiEssentialModule.forRoot()
  → McpPrimitivesAnkiGuiModule.forRoot()
```

All tools/prompts/resources are providers auto-discovered by `@rekog/mcp-nest`. MCP-Nest 1.9.0+ requires tools to also be listed in `AppModule.providers` (see `ESSENTIAL_MCP_TOOLS` and `GUI_MCP_TOOLS` arrays).

### Key Patterns

**Tool response format**: Success paths return raw objects matching the tool's `outputSchema`. The mcp-nest handler validates and wraps them automatically. Error paths use `createErrorResponse(error, context)` from `anki.utils.ts` which returns `CallToolResult` with `isError: true` and bypasses outputSchema validation.

**Action tool pattern**: Complex tools like `deckActions`, `tagActions`, `mediaActions` use a dispatch pattern — a single `@Tool` with an `action` discriminant that switches to handler functions in an `actions/` subdirectory. Each action is a pure function taking `(params, ankiClient)`.

**Read-only mode**: `AnkiConnectClient` enforces read-only mode by checking actions against a `WRITE_ACTIONS` set before sending requests. Throws `ReadOnlyModeError`. Review/scheduling operations are always allowed.

**Config system**: Two injection tokens:
- `ANKI_CONFIG` — Symbol token for AnkiConnect-specific config (IAnkiConfig interface). Provided via `useClass: AnkiConfigService` in each module's `forRoot()`. Modules can swap the config provider for testing.
- `ConfigModule` — NestJS config module reads environment variables. `AnkiConfigService` reads from it and sanitizes MCPB config values.

### Upstream AnkiConnect Quirks

These are upstream behaviors that shape tool design — surface them in tool descriptions so the AI can avoid them:

- **`updateNoteFields` silently fails** if the target note is open in Anki's Browse window. The request returns 200 but fields don't persist. Warn users in the tool description.
- **Model CSS is per-note-type, not per-note.** Use `modelStyling` to fetch CSS for a model; `notesInfo` tells you which model each note uses. `updateNoteFields` should preserve inline styles.
- **`sync` relies on the desktop app being logged into AnkiWeb.** There's no API path to authenticate — surface a helpful error hint.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ankimcp/anki-mcp-server](https://github.com/ankimcp/anki-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
