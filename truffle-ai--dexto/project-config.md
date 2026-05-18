---
trigger: always_on
description: This repo is reviewed by automated agents (including CodeRabbit). This file is the source of truth for repo-wide conventions and review expectations.
---

# Dexto Development Guidelines for AI Assistants

This repo is reviewed by automated agents (including CodeRabbit). This file is the source of truth for repo-wide conventions and review expectations.

**Package manager: pnpm** (do not use npm/yarn)

## Code Quality Requirements

Before completing significant tasks, run:

```bash
/quality-checks
```

This runs `scripts/quality-checks.sh` for build, tests, lint, typecheck, and Hono client inference. See `.claude/commands/quality-checks.md`.
Do not ask the user whether to run quality checks first; run them proactively.

## General Rules

- Optimize for correctness. Use facts and code as the source of truth.
- Read relevant code before recommending changes. Prefer grep/glob + direct file references over assumptions.
- If something requires assumptions, state them and ask for confirmation.
- Don't communicate to the user via code comments. Comments are for future readers of the code, not for explaining decisions to the user.
- Keep npm/pnpm scripts cross-platform (Unix + Windows):
    - In `package.json` scripts, avoid single-quoted CLI arguments like `--filter='...'` or `--exclude '...'` because Windows `cmd.exe` treats single quotes as literal characters.
    - Prefer unquoted `--flag=value` when possible, or escaped double quotes (`\"...\"`) for glob patterns.

## Stack Rules (important)

These rules are intended to prevent stack fragmentation and review churn.

### WebUI (`packages/webui`)

- Build tool: **Vite**
- Routing: **TanStack Router** (`@tanstack/react-router`). Do not introduce `react-router-dom` or other routing systems unless explicitly migrating.
- Server-state/data fetching: **TanStack Query** (`@tanstack/react-query`). Prefer it for request caching, invalidation, and async state.
- Client-side state: Zustand exists; prefer it only for genuinely client-only state (UI preferences, local toggles). Avoid duplicating server state into stores.

### Server (`packages/server`)

- HTTP API: **Hono** routes live in `packages/server/src/hono/routes/*.ts`.
- Error mapping middleware: `packages/server/src/hono/middleware/error.ts`.
- JSON API routes under `packages/server/src/hono/routes/*.ts` must use **`OpenAPIHono` + `createRoute(...)`** so they participate in generated OpenAPI docs and typed clients. Plain `Hono` is only for explicit transport/protocol/static exceptions, and those exceptions must carry an inline lint disable with a concrete reason.

### Core (`packages/core`)

- Core is the business logic layer. Keep policy, validation boundaries, and reusable services here.

### CLI (`packages/cli`)

- Entry point: `packages/cli/src/cli/index.ts`
- Static commands (e.g., `dexto init`, `dexto setup`): `packages/cli/src/cli/commands/`

### TUI (`packages/tui`)

- Interactive CLI commands (e.g., `/help`, `/compact`): `packages/tui/src/interactive-commands/`
- Ink-based UI components: `packages/tui/src/`

### Other Important Packages

- **`@dexto/client-sdk`**: Lightweight type-safe client for the Dexto API (Hono-based). Use for external integrations.
- **`@dexto/agent-management`**: Agent registry, config discovery, preferences, and agent resolution logic.
- **`@dexto/analytics`**: Shared PostHog analytics utilities for CLI and WebUI (opt-in telemetry).
- **`@dexto/registry`**: Shared registry data (MCP server presets, etc.) for CLI and WebUI.
- **`@dexto/tools-*`**: Modular tool packages (`tools-filesystem`, `tools-process`, `tools-todo`, `tools-plan`). Each provides a tool provider that registers with the core tool registry.

### Images (`packages/image-*`)

Images are pre-configured bundles of providers, tools, and defaults for specific deployment targets. They use `defineImage()` from core.

- **`@dexto/image-local`**: Local development image with filesystem/process tools, SQLite storage.
- **`@dexto/image-bundler`**: Build tool for bundling images (`dexto-bundle` CLI).

Image definition files use the convention `dexto.image.ts` and register providers (blob stores, custom tools) as side-effects when imported.

### Adding New Packages

All `@dexto/*` packages use **fixed versioning** (shared version number).

When creating a new package:

1. Add the package name to the `fixed` array in `.changeset/config.json`
2. Set its `version` in `package.json` to match other packages (check `packages/core/package.json`)

**Build pattern**: Use `tsup` for JS bundling + `tsc` for type generation:

```json
"build": "tsup && node ../../scripts/clean-tsbuildinfo.mjs && tsc -b tsconfig.json --emitDeclarationOnly"
```

- `tsup`: Fast ESM/CJS bundling via esbuild
- `clean-tsbuildinfo.mjs`: Cleans tsc's cache after tsup (prevents stale cache issues)
- `tsc -b --emitDeclarationOnly`: Generates `.d.ts` files (faster than tsup's dts)

**Exceptions**:

- **Apps** (cli, webui): Use `tsc -p` or `vite build` directly
- **Simple libraries**: Can use just `tsc` if no bundling needed

## Avoiding Duplication (repo-wide)

**Before adding any new helper/utility/service:**

1. Search the codebase first (glob/grep for similar patterns).
2. Prefer extending existing code over creating new.
3. If new code is necessary, justify why existing code doesn't work.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [truffle-ai/dexto](https://github.com/truffle-ai/dexto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
