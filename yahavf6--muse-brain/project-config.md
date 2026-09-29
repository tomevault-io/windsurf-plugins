---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Muse Brain: a typed knowledge graph (thought/action/rule/conclusion nodes, typed edges) on SQLite, served over MCP + a small JSON API to every coding agent on the machine (Claude Code, Codex, Cursor, Gemini CLI, Claude Desktop). Full thesis and comparison to memory-store approaches: `README.md`.

## Commands

```bash
npm start                                          # runs src/server.ts directly on :4747, no build step
npm test                                            # node:test, all of test/*.test.ts
node --no-warnings=ExperimentalWarning --test test/brain.test.ts        # one file
node --no-warnings=ExperimentalWarning --test --test-name-pattern="guard" test/*.test.ts   # by name
bash hooks/brain-hook.sh --selftest                # hook fixtures (start/pre/mark/stop) -- separate from npm test, also run in CI
npm run seed                                        # demo graph, project: 'demo'; refuses a non-empty DB without --force
npm run bench                                       # synthetic-graph numbers via the real log() verb; BENCH_SIZES=1000000 for the 1M run
npm run token -- mint <agent> [--read-only]         # bearer tokens for the public listener; also `list`, `revoke <hash>`; needs no running server
```

No build step, no lint/format config in the repo. Node's native TypeScript type-stripping runs `.ts` files directly (requires Node >= 24). `scripts/service.sh {install,uninstall,restart,status,render}` manages the macOS launchd background service.

## Architecture

**One process, three surfaces, one `callVerb()` path.** `src/server.ts` is the only entry point: a raw `node:http` server that serves the MCP endpoint (`/mcp`), a JSON API (`/api/*`) used by the 3D graph page in `public/`, and static files. Every mutation and query -- whether it arrived as an MCP tool call or a `POST /api/call` -- funnels through `callVerb()` in `src/verbs.ts`, which zod-validates args and dispatches to one of the 12 verbs in the `VERBS` table: 8 registered as MCP tools (`ask`, `search`, `context`, `get`, `log`, `link`, `update`, `approve_rule`) and 4 UI-only (`delete_node`, `delete_edge`, `agent_policies`, `set_agent_policy`), flagged `ui_only: true` in that table and skipped when `server.ts` registers MCP tools. Read `src/verbs.ts` top to bottom to see the whole write/read surface; there is no other business-logic layer.

**Two scopes gate what a caller can do.** `Who = { agent, scope: 'full' | 'admin' }`. MCP callers (any agent) always get `scope: 'full'`, set in `whoIs()` from the `?agent=` query param on the loopback listener, or from a bearer token on the public one (see "Public listener"). `POST /api/call` (the local UI only) always gets `scope: 'admin'`, and `callVerb()` itself refuses every `ui_only` verb for any non-admin caller. `update()` in `verbs.ts` is where this matters most: `full` cannot touch an approved rule's fields, cannot set a conclusion's verdict, cannot edit a guard, cannot revive a retired node -- `admin` can. This is the only privilege boundary in the system; the loopback listener has no auth beyond loopback + `Host`/Origin validation, and the public listener adds only bearer tokens on `/mcp` and `/api/v1/*` (see "Status and limits" in README.md and `SECURITY.md`).

**Per-agent access policy narrows `scope: 'full'` further than the two scopes above, and sits in a separate table.** `agent_policy` (`src/schema.sql`) holds one optional row per agent: `can_read`, `can_write`, and `projects` (`NULL` = every project, a JSON array = only those; a node with no `project`, company-wide, is always readable regardless and never writable by a scoped agent). `whoIs()` sets `Who.read`/`write`/`projects` from that row when one exists; no row leaves them `undefined`, which is unrestricted -- every install before this table existed, and every agent nobody has scoped since, behaves exactly as before. The single enforcement point is `callVerb()`: for `scope: 'full'` it refuses a verb whose `access` (`'read'` or `'write'`, a field on each `Verb`) doesn't match, and read/write handlers apply `scopeClause()`/`assertWriteProject()` against `who.projects`; `scope: 'admin'` drops `read`/`write`/`projects` before any of this runs, so it bypasses agent policy the same way it bypasses every other `full`-only refusal. `mcpServerFor(who)` in `server.ts` (a fresh `McpServer` per `/mcp` request) mirrors this by only registering the tools that caller's policy allows -- cosmetic on top of `callVerb()`'s real check, not a second one. Two `ui_only` verbs manage this table: `agent_policies` (list) and `set_agent_policy` (upsert), reachable only via `POST /api/call`. Identity is unchanged underneath all of this: on the loopback listener still just the self-declared `?agent=` query param, so this policy is a well-behaved-agent guardrail, not a security boundary.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yahavf6/muse-brain](https://github.com/yahavf6/muse-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
