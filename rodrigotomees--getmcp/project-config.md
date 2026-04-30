---
trigger: always_on
description: > For the complete specification (schemas, transformation rules, config formats, research appendix), see [`SPECIFICATION.md`](./.agents/docs/SPECIFICATION.md).
---

# agent.md — getmcp Codebase Guide

> For the complete specification (schemas, transformation rules, config formats, research appendix), see [`SPECIFICATION.md`](./.agents/docs/SPECIFICATION.md).

---

## Project Summary

**getmcp** is a universal installer and configuration tool for MCP (Model Context Protocol) servers across all AI applications. Every AI app (Claude Desktop, VS Code, Cursor, Goose, Windsurf, Zed, etc.) uses a different config format for MCP servers — different root keys, field names, file formats, and conventions. getmcp solves this by defining a single canonical format (aligned with the [official MCP registry schema](https://registry.modelcontextprotocol.io/)), then using generators to transform it into each app's native format. A registry of popular servers, a CLI for automated installation, and a web directory complete the toolchain.

---

## Architecture Overview

This is a **TypeScript monorepo** (npm workspaces, ESM-only, Node >= 22.17) with 5 packages:

```
@getmcp/cli -----> @getmcp/generators -----> @getmcp/core
            \----> @getmcp/registry   -----> @getmcp/core
@getmcp/web -----> @getmcp/core + @getmcp/generators + @getmcp/registry
```

| Package               | npm Name             | Purpose                                                                                                                                                                                                                         |
| --------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `packages/core`       | `@getmcp/core`       | Zod schemas, TypeScript types, utility functions (type guards, transport inference)                                                                                                                                             |
| `packages/generators` | `@getmcp/generators` | 19 config generators (one per AI app), each transforms canonical format to app-native format                                                                                                                                    |
| `packages/registry`   | `@getmcp/registry`   | Catalog of MCP server definitions with search/filter API                                                                                                                                                                        |
| `packages/cli`        | `@getmcp/cli`        | CLI tool: `add`, `remove`, `list`, `find`, `check`, `update`, `doctor`, `import`, `sync`, `registry` commands with app auto-detection, config merging, multi-registry support, and installation tracking via `getmcp-lock.json` |
| `packages/web`        | `@getmcp/web`        | Next.js (App Router) web directory for browsing servers and generating config snippets, with Vercel Analytics and Speed Insights                                                                                                |

**Tech stack**: TypeScript 5.7+, Zod 4.0+, Vitest 3.0+, Next.js 15.3+ (web), Tailwind CSS 4.0+ (web), `@clack/prompts` (CLI). **Linting/Formatting**: oxlint + oxfmt, enforced via lefthook pre-commit hook.

> See `.agents/docs/SPECIFICATION.md` Section 3 for the full architecture breakdown.

---

## Key Concepts

- **Canonical Format**: Single format aligned with the [official MCP registry schema](https://registry.modelcontextprotocol.io/) (root key `mcpServers`). Stdio: `command`, `args`, `env`, `cwd`, `timeout`, `description`. Remote: `url`, `transport`, `headers`, `timeout`, `description`. Transport auto-inferred from URL.
- **Generators**: Extend `BaseGenerator` (`packages/generators/src/base.ts`), implement `generate()` to transform canonical → app-native format. Each implements `detectInstalled()` for platform-specific app detection.
- **Registry**: `Map<string, InternalRegistryEntry>` keyed by official reverse-DNS name (e.g. `io.github.github/github-mcp-server`), with search and filtering functions. `getServer()` is ID-only; use `getServerBySlug()` for slug lookups (web URLs).
- **CLI**: Auto-detects installed apps, prompts for env vars, generates app-specific configs, and **merges** into existing files (never overwrites). Tracks installations in `getmcp-lock.json`.
- **Design Principles**: (1) Never overwrite (2) Canonical format as single source of truth (3) Auto-detect apps (4) Platform-aware path resolution (5) Schema-validated via Zod

> See [file-map.md](./.agents/docs/file-map.md) for the complete file-by-file reference.

---

## Common Tasks

### MCP server registry

Server data is synced from the [official MCP registry](https://registry.modelcontextprotocol.io) via `packages/registry/scripts/sync.ts`. To add a new server, submit it to the official registry — getmcp syncs automatically via the daily GitHub Actions workflow.

### Adding a new generator (supporting a new AI app)

1. Add the new app ID to the `AppId` enum in `packages/core/src/schemas.ts`
2. Create `packages/generators/src/<app-name>.ts` extending `BaseGenerator`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RodrigoTomeES/getmcp](https://github.com/RodrigoTomeES/getmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
