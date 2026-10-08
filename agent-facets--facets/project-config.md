---
trigger: always_on
description: A Turborepo monorepo, built and tested with Bun, that ships the `facet`
---

# Agent Facets

A Turborepo monorepo, built and tested with Bun, that ships the `facet`
CLI and the packages it is assembled from.

This file is the **map**. It carries only what applies to the whole
repository. Anything that is true of exactly one package lives in that
package's own `AGENTS.md` — follow the link before working in a
package.

## Running Bun commands

Sometimes `bun` is not available in the shell. If that happens prefix the `bun`
commands with `mise exec --` like the following: `mise exec -- bun format`.

Mise will ensure you find the `bun` executable correctly.

## Architecture

Three layers, plus the SDK that third parties implement against:

- **protocol** (Layer 1) — the facet artifact specification. Schemas,
  integrity, content hashing, archive format, materialization planning.
  Runs on Node; published to npm.
- **engine** (Layer 2) — one concrete implementation of that spec on a
  developer's machine. Install/update/remove pipelines, registry client,
  cache, filesystem transaction. Bun-native; never published.
- **cli** (Layer 3) — argument parsing, Ink views, error formatting,
  exit codes. Replaceable skin over the two layers below it.
- **adapter** — the SDK an adapter implements to teach the pipeline about
  an AI coding tool. Published; must stay a leaf.

The test for where code goes: **if `engine` were rewritten in Rust
tomorrow, would this line change?** No means protocol. Yes means engine.
If it is intrinsically about a terminal, it is cli.

## Workspaces

`packages/*` and `packages/adapters/*` (root `package.json`). Eleven
packages; seven publish to npm.

| Package | npm | Purpose | Rules |
|---|---|---|---|
| `packages/protocol` | public | Reference implementation of the facet spec | [AGENTS.md](packages/protocol/AGENTS.md) |
| `packages/engine` | no | CLI machinery: install, registry, cache, fs transaction | [AGENTS.md](packages/engine/AGENTS.md) |
| `packages/cli` | public (`agent-facets`) | The `facet` binary; Ink TUI | [AGENTS.md](packages/cli/AGENTS.md) |
| `packages/adapter` | public | Adapter SDK (`defineAdapter`) | [AGENTS.md](packages/adapter/AGENTS.md) |
| `packages/common` | no | Primitives shared across published packages | [AGENTS.md](packages/common/AGENTS.md) |
| `packages/brand` | public | Color, theme, and font tokens | [AGENTS.md](packages/brand/AGENTS.md) |
| `packages/adapter-jsonc` | no | JSONC parse/edit helpers for adapters | [AGENTS.md](packages/adapter-jsonc/AGENTS.md) |
| `packages/adapter-test-kit` | no | Shared adapter conformance fixtures | [AGENTS.md](packages/adapter-test-kit/AGENTS.md) |
| `packages/adapters/claude-code` | public | First-party adapter | [AGENTS.md](packages/adapters/AGENTS.md) |
| `packages/adapters/codex` | public | First-party adapter | [AGENTS.md](packages/adapters/AGENTS.md) |
| `packages/adapters/opencode` | public | First-party adapter | [AGENTS.md](packages/adapters/AGENTS.md) |

Do not maintain a directory listing here. Each package's `src/index.ts`
and its own `AGENTS.md` are the source of truth for what it contains.

### Other directories

| Directory | Purpose | Rules |
|---|---|---|
| `docs/` | Mintlify documentation site | [AGENTS.md](docs/AGENTS.md) |
| `scripts/` | Release and repo tooling | [AGENTS.md](scripts/AGENTS.md) |
| `.circleci/` | Packed CircleCI configs | [AGENTS.md](.circleci/AGENTS.md) |
| `openspec/` | Change management (specs, changes, schemas) | — |

## Runtime

There is no single runtime rule. It is per package:

- **`protocol`** must run on **Node 22+ with no Bun on `$PATH`**. No
  `Bun.*` globals, no subprocesses, no network, no filesystem writes.
- **`engine`, `cli`, `scripts/`** are Bun-native. Use the `Bun.*`
  globals (`Bun.file`, `Bun.spawn`, `Bun.Glob`, `Bun.gzipSync`), never
  `import ... from 'bun'`.
- **`adapter`, `brand`, `common`** are published or bundled into
  published artifacts and stay runtime-agnostic.

`node:*` imports are not a smell. Engine's filesystem transaction is
built on `node:fs` deliberately (`packages/engine/src/fs/syscalls.ts`),
because it needs syscall-level control that `Bun.file` does not expose.

Beyond that, Bun is the toolchain: `bun install`, `bun run <script>`,
`bun test`, `bun <file>`. Libraries build with `tsdown`; the CLI binary
builds with `bun build --compile`.

OpenSpec commands MUST run as `bun openspec ...`, never `npx openspec ...`.

## Errors are values, not control flow

**Do not throw errors to model expected outcomes.** Return discriminated
union results instead. Throwing is for genuinely unexpected, unrecoverable
conditions (programmer bugs, environment-level failures the function has
no contract for). Anything a caller can reasonably handle — validation
failures, missing files, cache misses, integrity mismatches, parse
errors, lock contention — is part of the function's *contract* and
belongs in its return type.

This is not a stylistic preference. Thrown errors are invisible to the
type system: TypeScript cannot tell you which functions throw, what they
throw, or whether you handled it. Result types make every failure mode
a static obligation — the compiler refuses to let you forget a case.

### The pattern

```ts
// Bad — failure is invisible to the type system; caller may forget

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-facets/facets](https://github.com/agent-facets/facets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
