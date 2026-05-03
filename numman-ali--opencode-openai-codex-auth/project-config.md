---
trigger: always_on
description: This file provides coding guidance for AI agents (including Claude Code, Codex, and others) when working with code in this repository.
---

# AGENTS.md

This file provides coding guidance for AI agents (including Claude Code, Codex, and others) when working with code in this repository.

## Overview

This is an **opencode plugin** that enables OAuth authentication with OpenAI's ChatGPT Plus/Pro Codex backend. It allows users to access `gpt-5.2-codex`, `gpt-5.1-codex`, `gpt-5.1-codex-max`, `gpt-5.1-codex-mini`, `gpt-5.2`, and `gpt-5.1` models through their ChatGPT subscription instead of using OpenAI Platform API credits. Legacy GPT-5.0 models are automatically normalized to their GPT-5.1 equivalents.

**Key architecture principle**: 7-step fetch flow that intercepts opencode's OpenAI SDK requests, transforms them for the ChatGPT backend API, and handles OAuth token management.

## Build & Test Commands

```bash
# Build (compiles TypeScript + copies HTML file)
npm run build

# Type checking only (no build)
npm run typecheck

# Run all tests
npm test

# Watch mode for TDD
npm run test:watch

# Interactive test UI
npm run test:ui

# Coverage report
npm run test:coverage
```

**Important**: The build script has a critical step that copies `lib/oauth-success.html` to `dist/lib/`. This HTML file is required for the OAuth callback flow.

## Code Architecture

### Plugin Flow (index.ts)

The main entry point orchestrates a **7-step fetch flow**:

1. **Token Management**: Check token expiration, refresh if needed
2. **URL Rewriting**: Transform OpenAI Platform API URLs → ChatGPT backend API (`https://chatgpt.com/backend-api/codex/responses`)
3. **Request Transformation**:
   - Normalize model names (all variants → `gpt-5.2`, `gpt-5.2-codex`, `gpt-5.1`, `gpt-5.1-codex`, `gpt-5.1-codex-max`, `gpt-5.1-codex-mini`, `gpt-5`, `gpt-5-codex`, or `codex-mini-latest`)
   - Inject Codex system instructions from latest GitHub release
   - Apply reasoning configuration (effort, summary, verbosity)
   - Add CODEX_MODE bridge prompt (default) or tool remap message (legacy)
   - Filter OpenCode system prompts when in CODEX_MODE
   - Filter conversation history (remove `rs_*` IDs for stateless operation)
4. **Headers**: Add OAuth token + ChatGPT account ID
5. **Request Execution**: Send to Codex backend
6. **Response Logging**: Optional debug logging (ENABLE_PLUGIN_REQUEST_LOGGING=1)
7. **Response Handling**: Convert SSE to JSON (non-tool requests) or pass through

### Module Organization

**Core Plugin** (`index.ts`)
- Plugin definition and main fetch orchestration
- OAuth loader (extracts ChatGPT account ID from JWT)
- Configuration loading and CODEX_MODE determination

**Authentication** (`lib/auth/`)
- `auth.ts`: OAuth flow (PKCE, token exchange, JWT decoding, refresh)
- `server.ts`: Local HTTP server for OAuth callback (port 1455)
- `browser.ts`: Platform-specific browser opening

**Request Handling** (`lib/request/`)
- `fetch-helpers.ts`: 10 focused helper functions for main fetch flow
- `request-transformer.ts`: Body transformations (model normalization, reasoning config, input filtering)
- `response-handler.ts`: SSE to JSON conversion

**Prompts** (`lib/prompts/`)
- `codex.ts`: Fetches Codex instructions from GitHub (ETag-cached), tool remap message
- `codex-opencode-bridge.ts`: CODEX_MODE bridge prompt for CLI parity

**Configuration** (`lib/`)
- `config.ts`: Plugin config loading, CODEX_MODE determination
- `constants.ts`: All magic values, URLs, error messages
- `types.ts`: TypeScript type definitions
- `logger.ts`: Debug logging (controlled by env var)

### Key Design Patterns

**1. Stateless Operation**: Uses `store: false` + `include: ["reasoning.encrypted_content"]`
- Allows multi-turn conversations without server-side storage
- Encrypted reasoning content persists context across turns

**2. CODEX_MODE** (enabled by default):
- **Priority**: `CODEX_MODE` env var > `~/.opencode/openai-codex-auth-config.json` > default (true)
- When enabled: Filters out OpenCode system prompts, adds Codex-OpenCode bridge prompt with Task tool & MCP awareness
- When disabled: Uses legacy tool remap message
- Bridge prompt (~550 tokens): Tool mappings, available tools, working style, **Task tool/sub-agent awareness**, **MCP tool awareness**
- **Prompt verification**: Caches OpenCode's codex.txt from GitHub (ETag-based) to verify exact prompt removal, with fallback to text signature matching

**3. Configuration Merging**:
- Global options (`provider.openai.options`) + per-model options (`provider.openai.models[name].options`)
- Model-specific options override global
- Plugin defaults: `reasoningEffort: "medium"`, `reasoningSummary: "auto"`, `textVerbosity: "medium"`

**4. Model Normalization** (GPT-5.0 → GPT-5.1 migration):
- All `gpt-5.2-codex*` variants → `gpt-5.2-codex` (newest Codex model, supports xhigh)
- All `gpt-5.1-codex-max*` variants → `gpt-5.1-codex-max`
- All `gpt-5.1-codex*` variants → `gpt-5.1-codex`
- All `gpt-5.1-codex-mini*` variants → `gpt-5.1-codex-mini`
- All `gpt-5.2` variants → `gpt-5.2`
- All `gpt-5.1` variants → `gpt-5.1`
- **Legacy mappings** (GPT-5.0 being phased out):
  - `gpt-5-codex*` variants → `gpt-5.1-codex`
  - `gpt-5-codex-mini*` or `codex-mini-latest` → `gpt-5.1-codex-mini`
  - `gpt-5*` variants (including `gpt-5-mini`, `gpt-5-nano`) → `gpt-5.1`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numman-ali/opencode-openai-codex-auth](https://github.com/numman-ali/opencode-openai-codex-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
