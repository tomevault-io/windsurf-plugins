---
trigger: always_on
description: This repo is the private `atomic-monorepo` Bun workspace. It currently houses:
---

# Development Rules

## Overview

This repo is the private `atomic-monorepo` Bun workspace. It currently houses:

- `@bastani/atomic` in `packages/coding-agent` — the Atomic-branded fork of pi's coding-agent CLI and the only independently published package.
- `@bastani/workflows` in `packages/workflows` — a first-party extension for Atomic/pi that brings multi-stage, DAG-driven workflow execution to agent sessions.
- `@bastani/subagents` in `packages/subagents` — builtin subagent orchestration, reusable agent definitions, skills, prompts, chains, and TUI clarification support.
- `@bastani/mcp` in `packages/mcp` — builtin MCP adapter extension that exposes MCP servers as agent tools.
- `@bastani/web-access` in `packages/web-access` — builtin web search, URL fetching, GitHub repository, PDF, and video extraction tools.
- `@bastani/intercom` in `packages/intercom` — builtin coordination channel for parent/child and cross-session agent communication.

Companion packages under `packages/*` ship as **raw TypeScript** (no compile step) and are bundled into `@bastani/atomic` at build time rather than published independently. The coding-agent package follows upstream pi's compiled-package layout.

## Tech Stack

- **[Bun](https://bun.sh) ≥ 1.3.14** for the runtime, package manager, and test runner
- TypeScript ≥ 5.x (strict, `noUnusedLocals`, `noUnusedParameters`)
- `bun:test` + `node:assert/strict` for tests
- `@sinclair/typebox` for schema definitions
- `jiti` for runtime TS loading where needed

## Quick Reference

### Commands

Default to using **Bun**, not Node/npm/yarn/pnpm.

- Use `bun <file.ts>` instead of `node --experimental-strip-types <file.ts>` or `ts-node <file>`
- Use `bun test <path>` instead of `node --test` or Jest/Vitest CLIs
- Use `bun run typecheck` to run TypeScript type checks (`tsc --noEmit`)
- Use `bun install` instead of `npm install`, `yarn install`, or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Repo commands: `bun run test:unit`, `bun run test:integration`, `bun run test:all`, `bun run typecheck`, `bun run check:file-length`, `bun run lint`, `bun run hooks:install`, `bun run hooks:run`
- Git hooks are configured in `prek.toml`; `bun install` runs the root `prepare` script to install hooks with `prek install --prepare-hooks` using `default_install_hook_types`.

**Exception — publishing:** `npm publish --provenance` is still the registry publish tool because npm's OIDC-signed provenance lives in the npm CLI. Everything else is Bun.

## Best Practices

- Avoid ambiguous types like `any` and `unknown`. Use specific types instead.
- Source files use `.js` import extensions (TypeScript ESM convention). The repo ships as `.ts` files; Bun resolves `.js` specifiers to the underlying `.ts` source directly — no loader hook required. atomic's loader follows the same convention as pi.
- Do not add a build step (`dist/`, `tsconfig.build.json`, etc.) to `packages/workflows`; it distributes raw TypeScript and the host loads it directly. `packages/coding-agent` is copied from upstream pi and keeps its existing build setup.
- When using skills, if you see a frontmatter of `metadata: internal` set to `true` (if missing assume `false`), that means the skill is for internal developers of this package. If this flag is omitted, the skill is meant for consumers/everyday users.

## Design Context

Refer to `DESIGN.md` and `PRODUCT.md`.

## Testing

Use `bun run test:unit` (or `test:integration`, `test:all`) and make use of your tdd skill to write high quality tests. Tests use `bun:test` + `node:assert/strict`:

```ts#test/unit/index.test.ts
import { test } from "bun:test";
import assert from "node:assert/strict";

test("hello world", () => {
  assert.equal(1, 1);
});
```

### Hook name compatibility

Bun's `bun:test` exports `beforeAll`/`afterAll` (not `before`/`after`). Use `beforeAll`/`afterAll` for once-per-suite setup/teardown and `beforeEach`/`afterEach` for per-test hooks.

### AI Agent Integration

When using Bun’s test runner with AI coding assistants, you can enable quieter output to improve readability and reduce context noise. This feature minimizes test output verbosity while preserving essential failure information.
​
**Environment Variables**

Set any of the following environment variables to enable AI-friendly output:
`CLAUDECODE=1` - For Claude Code
`REPL_ID=1` - For Replit
`AGENT=1` - Generic AI agent flag

### Code Quality

- Frequently run linters and type checks using `bun run typecheck` and `bun run lint` (both `tsc --noEmit`), and run `bun run check:file-length` to enforce the 500-line file-length gate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bastani-inc/atomic](https://github.com/bastani-inc/atomic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
