---
trigger: always_on
description: The map of this repository, for people and coding agents alike. `README.md`
---

# AGENTS.md

The map of this repository, for people and coding agents alike. `README.md`
is the product page. Read `docs/CONSTITUTION.md` before structural changes.

VibeKit exposes Algorand capabilities through one tool contract across MCP,
CLI, and an LLM agent loop, and ships an Explorer (terminal and web) that
renders what those tools return.

## What runs where

| Path | What it is |
| --- | --- |
| `packages/vibekit` | The one published package, `@initlabs/vibekit`. Every `src/` area is a subpath export. |
| `packages/vibekit/src/core` (`.`) | The tool contract, `resolveDeployment`, `executeToolCall`, the codec, and the compose engine (`core/compose/`). |
| `packages/vibekit/src/tools` (`./tools`, `./tools/views`) | The domain tools: accounts, assets, contracts, network, transactions — each a flat directory of `index.ts` (the tool definitions), `schemas.ts`, and the functions behind them. `views.ts` maps each view id to its wire schema. |
| `packages/vibekit/src/plugins/*` (`./plugins/<name>`) | Third-party tool plugins: nfd, pera, vestige, alpha-arcade. |
| `packages/vibekit/src/signer-keystore` (`./signer-keystore`) | The algosdk signer over the keystore daemon, its account tools, the testnet dispenser. |
| `packages/vibekit/src/mcp` (`./mcp`, `./mcp/stdio`, `./mcp/http`) | Host: ToolDefinition-to-MCP adapter and transports. |
| `packages/vibekit/src/agent` (`./agent`, `./agent/config`) | Host: the LLM tool loop and its stored config. |
| `packages/vibekit/src/preset` (`./preset`) | The stock tool and plugin mix every stock host composes from. |
| `packages/vibekit/examples` | Reference stdio and HTTP deployments, typechecked with the package. |
| `packages/explorer` (private) | The Explorer protocol (`src/core`: result envelope, view ids, write-stage events), view models (`src/views`), the write flow (`src/flows`), the tool-result bridge (`src/bridge.ts`), input classification (`src/input.ts`), formatting (`src/format.ts`), recorded sample data (`src/sample`), and the live host (`src/live`). Not a UI. `src/index.ts` lists what the apps use; everything else is internal. |
| `apps/cli` | The `vibekit` binary: `new`, `init`, `localnet`, `keystore`, `dispenser`, `doctor`, `tool`, `mcp`, `explore`. Host: `commands/tool.ts` and `commands/mcp.ts`. |
| `apps/tui` | The terminal Explorer (OpenTUI). Live against a network when reachable, sample data otherwise. `features/<name>/` holds one feature's hooks, screen, and cards; `feed/` is the transcript; `app.tsx` composes them. |
| `apps/web` | The web Explorer (Next.js). Sample-backed reads plus a compose-only flow route. |
| `apps/website` | The public site (Astro/Starlight). |
| `skills/` | Canonical skills, compiled into the CLI by `bun run --cwd apps/cli bundle-skills`. `.agents/skills`, `.claude/skills`, `.grok/skills` are symlinks into it. |
| `verify/` | The packed-consumer gate (`bun run verify:packed`). The LocalNet smoke is `apps/cli/scripts/smoke-localnet.ts` (`bun run smoke:localnet`, needs `vibekit localnet start`; CI runs it). |
| `test-prompts/` | Agent-run MCP acceptance prompts and their transcripts. |
| `ideas/` | Dated design notes that are not yet work. |
| `out/` | Marketing video build output (see the `marketing-content` skill). |
| `docs/` | `CONSTITUTION.md` and `PRODUCTION.md` only. |

Tests mirror `src/` under each package's `test/`. Apps consume packages only
through their public exports (`workspace:*`); packages never depend on apps.

## How a call flows

Read:

```
host (mcp | agent | cli tool) → executeToolCall(deployment, tool, args)
  → picks the network context → tool.handler(ctx, args) → jsonSafe()
  → output schema check → wire result
Explorer: wire → build*Record (packages/explorer/src/views) → StructuredResult
  → ViewSpec → create*ViewModel → card (apps/tui/src/features/*/cards.tsx)
```

Write:

```
tool.handler → composeOrExecute(ctx, TxnSpec[]) (core/compose)
  → compose mode: unsigned group, base64 | execute mode: sign, send, confirm
Explorer write flow: draft → simulate → inspect → approve → sign → confirm
  (packages/explorer/src/flows; every stage is a recorded result)
```

Every host sends calls through `executeToolCall` in
`packages/vibekit/src/core/deployment.ts`, which validates the arguments
against the tool's schema whatever the host parsed. Resolved contexts are
frozen before handlers receive them. Local file reads are a capability the
host grants (`readFile`); remote hosts leave it unset. Tools return structured data and declare a
`view` id; they never return JSX, HTML, or terminal markup.

## Glossary

One word per concept. Do not introduce a synonym.

- **tool** — a `ToolDefinition`: name, description, Zod parameters, optional
  output schema, flags, `view`, handler.
- **deployment** — a configured set of tools and plugins over one or more
  networks, in execute or compose mode, with an optional signer.
- **host** — a process that runs tool calls through `executeToolCall`: the
  MCP server, the agent loop, `vibekit tool`. In `packages/explorer`, a
  `*Host` interface is the backend an Explorer app calls for results
  (`LiveHost`, the fixture host).
- **core** — `packages/vibekit/src/core`. Not "kernel" or "engine".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [initlabsai/vibekit](https://github.com/initlabsai/vibekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
