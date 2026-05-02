---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Firewatch is a CLI tool that fetches, caches, and queries GitHub PR activity. It outputs pure JSONL designed for `jq` composition. The binary is `fw`.

## Development Commands

This is a Bun workspaces monorepo. Root scripts delegate to workspace packages:

```bash
bun run dev              # Run CLI with --watch (delegates to @outfitter/firewatch-cli)
bun run build            # Build all workspace packages
bun run check            # Lint (oxlint) + type check all packages
bun run test             # Run tests with bun test
bun run lint             # Run oxlint
bun run lint:fix         # Run oxlint with auto-fix
bun run format           # Run oxfmt
```

Run the CLI directly during development:

```bash
bun apps/cli/bin/fw.ts sync outfitter-dev/firewatch
bun apps/cli/bin/fw.ts query --type review --since 24h
bun apps/cli/bin/fw.ts status --short
```

For a local `fw-dev` alias during development:

```bash
./scripts/symlink-dev.sh          # Links ~/.bun/bin/fw-dev -> apps/cli/dist/fw
./scripts/symlink-dev.sh fw-local # Custom alias
```

## Architecture

```
apps/
├── cli/                    # @outfitter/firewatch-cli - Commander-based CLI
│   ├── bin/fw.ts           # Entry point
│   └── src/commands/       # Command implementations
└── mcp/                    # @outfitter/firewatch-mcp - MCP server interface

packages/
├── core/                   # @outfitter/firewatch-core - Pure library logic
│   ├── ack.ts              # Local acknowledgement tracking
│   ├── auth.ts             # Adaptive auth: gh CLI → env → config
│   ├── authors.ts          # Author filtering and bot detection
│   ├── batch.ts            # Batch operations (reactions, ID resolution)
│   ├── cache.ts            # XDG-compliant paths and directory setup
│   ├── check.ts            # Staleness detection & file activity hints
│   ├── config.ts           # Config loading (.firewatch.toml, XDG config)
│   ├── db.ts               # SQLite database initialization
│   ├── github.ts           # GraphQL client using native fetch
│   ├── parity.ts           # CLI/MCP shared operations
│   ├── query.ts            # Entry filtering with SQLite queries
│   ├── repo-detect.ts      # Auto-detect repo from git/package.json/Cargo.toml
│   ├── repository.ts       # SQLite repository operations (CRUD, sync metadata)
│   ├── short-id.ts         # Short ID generation and resolution
│   ├── sync.ts             # Incremental sync with cursor tracking
│   ├── time.ts             # Duration parsing (7d, 24h, etc.)
│   ├── worklist.ts         # Per-PR summary aggregation, stack-aware ordering
│   ├── schema/             # Zod schemas for type safety
│   ├── stack/              # Stack provider abstraction (Graphite, future GitHub)
│   └── plugins/            # Plugin interface (e.g., Graphite stacks)
├── shared/                 # @outfitter/firewatch-shared - Shared utilities
└── claude-plugin/          # Local Claude Code plugin marketplace
    ├── .claude-plugin/
    │   └── marketplace.json  # Marketplace manifest
    └── firewatch/            # Firewatch plugin
        ├── .claude-plugin/
        │   └── plugin.json   # Plugin manifest
        ├── commands/         # /firewatch:* commands
        ├── hooks/            # Session lifecycle hooks
        ├── scripts/          # Hook scripts
        └── skills/firewatch/ # triage, respond skills
```

### CLI Commands

| Command    | Description                                    |
| ---------- | ---------------------------------------------- |
| `query`    | Query cached activity (JSONL output for jq)    |
| `list`     | Opinionated feedback view (`list prs` for PRs) |
| `view`     | View PR or comment details (polymorphic)       |
| `reply`    | Reply to a comment                             |
| `comment`  | Add PR-level comment                           |
| `ack`      | Acknowledge feedback (local tracking)          |
| `close`    | Resolve thread or close PR (polymorphic)       |
| `approve`  | Approve PR (GitHub review)                     |
| `reject`   | Request changes on PR (GitHub review)          |
| `edit`     | Edit PR or comment (polymorphic)               |
| `sync`     | Sync cache (`--clear` to reset)                |
| `status`   | Show Firewatch state                           |
| `config`   | View/edit configuration                        |
| `doctor`   | Diagnose setup issues                          |
| `schema`   | Print JSON schema                              |
| `examples` | Show jq patterns                               |
| `mcp`      | Start MCP server                               |

### Key Patterns

**Layered architecture**: Core library functions in `packages/core/` are interface-agnostic. CLI (`apps/cli/`) and MCP server (`apps/mcp/`) wire these to different interfaces.

**Adaptive auth** (`packages/core/src/auth.ts`): Tries gh CLI first, then environment variables, then config file token.

**SQLite storage**: Entries stored in SQLite with denormalized PR context. Output as JSONL for jq composition.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [outfitter-dev/firewatch](https://github.com/outfitter-dev/firewatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
