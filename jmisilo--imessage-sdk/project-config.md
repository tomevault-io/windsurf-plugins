---
trigger: always_on
description: This file provides repository context and working rules for AI coding assistants contributing to
---

# AGENTS.md

This file provides repository context and working rules for AI coding assistants contributing to
`imessage-sdk`.

## Project overview

`imessage-sdk` is a provider-neutral, ESM-only TypeScript conversation layer for iMessage
infrastructure. It normalizes provider messages, conversations, events, capabilities, and errors
while preserving provider-specific functionality on the concrete provider object.

- Repository: https://github.com/jmisilo/imessage-sdk
- License: MIT
- Package manager: pnpm workspaces
- Build tool: tsup
- Test framework: Vitest
- Release tooling: Changesets

The architectural boundary is:

```text
Provider APIs
    -> @imessage-sdk/<provider>
    -> imessage-sdk public provider contract and client
    -> adapters and integrations
```

The core package must not contain provider-specific request logic. Adapters and integrations must
depend on the public `imessage-sdk` interface rather than provider internals.

## Repository structure

| Directory                       | Purpose                                                               |
| ------------------------------- | --------------------------------------------------------------------- |
| `packages/imessage-sdk`         | Provider-neutral core package (`imessage-sdk`)                        |
| `packages/providers/<provider>` | Independently published provider package                              |
| `packages/providers/README.md`  | Cross-provider feature support matrix                                 |
| `packages/chat-adapter`         | Chat SDK integration (`@imessage-sdk/chat-adapter`)                   |
| `packages/cli`                  | Provider-neutral CLI (`imessage-cli`)                                 |
| `examples/basic-blooio`         | Opt-in live example using only published Blooio and core APIs         |
| `test/package-consumer`         | Clean TypeScript consumer used by package smoke tests                 |
| `.changeset`                    | Changesets configuration, prerelease state, and pending release notes |
| `.github/workflows`             | CI and automated release workflows                                    |
| `RELEASING.md`                  | Maintainer release and registry setup guide                           |

The repository does not use Turborepo. Workspace membership is defined only by
`pnpm-workspace.yaml`.

## Package relationships

```text
@imessage-sdk/<provider>     -> imessage-sdk
@imessage-sdk/chat-adapter   -> imessage-sdk
imessage-cli                 -> imessage-sdk + every official provider
```

Workspace packages import package names, never source files from another package:

```ts
import { blooio } from '@imessage-sdk/blooio';
import { createIMessageClient } from 'imessage-sdk';
```

Use `workspace:^` for runtime dependencies on other publishable workspace packages. Do not add
TypeScript path aliases that allow package boundaries to be bypassed.

## Development setup

Requirements:

- Node.js `^20.19.0`, `^22.13.0`, or `>=24`
- pnpm `10.18.3`

Install and build from the workspace root:

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm build
```

Do not replace pnpm with npm, Yarn, or Bun for workspace operations. Keep the shared
`pnpm-lock.yaml` current when dependencies change.

## Development commands

### Root commands

| Command                 | Purpose                                                        |
| ----------------------- | -------------------------------------------------------------- |
| `pnpm build`            | Build every package that defines a build script                |
| `pnpm typecheck`        | Type-check every package that defines a typecheck script       |
| `pnpm test`             | Run unit tests across the workspace; live tests remain skipped |
| `pnpm lint`             | Run ESLint and check Prettier formatting                       |
| `pnpm format`           | Apply Prettier and import sorting                              |
| `pnpm package:check`    | Validate packed packages and a clean TypeScript consumer       |
| `pnpm changeset`        | Create release metadata for changed public packages            |
| `pnpm changeset status` | Inspect pending package version changes                        |

### Package-scoped commands

Use pnpm filters instead of changing unrelated packages:

```bash
pnpm --filter imessage-sdk build
pnpm --filter @imessage-sdk/blooio test
```

Provider packages also expose opt-in live integration tests. These contact real services and may
send messages or mutate provider state. Never run them unless the user explicitly authorizes the
live operation and the required credentials and test targets are already configured.

## Public API and import rules

The public surface of each package is defined by its `package.json` `exports` map.

- Core APIs and types come from `imessage-sdk`.
- Concrete providers come from their own packages, such as `@imessage-sdk/blooio`.
- Do not restore provider subpath exports such as `imessage-sdk/providers/<provider>`.
- Do not expose `src`, `dist` internals, mappers, transport helpers, or test utilities.
- Do not use deep imports across package boundaries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmisilo/imessage-sdk](https://github.com/jmisilo/imessage-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
