---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **When working on a new feature, always consult [`skills/aixyz/SKILL.md`](./skills/aixyz/SKILL.md) first.**
> It is the canonical reference for how the project is designed and what conventions to follow.
> Available externally at [`skills.sh/agentlyhq/aixyz`](https://skills.sh/agentlyhq/aixyz).

## Project overview

Monorepo for aixyz — a framework for bundling AI agents from any framework into deployable services with A2A, MCP, x402 payments, and ERC-8004 identity protocols. Managed with Bun workspaces and Turbo. Runtime: Bun 1.3.9+.

## Commands

```bash
bun install                          # Setup
bun run build                        # Build all packages (turbo run build)
bun run test                         # Test all packages (turbo run test)
bun run lint                         # Lint all packages with --fix
bun run format                       # Format with Prettier (printWidth: 120)
bun run clean                        # Clean build artifacts
```

### Package-specific commands

```bash
bun run build --filter=aixyz         # Build a specific package
bun run test --filter=@aixyz/erc-8004  # Test a specific package
```

Or navigate directly:

```bash
cd packages/aixyz && bun run build
cd examples/flight-search && bun run dev
```

### Running a single test

Tests use Bun's built-in test runner (`.test.ts` files):

```bash
bun test packages/aixyz-erc-8004/src/schemas/registration.test.ts
```

### Example agent dev/build

```bash
cd examples/flight-search
bun run dev    # → aixyz dev (hot reload, watches app/ and aixyz.config.ts)
bun run build  # → aixyz build (bundles for deployment)
```

## Repo layout

### Packages (`packages/*`)

| Package            | npm name           | Description                                                                |
| ------------------ | ------------------ | -------------------------------------------------------------------------- |
| `aixyz`            | `aixyz`            | Main framework: server, plugins (A2A, MCP), x402 integration, web-standard |
| `aixyz-cli`        | `@aixyz/cli`       | CLI: `dev`, `build`, `erc-8004 register`, `erc-8004 update`                |
| `aixyz-config`     | `@aixyz/config`    | Zod-validated config loading from `aixyz.config.ts` + .env files           |
| `aixyz-stripe`     | `@aixyz/stripe`    | Experimental Stripe payment adapter                                        |
| `create-aixyz-app` | `create-aixyz-app` | Project scaffolding (`bunx create-aixyz-app`)                              |
| `aixyz-erc-8004`   | `@aixyz/erc-8004`  | ERC-8004 contract ABIs, addresses, Zod schemas                             |

### Examples (`examples/*`)

Working agents demonstrating patterns. All share the same structure:

```
examples/*/
  aixyz.config.ts     # Agent metadata and config (required)
  app/
    agent.ts          # Agent definition (optional, enables A2A endpoints)
    server.ts         # Custom server (optional, overrides auto-generation)
    accepts.ts        # Custom x402 facilitator (optional)
    tools/*.ts        # Tool implementations (files starting with _ ignored)
    icon.png          # Agent icon (served as static asset)
  vercel.json
```

### Documentation (`docs/`)

Mintlify documentation site (`mint dev` to preview locally). Structure:

- `docs/getting-started/` — Installation, project structure, why Bun, agent and tools, payments (x402), deploying, testing
- `docs/config/` — aixyz.config.ts reference, environment variables
- `docs/api-reference/` — File-system conventions (agent.ts, agent.test.ts, tools/) and Functions (AixyzApp, A2APlugin, MCPPlugin, loadEnv, Accepts)
- `docs/protocols/` — A2A, MCP, x402, ERC-8004 (collapsed under Documentation tab)
- `docs/packages/` — Package reference docs (collapsed under Documentation tab)
- `docs/templates/` — Individual pages for each example template (separate Templates tab)
- `docs/docs.json` — Mintlify navigation configuration

Protocols and Packages are groups within the Documentation tab (not separate tabs).
Templates have their own tab with one page per example.

Each `docs/templates/<name>.mdx` documents the corresponding `examples/*/` example.

### Other

- `CLAUDE.md` symlinks to `AGENTS.md`

## Architecture

### How the build pipeline works (`@aixyz/cli`)

The `aixyz build` command in `packages/aixyz-cli/build/index.ts`:

1. Loads config from `aixyz.config.ts` via `@aixyz/config`
2. Uses two Bun build plugins:
   - **`AixyzConfigPlugin`** — Materializes resolved config into the bundle (replaces `aixyz/config` imports)
   - **`AixyzServerPlugin`** — Auto-generates `server.ts` from `app/` structure if no `app/server.ts` exists. Scans
     `app/agent.ts` and `app/tools/*.ts`, wires up A2A + MCP + x402 middleware
3. Bundles with `Bun.build()` targeting Node.js
4. Output format:
   - **Vercel** (when `VERCEL=1` or `--output vercel`): `.vercel/output/` Build Output API v3
   - **Standalone** (default): `.aixyz/output/server.js`
   - **Executable** (`--output executable`): `.aixyz/output/server` self-contained binary


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentlyHQ/aixyz](https://github.com/AgentlyHQ/aixyz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
