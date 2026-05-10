---
trigger: always_on
description: This file is loaded automatically by AI coding assistants (Gemini, Claude, Cursor, etc.)
---

# ag-local-bridge — AI Agent Instructions

This file is loaded automatically by AI coding assistants (Gemini, Claude, Cursor, etc.)
and defines the conventions that **must be followed** when making changes to this project.

## Critical: Before Every Commit

**Always run all three checks in order before staging any changes:**

```bash
npm run format       # 1. Auto-fix formatting (Prettier)
npm run lint         # 2. Check for lint errors (ESLint)
npm test             # 3. Run all tests (node:test)
```

Or as a single pipeline:

```bash
npm run format && npm run lint && npm test
```

> **Never skip formatting.** The CI runs `format:check` and will fail if files aren't
> formatted. `npm run format` is non-destructive and safe to run at any time.

## All Available Scripts

| Command                | What it does                                                      |
| ---------------------- | ----------------------------------------------------------------- |
| `npm test`             | Run all tests with Node's built-in test runner                    |
| `npm run lint`         | ESLint over `src/` and `test/`                                    |
| `npm run lint:fix`     | ESLint with auto-fix                                              |
| `npm run format`       | Prettier write (auto-fixes all files)                             |
| `npm run format:check` | Prettier check only (what CI runs)                                |
| `npm run dev:deploy`   | Deploy source to local Antigravity extension (Windows/PowerShell) |

## Code Style

Enforced by **Prettier** (`.prettierrc`) and **ESLint** (`eslint.config.js`).

Key rules:

- Single quotes, trailing commas, semicolons
- 2-space indent, 120-char print width, LF line endings
- `no-unused-vars` → **warn** (prefix with `_` to suppress: `_unused`)
- `eqeqeq` → **error** (always use `===`)
- All files must be `'use strict';` at the top

**Unused imports in new files will produce lint warnings** — remove them before committing.

## Project Structure

Always update this structure when files are changed.

```
src/
  extension.js          # VS Code extension entry point
  context.js            # Shared bridge context object
  server.js             # HTTP server setup
  utils.js              # Shared helpers (buildStreamChunk, extractText, log, …)
  models.js             # Model enum → display name mapping + resolveModel()
  images.js             # Image extraction from request content
  workspace.js          # VS Code workspace helpers
  handlers/
    openai.js           # POST /v1/chat/completions  (OpenAI format)
    anthropic.js        # POST /v1/messages + /v1/messages/count_tokens  (Anthropic format)
    gemini.js           # POST /v1beta/models/:model:generateContent  (Gemini native format)
    models.js           # GET /v1/models
    proxy.js            # Generic proxy fallback
    debug.js            # Debug/status endpoints
  interceptors/
    http-server.js      # HTTP/1.1 interception
    https.js            # HTTPS interception
    h2.js               # HTTP/2 interception
  sidecar/
    discovery.js        # Cross-platform sidecar process discovery
    rpc.js              # HTTP/2 JSON RPC calls to sidecar
    cascade.js          # Cascade (chat) RPC helpers
    proto.js            # Binary protobuf encoding (minimal descriptors)
    raw.js              # Raw LLM inference (GetModelResponse bypass)
test/
  setup.js              # Global test setup (VS Code mock loader)
  __mocks__/vscode.js   # VS Code API mock
  *.test.js             # Tests — all use node:test (describe/it/assert)
scripts/
  dev-deploy.ps1        # Local dev deployment script (Windows)
  probe-sidecar.js      # Standalone sidecar RPC probe (dev/debug)
.agents/workflows/
  dev-deploy.md         # /dev-deploy workflow instructions
```

## Testing Conventions

- Use **Node's built-in test runner** (`node:test`) — no Jest, no Mocha.
- Import pattern:
  ```js
  const { describe, it } = require('node:test');
  const assert = require('node:assert/strict');
  ```
- Tests live in `test/*.test.js` and are auto-discovered by `npm test`.
- Mock VS Code API via `test/__mocks__/vscode.js` (loaded by `test/setup.js`).
- **Do not use `console.assert`** — use `assert` from `node:assert/strict`.

## Logging Conventions

Two logging helpers live in `src/utils.js`:

| Helper                 | When to use                                                              |
| ---------------------- | ------------------------------------------------------------------------ |
| `log(ctx, msg)`        | Essential operational info (cascade created, request dispatched, errors) |
| `verboseLog(ctx, msg)` | Payloads, polling ticks, workspace switches, debug detail                |

`verboseLog` is **gated behind the `agLocalBridge.logRequests` setting** — it produces
no output unless the user explicitly opts in.

Key rules:

- **Never write logs to files** (`fs.appendFileSync`, `fs.writeFileSync` for logs) — all
  logging goes through the VS Code Output channel only.
- **Never use `console.log`** — use `log()` or `verboseLog()`.
- **Never dump full request/response bodies** in `log()` — use `verboseLog()` for those.

## Architecture Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcodiniz/ag-local-bridge](https://github.com/marcodiniz/ag-local-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
