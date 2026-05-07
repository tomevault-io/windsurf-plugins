---
trigger: always_on
description: Guidelines for AI coding agents and developers working in this repository.
---

# AGENTS.md

Guidelines for AI coding agents and developers working in this repository.

## Repository Reality

Synergy is an open-source AI agent platform built as a Bun monorepo with TypeScript ESM modules.

This repository has evolved substantially. Do not assume older README text, old blog-style architecture notes, or legacy naming still reflect the current system. Before changing code or docs, verify the current implementation.

Synergy is an AI agent platform with multiple product surfaces. The current product surface includes:

- a stateless server runtime
- a Web client
- one-off CLI execution via `send`
- configurable agents and subagents
- session persistence
- MCP integration
- channels and Holos-related identity flows
- agenda and automation features
- note, memory/engram, and community-facing capabilities

## Current Architecture Vocabulary

Use current terms consistently.

- Prefer `Scope` over older `Project`-centric descriptions when referring to current scope resolution and workspace context.
- Prefer current `engram` terminology for the knowledge/memory subsystem rather than older historical naming.
- Prefer current session management terminology and current CLI command names.
- Do not reintroduce old names in new docs unless you are explicitly documenting migration history.

## Monorepo Map

### Primary packages

- `packages/synergy` — core runtime, server, CLI, sessions, tools, permissions, integrations, orchestration
- `packages/app` — main Web application
- `packages/config-ui` — dedicated configuration UI package
- `packages/plugin` — plugin SDK published as `@ericsanchezok/synergy-plugin`
- `packages/sdk/js` — TypeScript SDK published as `@ericsanchezok/synergy-sdk`
- `packages/ui` — shared UI component library
- `packages/util` — shared utilities and error helpers
- `packages/script` — build and release utilities

### Important areas in `packages/synergy/src`

Current work commonly touches these domains:

- `agent/` — built-in agent definitions and prompts
- `agenda/` — scheduling and autonomous task execution
- `bus/` — eventing
- `channel/` — external messaging/channel integrations
- `cli/` — CLI commands, startup flows, and user-facing entrypoints
- `config/` — config loading, merging, resolution, setup
- `cortex/` — task orchestration and background execution
- `engram/` — memory/knowledge infrastructure
- `mcp/` — MCP support
- `note/` — notes system
- `permission/` — permission model
- `process/` and `pty/` — process/runtime plumbing
- `provider/` — LLM provider integration
- `scope/` — scope resolution and context
- `server/` — HTTP server and API routes
- `session/` — session lifecycle, prompting, recall, summaries, progress
- `skill/` — skill loading and built-ins
- `tool/` — tool implementations

If you touch files in one area, scan adjacent domains before assuming the abstraction boundary.

## Core Runtime Model

Synergy uses a client-server model.

Key practical consequence:

- the server is central and stateless relative to a single project directory
- clients attach to it and provide a working directory or scope context
- many CLI flows are built around `server` first, then `web` or `send`

Do not write docs or code comments that assume the old "single local CLI process bound to one directory" mental model.

## Development Commands

### Primary development flow

Build the frontend first (required before the server can serve the Web UI):

```bash
bun run --cwd packages/app build
```

Then start the server:

```bash
bun dev
```

This starts the server from `packages/synergy` and preserves the invoking directory via `SYNERGY_CWD`.

Connect clients from another terminal:

```bash
bun dev web --dev
bun dev send "your message here"
```

### Type checking and formatting

```bash
bun run typecheck
./script/format.ts
```

### Tests

Run tests from `packages/synergy`, not from repo root:

```bash
cd packages/synergy
bun test
bun test test/tool/read.test.ts
bun test --watch
```

### Build and SDK generation

```bash
./packages/synergy/script/build.ts --single
./script/generate.ts
```

Regenerate the SDK after modifying server routes or route schemas.

## Code Style

### General principles

- Fix root causes, not just symptoms.
- Prefer minimal, focused changes.
- Match the surrounding style.
- Do not add inline comments unless explicitly requested.
- Do not add copyright or license headers.
- Do not introduce unrelated cleanup while working on a task.

### Module organization

Use namespace-based organization where that is the established local pattern.

```ts
export namespace Tool {
  export function define(...) {}
  export interface Info {}
}
```

Prefer extending existing patterns over introducing a parallel style.

### Imports

- `@/` aliases and relative imports are both used
- named imports are preferred where appropriate
- import `z` from `"zod"` as default

```ts
import z from "zod"
```

### Types and validation

- use Zod for runtime validation
- add `.meta({ ref: "TypeName" })` for API-exposed schemas where needed
- infer TypeScript types from schemas
- avoid `any`

### Variables and control flow

- prefer `const`
- use early returns to reduce nesting
- avoid unnecessary destructuring when it harms clarity or context

### Error handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SII-Holos/synergy](https://github.com/SII-Holos/synergy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
