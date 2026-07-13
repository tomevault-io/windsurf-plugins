---
trigger: always_on
description: ﻿# AIRoute — Agent Guidelines
---

﻿# AIRoute — Agent Guidelines

## Project

Unified AI proxy/router — route any LLM through one endpoint. Multi-provider support
with **237 provider entries** (OpenAI, Anthropic, Gemini, DeepSeek, Groq, xAI, Mistral, Fireworks,
Cohere, NVIDIA, Cerebras, Pollinations, Puter, Cloudflare AI, HuggingFace, DeepInfra,
SambaNova, Meta Llama API, Moonshot AI, AI21 Labs, Databricks, Snowflake, and many more)
with **MCP Server** (94 tools), **A2A v0.3 Protocol**, and **Electron desktop app**.

> **Live counts (v3.8.43)**: providers 237 · MCP tools 94 · MCP scopes 30 · A2A skills 6 ·
> open-sse services 134 · routing strategies 17 · auto-combo scoring factors 12 ·
> DB modules 95 · DB migrations 110 · base tables 17 · search providers 11 ·
> i18n locales 42. **Refresh with `npm run check:docs-all`.**

## Doc Accuracy Discipline (read before writing any doc)

> **If `grep -rn "name" src/ open-sse/ bin/` returns nothing, the name does not exist. Do not document it.**

The recurring failure mode in AI-generated docs is _plausible-but-unverified specifics_.
Every claim in a `.md` file under `docs/` should be verifiable against the source.

**Rules (enforced by `npm run check:fabricated-docs`):**

1. **Never state an API name, endpoint, path, CLI command, or env var without grepping for it first.**
   ```bash
   grep -rn "theName" src/ open-sse/ bin/
   # 0 hits → do not document
   ```
2. **Never write a line count, file size, migration count, provider count, or strategy count from memory.**
   ```bash
   wc -l <file>           # exact line count
   ls <dir>/*.ts | wc -l  # file count
   ```
3. **Every code example should be copy-pasted from real usage or actually run** — not synthesized.
   Link to a real call site (`path:line`) instead of inventing a signature.
4. **Prefer citing real source (`file.ts:line`) over paraphrasing behavior** — verifiable and self-correcting.
5. **A shorter doc that is 100% accurate beats a comprehensive one with fabrications.**
   Wrong docs cost more than missing docs, because people trust and act on them.

The script `scripts/check/check-fabricated-docs.mjs` extracts every route path, env var, hook
name, function name, and file reference from `docs/**/*.md` and verifies each one against the
codebase. Run it locally before pushing docs; it runs in CI via `npm run check:docs-all`.

## Stack

- **Runtime**: Next.js 16 (App Router), Node.js `>=22.0.0 <23 || >=24.0.0 <27`, ES Modules (`"type": "module"`)
- **Language**: TypeScript 6.0 (`src/`) + JavaScript (`open-sse/`, `electron/`)
- **Database**: better-sqlite3 (SQLite) — `DATA_DIR` configurable, default `~/.AIRoute/`
- **Streaming**: SSE via `open-sse` internal workspace package
- **Styling**: Tailwind CSS v4
- **i18n**: next-intl with 42 locales (`src/i18n/messages/`) — refresh with `ls src/i18n/messages/*.json | wc -l`
- **Desktop**: Electron (cross-platform: Windows, macOS, Linux)
- **Schemas**: Zod v4 for all API / MCP input validation

---

## Build, Lint, and Test Commands

| Command                             | Description                                                        |
| ----------------------------------- | ------------------------------------------------------------------ |
| `npm run dev`                       | Start Next.js dev server                                           |
| `npm run build`                     | Production build: `next build` → `.build/next/` + assemble `dist/` |
| `npm run build:release`             | Clean rebuild + HEAD sentinel (`dist/BUILD_SHA`) — use for deploy  |
| `npm run start`                     | Run production build                                               |
| `npm run build:cli`                 | Build CLI package                                                  |
| `npm run lint`                      | ESLint on all source files                                         |
| `npm run typecheck:core`            | TypeScript core type checking                                      |
| `npm run typecheck:noimplicit:core` | Strict checking (no implicit any)                                  |
| `npm run check`                     | Run lint + test                                                    |
| `npm run check:cycles`              | Check for circular dependencies                                    |
| `npm run electron:dev`              | Run Electron app in dev mode                                       |
| `npm run electron:build`            | Build Electron app for current OS                                  |

**Build output layout:**

| Directory | Purpose                                            | Gitignored |
| --------- | -------------------------------------------------- | ---------- |
| `src/`    | Application source (TypeScript / TSX)              | No         |
| `.build/` | Build intermediates (`distDir = .build/next`)      | Yes        |
| `dist/`   | Shippable bundle assembled by `assembleStandalone` | Yes        |

The pipeline is a single `next build` pass — intermediates land in `.build/next/`, the
assembled bundle in `dist/`. VPS deploys rsync `dist/` into the remote
`/usr/lib/node_modules/AIRoute/app/` directory (VPS image path is unchanged).

### Running Tests

```bash
# All tests (unit + vitest + ecosystem + e2e)
npm run test:all


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaccen/AIRoute](https://github.com/jaccen/AIRoute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
