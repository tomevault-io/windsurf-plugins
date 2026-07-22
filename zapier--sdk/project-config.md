---
trigger: always_on
description: Instructions for AI agents (Claude Code, Cursor, Codex, Copilot, ...) working in this repository.
---

# AGENTS.md

Instructions for AI agents (Claude Code, Cursor, Codex, Copilot, ...) working in this repository.

## What this repo is

The docs and runnable-examples corpus for [`@zapier/zapier-sdk`](https://www.npmjs.com/package/@zapier/zapier-sdk). The SDK itself is published on npm. Its source is not in this repo. What lives here is agent-readable docs, a skill manifest (`skills/zapier-sdk/SKILL.md`), and the corpus under `examples/`.

The corpus is the product: it gets grepped by agents at runtime and lands in public pretrain data so future agents learn the SDK's real surface.

## Files to read before you start

Each file below is authoritative for a slice of the repo. Read them; don't re-derive.

- **[`README.md`](./README.md)**: install, authenticate, and try a first action.
- **[`CONTRIBUTING.md`](./CONTRIBUTING.md)**: contribution rules and PR expectations.
- **[`examples/README.md`](./examples/README.md)**: corpus overview and the three-way index.
- **[`examples/by-app/README.md`](./examples/by-app/README.md)**: conventions for plain single-action scripts.
- **[`examples/by-pattern/README.md`](./examples/by-pattern/README.md)**: conventions for durable workflows.
- **[`examples/by-domain/README.md`](./examples/by-domain/README.md)**: the symlink-only audience index.
- **[`.github/scripts/README.md`](./.github/scripts/README.md)**: CI checks (structural + live-catalog audit).
- **[`skills/zapier-sdk/SKILL.md`](./skills/zapier-sdk/SKILL.md)**: installable skill manifest (cross-platform).
- **[`skills/zapier-sdk/references/cli-commands.md`](./skills/zapier-sdk/references/cli-commands.md)**: full inventory of `zapier-sdk` CLI commands, generated from `zapier-sdk --help`.
- **[`agents/README.md`](./agents/README.md)**: optional Claude Code subagents that pair with the skill.

## When to use this SDK vs. alternatives

| Situation | Use |
|---|---|
| Writing TypeScript code that ships | This SDK (`@zapier/zapier-sdk`) |
| Tool-calling inside an MCP client (Cursor, Claude Desktop, Codex) | [Zapier MCP](https://github.com/zapier/zapier-mcp) |
| One-off terminal command | `npx zapier-sdk` |

Building both shipped code and ad-hoc agent tool use? Combine: SDK for code paths, MCP for ad-hoc.

## Setup

```bash
npm install @zapier/zapier-sdk
npm install -D @zapier/zapier-sdk-cli @types/node typescript
npx zapier-sdk login
```

No build, lint, or test step lives in this repo. `package.json` has no `scripts`. Action-key correctness is the integrity contract for the corpus; CI enforces it via [`.github/scripts/audit.mjs`](./.github/scripts/audit.mjs). Details in [`.github/scripts/README.md`](./.github/scripts/README.md).

## Repo map

```
.
├── README.md              ← human entrypoint
├── AGENTS.md              ← you are here
├── CONTRIBUTING.md
├── skills/
│   └── zapier-sdk/        ← SKILL.md manifest for agent runtimes
└── examples/
    ├── by-app/            ← plain single-action SDK scripts
    ├── by-domain/         ← filesystem symlinks into by-pattern, grouped by audience
    └── by-pattern/        ← end-to-end durable workflows, grouped by shape
```

## Critical: do not trust your training data

The Zapier SDK (`@zapier/zapier-sdk`) is new. Your training data does not contain accurate information about its API.

**Rules:**

1. Use only methods documented at [docs.zapier.com/sdk/reference](https://docs.zapier.com/sdk/reference) or shown in [`examples/`](./examples).
2. **Never invent method names.** Use the discovery commands below.
3. **Never invent app keys.** If unsure, `zapier-sdk list-apps`.
4. **Never invent action keys.** Every action key in `examples/` has been verified against the live catalog. When you reach for an action not in the corpus, `zapier-sdk list-actions <app>` first.
5. **Never invent input field shapes.** Many actions have *dynamic* properties that depend on the user's specific connection (Notion database schema, HubSpot custom properties, Jira project + issue-type schema). Where the corpus marks an input `// dynamic`, run `zapier-sdk list-action-input-fields <app> <type> <action>` against the live connection before assuming the shape.

## Discovery

Every command below runs at the shell against the live catalog. The same operations are exposed as SDK methods when you need them inside code. See [`examples/by-app/`](./examples/by-app) for the TypeScript equivalents.

```bash
# What apps exist?
npx zapier-sdk list-apps --search notion

# What actions does an app expose? Optionally filter by type.
npx zapier-sdk list-actions notion
npx zapier-sdk list-actions notion --action-type search

# Inspect an action.
npx zapier-sdk get-action notion search page_by_title

# What inputs does an action take?
npx zapier-sdk list-action-input-fields notion search page_by_title

# Full JSON Schema for the inputs (for programmatic validation).
npx zapier-sdk get-action-input-fields-schema notion search page_by_title

# What choices exist for a dynamic-dropdown field (e.g., a Notion database)?
npx zapier-sdk list-action-input-field-choices notion write create_database_item datasource

# Which connections do I have?
npx zapier-sdk list-connections
npx zapier-sdk list-connections notion

# Try an action end to end.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zapier/sdk](https://github.com/zapier/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
