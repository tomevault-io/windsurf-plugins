---
trigger: always_on
description: Unified AI proxy/router — route any LLM through one endpoint. Multi-provider support
---

# Routiform — Agent Guidelines

## Project

Unified AI proxy/router — route any LLM through one endpoint. Multi-provider support
with **60+ providers** (OpenAI, Anthropic, Gemini, DeepSeek, Groq, xAI, Mistral, Fireworks,
Cohere, NVIDIA, Cerebras, Pollinations, Puter, Cloudflare AI, HuggingFace, and many more)
with **MCP Server** (25 tools) and **A2A v0.3 Protocol**.

## Stack

- **Runtime**: Next.js 16 (App Router), Node.js ≥18 <25, ES Modules (`"type": "module"`)
- **Language**: TypeScript 5.9 (`src/`) + JavaScript (`open-sse/`)
- **Database**: better-sqlite3 (SQLite) — `DATA_DIR` configurable, default `~/.routiform/` — **backup/restore: copy whole `DATA_DIR` (includes `storage.sqlite` + `server.env` with `STORAGE_ENCRYPTION_KEY`); copying only the `.sqlite` file breaks encrypted API keys** — see `docs/BACKUP_AND_RESTORE.md`
- **Streaming**: SSE via `open-sse` internal workspace package
- **Styling**: Tailwind CSS v4
- **i18n**: next-intl with 30 languages
- **Schemas**: Zod v4 for all API / MCP input validation

---

## Build, Lint, and Test Commands

| Command                             | Description                       |
| ----------------------------------- | --------------------------------- |
| `npm run dev`                       | Start Next.js dev server          |
| `npm run build`                     | Production build (isolated)       |
| `npm run start`                     | Run production build              |
| `npm run build:cli`                 | Build CLI package                 |
| `npm run lint`                      | ESLint on all source files        |
| `npm run typecheck:core`            | TypeScript core type checking     |
| `npm run typecheck:noimplicit:core` | Strict checking (no implicit any) |
| `npm run check`                     | Run lint + test                   |
| `npm run check:cycles`              | Check for circular dependencies   |

### Running Tests

```bash
# All tests (unit + vitest + ecosystem + e2e)
npm run test:all

# Single test file (Node.js native test runner — most tests use this)
node --import tsx/esm --test tests/unit/your-file.test.mjs
node --import tsx/esm --test tests/unit/plan3-p0.test.mjs
node --import tsx/esm --test tests/unit/fixes-p1.test.mjs
node --import tsx/esm --test tests/unit/security-fase01.test.mjs

# Integration tests
node --import tsx/esm --test tests/integration/*.test.mjs

# Vitest (MCP server, autoCombo)
npm run test:vitest

# E2E with Playwright
npm run test:e2e

# Protocol clients E2E (MCP transports, A2A)
npm run test:protocols:e2e

# Ecosystem compatibility tests
npm run test:ecosystem

# Coverage (55% min thresholds — statements, lines, functions; 60% branches)
npm run test:coverage
```

---

## Code Style Guidelines

### Formatting (Prettier — enforced via lint-staged)

2 spaces · semicolons required · double quotes (`"`) · 100 char width · es5 trailing commas.
Always run `prettier --write` on changed files.

### TypeScript

- **Target**: ES2022 · **Module**: `esnext` · **Resolution**: `bundler`
- `strict: false` — prefer explicit types, don't rely on inference
- Path aliases: `@/*` → `src/`, `@routiform/open-sse` → `open-sse/`, `@routiform/open-sse/*` → `open-sse/*`

### ESLint Rules

- **Security (error, everywhere)**: `no-eval`, `no-implied-eval`, `no-new-func`
- **Relaxed in `open-sse/` and `tests/`**: `@typescript-eslint/no-explicit-any` = warn
- React hooks rules and `@next/next/no-assign-module-variable` disabled in `open-sse/` and `tests/`

### Naming

| Element             | Convention                       | Example                              |
| ------------------- | -------------------------------- | ------------------------------------ |
| Files               | camelCase / kebab-case           | `chatCore.ts`, `tokenHealthCheck.ts` |
| React components    | PascalCase                       | `Dashboard.tsx`, `ProviderCard.tsx`  |
| Functions/variables | camelCase                        | `getHealth()`, `switchCombo()`       |
| Constants           | UPPER_SNAKE                      | `MAX_RETRIES`, `DEFAULT_TIMEOUT`     |
| Interfaces          | PascalCase (`I` prefix optional) | `ProviderConfig`                     |
| Enums               | PascalCase (members too)         | `LogLevel.Error`                     |

### Imports

- **Order**: external → internal (`@/`, `@routiform/open-sse`) → relative (`./`, `../`)
- **No barrel imports** from `localDb.ts` — import from the specific `db/` module instead

### Error Handling

- try/catch with specific error types; always log with context (pino logger)
- Never silently swallow errors in SSE streams — use abort signals for cleanup
- Return proper HTTP status codes (4xx client, 5xx server)

### Security

- **NEVER** commit API keys, secrets, or credentials
- Validate all user inputs with Zod schemas
- Auth middleware required on all API routes
- Never log SQLite encryption keys
- Sanitize user content (dompurify for HTML)

---

## Architecture

### Data Layer (`src/lib/db/`)

All persistence uses SQLite through domain-specific modules:
`core.ts`, `providers.ts`, `models.ts`, `combos.ts`, `apiKeys.ts`, `settings.ts`,
`backup.ts`, `proxies.ts`, `prompts.ts`, `webhooks.ts`, `detailedLogs.ts`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linhnguyen-gt/Routiform](https://github.com/linhnguyen-gt/Routiform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
