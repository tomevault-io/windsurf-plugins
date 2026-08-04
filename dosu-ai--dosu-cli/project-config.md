---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dosu CLI (`@dosu/cli`) — a CLI tool that manages MCP (Model Context Protocol) server integrations for AI tools (Claude Code, Cursor, VS Code, Windsurf, Zed, etc.). It authenticates users via browser-based OAuth against Supabase, selects a Dosu deployment, and writes MCP server entries into each AI tool's config file.

## Commands

```bash
bun install                     # Install dependencies
bun run dev                     # Run CLI from source (loads .env.development; Bun's NODE_ENV default)
bun run dev:local               # Run CLI from source (local dev endpoints, DOSU_DEV=true)
bun run build                   # Compile to single binary via bun build --compile
bun run build:npm               # Bundle for npm distribution (bin/dosu.js)
bun run build:all               # Cross-platform build matrix

bun run test                    # Run all tests (vitest, forks pool)
bun run test:watch              # Run tests in watch mode
bunx vitest run src/config      # Run tests for a single module
bunx vitest run src/auth/flow   # Run a specific test file

bun run typecheck               # TypeScript type checking (tsc --noEmit)
bun run lint                    # Lint with Biome
bun run format                  # Format with Biome
bun run check                   # Biome lint + format check (used in CI)
```

## Architecture

**Entry point:** `src/index.ts` → `src/cli/cli.ts` (Commander program)

Running `dosu` with no args launches the interactive TUI (`src/tui/tui.ts`). Two broad families of subcommands are registered in `src/cli/cli.ts`:

- **Local / MCP management** — `login`, `logout`, `status`, `setup`, `mcp add|list`, `logs`.
- **Dosu platform** (require an authenticated deployment) — `ask`, `knowledge`, `docs`, `suggest`, `threads`, `review`, `sources`, `integrations`, `tags`, `members`, `org`, `deployments`, `analytics`, `insights`, `skill`, `hooks`. Each lives in `src/commands/<name>.ts` and talks to the backend via `src/client/`.

Key modules:

- **`src/auth/`** — Browser-based OAuth flow. Starts a local HTTP server on a random port, opens the browser to the Dosu web app, receives the token via redirect callback. `ticket.ts` backs the agent ticket flow (`login --request`/`--check`).
- **`src/client/`** — Authenticated HTTP client for the Dosu backend API. Handles token refresh (Supabase `/auth/v1/token`) and auto-retry on 401/403.
- **`src/config/`** — CLI's own config (`~/.config/dosu-cli/config.json`). Stores access/refresh tokens, deployment ID, API key. Supports OSS vs Cloud `mode`. `constants.ts` has env-aware URL getters (dev vs prod, overridable via env vars).
- **`src/mcp/`** — Provider system for AI tool integrations. Each provider in `providers/` knows how to detect, install, and remove the Dosu MCP entry from that tool's config file. `base.ts` provides `createJSONProvider()` — a factory that covers most tools with just config path + top-level key. `config-helpers.ts` handles JSON/JSONC read/write. `detect.ts` handles path expansion and platform-aware detection.
- **`src/commands/`** — The Dosu platform command layer (the list above). Thin Commander wrappers over `src/client/` calls; `output.ts` standardizes human vs JSON output.
- **`src/setup/`** — Interactive setup wizard (authenticate → select org → select deployment → mint API key → detect installed tools → configure). Uses `@clack/prompts`.
- **`src/agent/`** — Non-interactive setup for coding agents (`setup --agent --tool <id>`) and the ticket-based login commands (`login --request`/`--check`). Emits machine-readable JSON via `output.ts` for agent consumption.
- **`src/hooks/`** — Coding-agent hook entrypoints invoked as `dosu hooks <user-prompt-submit|post-tool-use|stop>`. They mint, poll, and inject Dosu "knowledge tickets" into an agent's turn. These run on the agent's hot path on every turn, so they must stay fast and keep stdout clean — `src/cli/cli.ts` deliberately skips the update checks for them.
- **`src/tui/`** — Main menu TUI when running `dosu` with no subcommand.
- **`src/version/`** — Version string from build-time env vars (`DOSU_VERSION`, `DOSU_COMMIT`, `DOSU_DATE`), plus background update checks (`update-check.ts`, `skill-update-check.ts`).

## CLI Contract Discipline

All tRPC calls MUST be typed through the generated contract (`CliApiClient` / `TypedClient` from `src/generated/dosu-api-types.d.ts`). Never hand-write client shapes — no `TypedClient & {...}` intersections and no local `{ query(input: ...) }` / `{ mutate(input: ...) }` interfaces. Hand shapes compile against procedures the backend may not serve, turning a missing route into a production 404 instead of a type error (how `dosu review` broke in v0.29.0). If a procedure is missing from the contract, register it in `cliRouter` (dosu repo, `frontend/packages/api/src/cli-root.ts`), regenerate, and re-vendor the contract. Enforced by `src/client/contract-discipline.test.ts`.

## Testing

- Tests live alongside source files as `*.test.ts`
- Vitest with `pool: "forks"` (not threads) — required for tests that mock `node:fs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dosu-ai/dosu-cli](https://github.com/dosu-ai/dosu-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
