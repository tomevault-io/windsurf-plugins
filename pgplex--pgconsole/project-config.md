---
trigger: always_on
description: pgconsole is a PostgreSQL management console with a React frontend and Express backend using ConnectRPC for API communication.
---

# Project Guidelines

## Overview

pgconsole is a PostgreSQL management console with a React frontend and Express backend using ConnectRPC for API communication.

## Working Guidelines

### Plan Mode

When using plan mode, create the plan under `plans/` directory.

### Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

### SQL Analysis: Use the Parser, Not Regex

**NEVER use regex to parse or analyze SQL. Always use `@libpg-query/parser`.**

The codebase has a full SQL parser in `src/lib/sql/`:
- `core.ts` - `parseSql()` returns typed AST with statement kinds and expressions
- `server/lib/sql-permissions.ts` - Example of proper parser usage

```typescript
// ❌ WRONG - Don't do this
const isSelect = sql.trim().toUpperCase().startsWith('SELECT')
const hasFunction = /pg_terminate_backend/i.test(sql)

// ✅ CORRECT - Use the parser
import { parseSql } from 'src/lib/sql/core'
const parsed = await parseSql(sql)
const stmt = parsed.statements[0]
if (stmt.kind === 'select') { /* ... */ }
```

The parser handles:
- Comments, string literals, quoted identifiers
- Multi-statement SQL
- Nested expressions and subqueries
- All PostgreSQL syntax

## Project Structure

```
/server              - Express backend with ConnectRPC services
  /lib               - Config, auth utilities
  /services          - RPC service implementations
/src                 - React frontend application
  /components        - React components
    /ui              - Reusable UI primitives (95+ components)
    /sql-editor      - SQL editor feature components
  /hooks             - Custom React hooks
  /lib               - Shared utilities
    /sql             - SQL parsing, formatting, autocomplete
      /autocomplete  - Autocomplete pipeline (8 modules)
  /gen               - Generated protobuf code
  /pages             - Page components
/proto               - Protocol buffer definitions
/scripts             - Build scripts
/tests               - Test suite (SQL parser, autocomplete, API)
/docs                - Documentation and design plans
/dist/client         - Built frontend (Vite output)
/dist/server.mjs     - Built server (esbuild output)
```

## Configuration

All configuration is done via `pgconsole.toml` (no environment variables required). See `docs/configuration/config.mdx` for the full reference. Source of truth for config fields: `server/lib/config.ts`.

### Provider Config Pattern

Both auth and AI providers use `[[array]]` of tables with a type/vendor discriminator:

```toml pgconsole.toml
# Auth providers — type discriminator, issuer_url required for keycloak/okta
[[auth.providers]]
type = "google"
client_id = "..."
client_secret = "..."

[[auth.providers]]
type = "keycloak"
client_id = "..."
client_secret = "..."
issuer_url = "https://keycloak.example.com/realms/myrealm"

# AI providers — vendor discriminator
[[ai.providers]]
id = "gpt4"
vendor = "openai"
model = "gpt-4o"
api_key = "sk-..."
```

**Types:** `AuthProviderConfig` (with `type` field) and `AIProviderConfig` (with `vendor` field) in `server/lib/config.ts`.

**Access:** Use `getAuthProvider(type)` to look up an auth provider by type. In OAuth route files, import `getAuthProvider` (not `getAuthConfig`).

### Config Docs Convention (`docs/configuration/config.mdx`)

When adding or updating config fields:
- **Section titles**: Human-readable (e.g., "Connections" not "`[[connections]]`")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgplex/pgconsole](https://github.com/pgplex/pgconsole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
