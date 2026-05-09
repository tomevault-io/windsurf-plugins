---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

nutrition-mcp is a Model Context Protocol (MCP) server for nutrition-related functionality, built with Bun and TypeScript. Entry point is `src/index.ts`. Server version must be updated in three places: `package.json`, `src/mcp.ts` (McpServer constructor), and `server.json`. The server icon is at `public/favicon.ico`. Tool call analytics (duration, success/failure, error category) are tracked via `src/analytics.ts` and persisted to a `tool_analytics` Supabase table.

## Commands

- `bun run src/index.ts` - Run the server
- `bun --watch src/index.ts` - Run with watch mode (restarts on file changes)
- `bun test` - Run all tests
- `bun test src/path/to/file.test.ts` - Run a single test file
- `bun run format` - Format code with Prettier (4-space indentation)

## Bun Runtime

Default to Bun for everything. Do not use Node.js equivalents.

- `bun <file>` instead of `node`/`ts-node`
- `bun install` instead of `npm install`
- `bun run <script>` instead of `npm run`
- `bunx <pkg>` instead of `npx`
- Bun auto-loads `.env` — don't use dotenv

### Preferred Bun APIs

- `Bun.serve()` for HTTP/WebSocket servers (not Express)
- `bun:sqlite` for SQLite (not better-sqlite3)
- `Bun.redis` for Redis (not ioredis)
- `Bun.sql` for Postgres (not pg/postgres.js)
- `Bun.file` for file I/O (not node:fs readFile/writeFile)
- `Bun.$\`cmd\`` for shell commands (not execa)
- Built-in `WebSocket` (not ws)

### Testing

```ts
import { test, expect } from "bun:test";
```

### Frontend (if needed)

Use HTML imports with `Bun.serve()` — not Vite. HTML files can directly import `.tsx`/`.jsx`/`.js` and Bun bundles automatically. Bun API docs: `node_modules/bun-types/docs/**.mdx`.

---

# Claude Code Operating Instructions

## Core Philosophy

Default to **parallel execution** and **web-verified information**. Sequential execution and offline assumptions are fallback modes, not defaults. When in doubt: parallelize, then search.

---

## 1. Parallelization Protocol

### Default Behavior: Parallel-First

**Before starting any multi-step task:**

1. Decompose the full task into atomic subtasks
2. Build a dependency graph — identify which subtasks have no prerequisite outputs
3. Dispatch ALL dependency-free subtasks simultaneously using parallel tool calls
4. Only after their completion, dispatch the next wave of now-unblocked subtasks
5. Repeat until task is complete

**Rule:** If two tasks do not share an input/output dependency, they MUST run in parallel. Sequential execution of independent tasks is a performance violation.

### Parallel Tool Call Patterns

Prefer batching tool calls in a single response turn rather than sequential turns:

```
# CORRECT — dispatch independent reads simultaneously
- Read file A
- Read file B
- Search web for library version
(all in one turn)

# WRONG — needless sequencing
- Read file A → wait → Read file B → wait → Search web
```

### Sub-Agent Parallelization (Task Tool)

When using the `Task` tool to spawn sub-agents:

- Spawn all independent sub-agents in a single dispatch batch
- Maximum **5 concurrent sub-agents** at any time to avoid context exhaustion
- Each sub-agent must have a clearly scoped, non-overlapping responsibility
- Define explicit output contracts for each agent before spawning
- After all agents complete, explicitly synthesize their outputs — do not present raw agent outputs as the final answer

### TodoWrite Protocol

When managing complex tasks with `TodoWrite`:

- Mark tasks as `in_progress` before starting a parallel batch
- Track each parallel thread separately
- Never mark a parent task `completed` until all parallel children resolve
- Flag dependency chains explicitly in todo descriptions

### When Sequential Execution Is Permitted

Sequential execution is only justified when:

- Task B requires Task A's output as direct input
- Tasks write to the same file or resource (race condition risk)
- A previous parallel batch returned an error that changes downstream logic
- User explicitly requests step-by-step confirmation

In all other cases: **parallelize**.

---

## 2. Web Search Mandate

### Search-First Triggers

**Always perform a web search before proceeding** when the task involves any of the following:

| Category                     | Examples                                                  |
| ---------------------------- | --------------------------------------------------------- |
| Library / framework versions | "What's the latest stable version of X?"                  |
| API behavior and signatures  | Any external SDK, REST API, or CLI tool                   |
| Security advisories          | CVEs, deprecated patterns, breaking changes               |
| Best practices               | Architecture patterns, language idioms updated post-2024  |
| Configuration options        | Tool flags, environment variables, cloud service settings |
| Error messages               | Unfamiliar stack traces, runtime errors                   |
| Compatibility questions      | Node/Python/Rust version support, browser APIs            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akutishevsky/nutrition-mcp](https://github.com/akutishevsky/nutrition-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
