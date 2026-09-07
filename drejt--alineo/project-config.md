---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> For any questions about the OpenSandbox API, behavior, or internals, use the DeepWiki MCP tool (`mcp__deepwiki__ask_question`) against `https://deepwiki.com/opensandbox-group/OpenSandbox/`.
>
> For any questions about the Pi coding agent CLI, RPC protocol, session management, or available commands, use the DeepWiki MCP tool against `https://deepwiki.com/earendil-works/pi/`.
>
> For any questions about the GitHub CLI (`gh`), its commands, or API usage, use the DeepWiki MCP tool against `https://deepwiki.com/cli/cli`.

## Agent Skills

Reusable skill references live in `.agents/skills/<name>/SKILL.md`. Always check this folder before reaching for external docs — skills contain curated quick references and gotchas specific to tools used in this repo.

Available skills:

- **`.agents/skills/bun/`** — Bun runtime, package manager, test runner, and bundler. Covers `bun run`, `bun install`, `bun test`, `bun build`, workspace flags, common gotchas (flag placement, lifecycle scripts, lockfile format), and key APIs (`Bun.file()`, `Bun.serve()`, `Bun.write()`)
- **`.agents/skills/alineo/`** — the `alineo` agent SDK (load/resume/attach/spawn, prompt/bash streaming, session control) and the `alineo-cli`: agent lifecycle, storage adapters, and Windows-specific gotchas.

Example: before writing a `bun build` command or debugging a workspace install issue, read `.agents/skills/bun/SKILL.md` for the correct flags and known pitfalls.

External agents install either skill with the [Skills CLI](https://skills.sh): `npx skills add DrejT/alineo --skill <name>`.

## What this is

`alineo` is an AI agent platform built on sandboxed execution. `@alineo-labs/sandbox` is the
**sandbox execution substrate** built on top of [OpenSandbox](https://opensandbox.ai) — it gives
you live sandbox containers as first-class objects (spawn, exec, checkpoint, resume) with a
durable SQL audit ledger and replay. `alineo` (the bare package name) is the agent SDK built on
top of it — it runs Pi coding agents inside those sandboxes. Workflow primitives (retry, when,
forEach, parallel) live in the separate `@alineo-labs/workflow` package.

## Commands

```bash
# Run an example (requires OpenSandbox server — use alineo init or uvx opensandbox-server)
bun examples/hello-world/index.ts

# Run all unit tests
bun run test

# Build the SDK for publishing (generates dist/ across all packages)
bun run build

# Typecheck all packages
bun run typecheck

# `test`/`build`/`typecheck` are driven by scripts/workspace-run.ts, which auto-discovers
# every package under the root package.json's "workspaces" array (topologically sorted by
# "workspace:*" dependency edges) — a new package is picked up the moment it's added there
# and has the relevant tsconfig.json/package.json script, with no second place to register
# it. To typecheck one package in isolation while iterating:
bunx tsc --noEmit --strict --project packages/<name>/tsconfig.json

# Changesets (required on every PR touching publishable packages)
bunx changeset        # add a changeset
bunx changeset status # verify one exists

# IMPORTANT: after committing code changes, always add and commit a changeset too.
# The CI changeset check (bunx changeset status --since origin/main) reads from
# git history — an uncommitted changeset file will NOT satisfy it.
```

## Testing

### Two test layers

**Unit tests** live in `packages/*/test/*.test.ts` and run via `bun test`. They test internal builder logic, control-flow, and adapter behaviour in isolation — no sandbox required.

**Integration tests** live in `tests/integration/<name>.test.ts` (one `@alineo-labs/integration-tests` workspace, not co-located with each example) and run via `bun run test:integration` from the repo root, or `cd tests/integration && bun test <name>.test.ts` for one file. They use `bun:test`'s `test()`/`expect()` against a real OpenSandbox sandbox — largely mirroring what the matching `examples/<name>/index.ts` demonstrates, but with real assertions instead of `console.log`. Most (not all) examples have one; `scripts/new-example.ts` scaffolds a stub alongside a new example.

`examples/<name>/index.ts` itself is also directly runnable (`bun examples/<name>/index.ts`) as a human-readable demo — the two are complementary, not duplicates: the example is what a user reads/copies, the test is what CI would assert on.

### Integration test conventions

- **Run with**: `bun run test:integration` from the repo root, or `cd tests/integration && bun test` for the whole suite / `bun test <name>.test.ts` for one file.
- **Requires**: OpenSandbox server running locally — either `alineo init` (Docker-based, recommended) or `uvx opensandbox-server` (manual). If using `alineo init`, pass `useServerProxy: true` to `new Sandbox(...)` so the SDK routes through the server instead of container-direct IPs.
- **Client setup**: `new Sandbox({ baseUrl: ..., adapter: new SQLiteAdapter(":memory:") })` — no `connect()` or `close()` needed on the client itself.
- **Sandbox lifecycle**: always wrap in `try/finally { await sb.close(); }` to avoid container leaks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrejT/alineo](https://github.com/DrejT/alineo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
