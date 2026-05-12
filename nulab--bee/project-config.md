---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Overview

bee (`bee`) — a CLI for the Backlog project management service. pnpm workspace monorepo with ESM-only packages.

## Commands

```sh
# Install
pnpm install

# Lint (oxlint, NOT ESLint)
pnpm run lint
pnpm run lint:fix

# Type check (tsc --noEmit per package via turbo)
pnpm run typecheck

# Format (oxfmt)
pnpm run format
pnpm run format:check

# Test
pnpm run test                                              # all tests
pnpm --filter @repo/backlog-utils exec vitest run src/client.test.ts # single file

# Build
pnpm --filter @nulab/bee build

# Dev (CLI)
pnpm --filter @nulab/bee dev
```

## Module Resolution: bundler

TypeScript is configured with `module: "preserve"` / `moduleResolution: "bundler"`. This means:

- **Relative imports omit the file extension**:
  ```ts
  // Correct
  import { foo } from "./foo";
  // Wrong
  import { foo } from "./foo.js";
  import { foo } from "./foo.ts";
  ```
- **Inline `type` keyword** — use `import { type Foo, bar }` instead of separate `import type { Foo }` (enforced by oxlint)

## Architecture

```
apps/cli             — CLI entry point (citty framework, consola logging)
apps/docs            — Astro Starlight documentation site
packages/backlog-utils — Backlog API client wrapper (backlog-js, OAuth auto-refresh, rate-limit handling)
packages/cli-utils   — Shared CLI utilities (output formatting, table, splitArg, prompts)
packages/config      — CLI configuration management (~/.config/bee RC file, space/auth resolution)
packages/test-utils  — Shared test helpers (mock client, mock consola, process.exit spy, vi.clearAllMocks setup)
packages/tsconfigs   — Shared TypeScript base config
```

### Documentation site (`apps/docs`)

Command reference pages are **auto-generated** from CLI source code — do NOT create `.md` files under `apps/docs/src/content/docs/commands/`. The dynamic route `apps/docs/src/pages/commands/[...slug].astro` uses `loadCommands()` (in `apps/docs/src/lib/commands.ts`) to import each command's `commandUsage` and `defineCommand` metadata at build time and render documentation pages automatically.

**When adding or removing CLI commands**, also update the command table in `skills/using-bee/SKILL.md` to keep the Skill in sync with the CLI.

### Skills (`skills/`)

AI エージェント向けの Skill 定義を格納するディレクトリ。

| Skill              | 用途                                           |
| ------------------ | ---------------------------------------------- |
| `using-bee`        | bee CLI の使い方（コマンド、フラグ、パターン） |
| `backlog-notation` | Backlog 記法（Backlog記法）の構文リファレンス  |

#### Definition lists

The docs site supports Markdown definition list syntax via `remark-definition-list`. Use this instead of raw `<dl>`/`<dt>`/`<dd>` HTML:

```mdx
<!-- Correct — Markdown definition list -->

用語
: 説明文

<!-- Wrong — raw HTML -->

<dl>
  <dt>用語</dt>
  <dd>説明文</dd>
</dl>
```

This also works inside JSX components like `<Card>`.

#### Internal link conventions

All internal links in documentation content (`apps/docs/src/content/docs/`) must use **absolute paths with the base prefix `/bee/` and a trailing slash**:

```mdx
<!-- Correct -->

[認証ガイド](/bee/guides/authentication/)

<LinkCard title="CI/CD" href="/bee/integrations/ci-cd/" />

<!-- Wrong — relative paths -->

[認証ガイド](../guides/authentication/)

<LinkCard title="CI/CD" href="../../integrations/ci-cd/" />

<!-- Wrong — missing trailing slash -->

[認証ガイド](/bee/guides/authentication)

<!-- Wrong — missing base prefix -->

[認証ガイド](/guides/authentication/)
```

- Links to file resources (`.md`, `.txt`) do NOT get a trailing slash: `[file](/bee/llms.txt)`
- `starlight-links-validator` runs at build time and fails on broken links
- Command reference pages (`/bee/commands/**`) are dynamically generated and excluded from link validation

**To add docs for a new command group**, only add sidebar entries to `apps/docs/astro.config.mjs`:

```js
{
  label: "issue",
  items: [
    { label: "issue list", link: "/commands/issue/list" },
    // ...
  ],
},
```

`@repo/backlog-utils` exposes `getClient()` which returns a `backlog-js` `Backlog` instance with OAuth 401 auto-refresh (via Proxy) and rate-limit error handling. Commands call `client.getIssues()`, `client.getProjects()`, etc. directly.

`@nulab/bee` uses citty's `defineCommand` / `runMain` with subcommand registration and a custom help system (see below).

## Command Help System

CLI commands use a **single-source help system** inspired by gh CLI. Each command defines a `CommandUsage` object that drives both `--help` output and documentation generation from the same data.

### How it works

1. Each command file exports `commandUsage: CommandUsage` alongside the command definition
2. The command is wrapped with `withUsage(defineCommand({ ... }), commandUsage)` to attach the usage data
3. `runMain` receives `showCommandUsage` as a custom `showUsage` handler, which renders gh-cli style help for commands with attached usage and falls back to citty's default for others

### Adding help to a command

```ts
import { defineCommand } from "citty";
import { type CommandUsage, withUsage } from "./lib/command-usage";

export const commandUsage: CommandUsage = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nulab/bee](https://github.com/nulab/bee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
