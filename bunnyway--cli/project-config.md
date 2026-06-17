---
trigger: always_on
description: This document describes the architecture, conventions, and implementation details for the bunny.net CLI. It serves as the canonical reference for AI agents and contributors working on this codebase.
---

# AGENTS.md: bunny.net CLI

This document describes the architecture, conventions, and implementation details for the bunny.net CLI. It serves as the canonical reference for AI agents and contributors working on this codebase.

---

## Overview

The bunny.net CLI (`bunny`) is a command-line interface for interacting with bunny.net services (Magic Containers, Edge Scripting, databases). It is written in TypeScript, runs on the Bun runtime, and follows patterns inspired by Cobra (Go).

The CLI supports profile-based authentication, browser-based login, and a modular command structure built on `yargs`.

---

## Runtime & Tooling

| Concern             | Tool    | Notes                                                           |
| ------------------- | ------- | --------------------------------------------------------------- |
| Runtime             | **Bun** | Runs TypeScript natively. No transpilation step in development. |
| Package manager     | **Bun** | `bun add`, `bun install`. Lockfile is `bun.lock`.               |
| Test runner         | **Bun** | `bun test`. Jest-compatible API.                                |
| Build / compile     | **Bun** | `bun build --compile` produces a single native executable.      |
| Watch mode          | **Bun** | `bun --watch packages/cli/src/index.ts` for development.        |
| Env loading         | **Bun** | Auto-loads `.env` files. No `dotenv` package needed.            |
| Local HTTP servers  | **Bun** | `Bun.serve()` for the auth callback server. No Express needed.  |
| Subprocess spawning | **Bun** | `Bun.spawn()` for opening browsers, running child processes.    |

### Why Bun

Bun replaces the entire Node.js toolchain. There are no separate tools for transpilation (`ts-node`, `tsx`), bundling (`esbuild`, `webpack`), testing (`jest`, `vitest`), or executable packaging (`pkg`, `nexe`). The `tsconfig.json` exists only for editor type-checking (`tsc --noEmit`); Bun handles all execution and compilation.

---

## Dependencies

### Runtime dependencies

| Package          | Purpose                                                        |
| ---------------- | -------------------------------------------------------------- |
| `yargs`          | Command routing, subcommands, flag parsing, auto-help          |
| `chalk`          | Terminal string styling (colors, bold, dim)                    |
| `ora`            | Terminal spinners for async operations                         |
| `prompts`        | Interactive input: password masks, confirmations, multi-select |
| `cli-table3`     | Formatted terminal tables                                      |
| `zod`            | Schema validation for config files and CLI input               |
| `@libsql/client` | libSQL database client (used by `db shell`)                    |
| `openapi-fetch`  | Type-safe HTTP client generated from OpenAPI specs             |
| `jsonc-parser`   | JSONC parser for `bunny.jsonc` config files                    |

### Dev dependencies

| Package              | Purpose                                            |
| -------------------- | -------------------------------------------------- |
| `@types/yargs`       | Type definitions for yargs                         |
| `@types/prompts`     | Type definitions for prompts                       |
| `typescript`         | Type-checking only (`tsc --noEmit`)                |
| `openapi-typescript` | Generates TypeScript types from OpenAPI JSON specs |

### Packages we explicitly do NOT use

- **No `dotenv`** — Bun loads `.env` automatically.
- **No `execa`** — Use `Bun.spawn()` or `Bun.$` shell.
- **No `express` or `http`** — Use `Bun.serve()` for HTTP servers.
- **No `ink` or `react`** — We use the lighter stack of `ora` + `prompts` + `chalk`.
- **No `commander` or `clipanion`** — We use `yargs`.
- **No `cosmiconfig`** — Config file resolution is hand-rolled to match the existing Go CLI behavior.

---

## Project Structure

This is a Bun workspace monorepo with four packages:

- **`@bunny.net/openapi-client`** (`packages/openapi-client/`) — Standalone, type-safe OpenAPI client for bunny.net, generated from OpenAPI specs. Zero CLI dependencies. Publishable to npm.
- **`@bunny.net/app-config`** (`packages/app-config/`) — Shared app configuration schemas (Zod), inferred types, JSON Schema generation, and API conversion functions. Used by the CLI and potentially other tools.
- **`@bunny.net/database-shell`** (`packages/database-shell/`) — Standalone interactive SQL shell for libSQL databases. Framework-agnostic REPL, dot-commands, formatting, masking, and history. Also usable as a standalone CLI (binary: `bsql`).
- **`@bunny.net/cli`** (`packages/cli/`) — The CLI. Depends on `@bunny.net/openapi-client`, `@bunny.net/app-config`, and `@bunny.net/database-shell`.

```
bunny-cli/
├── packages/
│   ├── openapi-client/                   # @bunny.net/openapi-client package
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── redocly.yaml                  # Multi-spec config for openapi-typescript
│   │   ├── specs/                        # OpenAPI specs (committed, JSON)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BunnyWay/cli](https://github.com/BunnyWay/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
