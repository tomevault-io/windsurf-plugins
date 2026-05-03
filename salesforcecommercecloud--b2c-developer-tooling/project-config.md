---
trigger: always_on
description: This is a monorepo project with the following packages:
---

# B2C CLI

This is a monorepo project with the following packages:
- `./packages/b2c-cli` - the command line interface built with oclif
- `./packages/b2c-tooling-sdk` - the SDK/library for B2C Commerce operations; supports the CLI and can be used standalone
- `./packages/b2c-dx-mcp` - Model Context Protocol server; also built with oclif
- `./packages/b2c-vs-extension` - VS Code extension (not published to npm; packaged as VSIX and versioned via git tags)
- `./docs` - documentation site (private `@salesforce/b2c-dx-docs` workspace package; not published to npm)

## Common Commands

```bash
# Install dependencies
pnpm install

# Build all packages
pnpm run build

# Build specific package
pnpm --filter @salesforce/b2c-cli run build
pnpm --filter @salesforce/b2c-tooling-sdk run build

# Dev mode for CLI (uses source files directly)
pnpm --filter @salesforce/b2c-cli run dev
# or using convenience script
./cli
```

## Commands for Coding Agents

These commands produce condensed output optimized for AI coding agents:

```bash
# Run tests (minimal output - only failures + summary)
pnpm run test:agent

# Run tests for specific package
pnpm --filter @salesforce/b2c-cli run test:agent
pnpm --filter @salesforce/b2c-tooling-sdk run test:agent

# Lint (errors only, no warnings); Be sure to run lint before committing or writing a changeset
pnpm run lint:agent

# Type-check (single-line errors, no color)
pnpm run typecheck:agent

# Format check (lists only files needing formatting)
pnpm run -r format:check
```

## Verbose Commands (Debugging/CI)

Use these for detailed output during debugging or in CI pipelines:

```bash
# Run tests with full output and coverage
pnpm run test

# Run tests for specific package
pnpm --filter @salesforce/b2c-cli run test
pnpm --filter @salesforce/b2c-tooling-sdk run test

# Format code with prettier
pnpm run -r format

# Lint with full output
pnpm run -r lint
```

## Copyright Header

All TypeScript source files must include this exact copyright header block:

```typescript
/*
 * Copyright (c) 2025, Salesforce, Inc.
 * SPDX-License-Identifier: Apache-2
 * For full license text, see the license.txt file in the repo root or http://www.apache.org/licenses/LICENSE-2.0
 */
```

The header is enforced by eslint via `eslint-plugin-header`. The canonical definition is in `eslint.config.mjs` (root) as `copyrightHeader`.

## Setup/Packaging

- use `pnpm` over `npm` for package management
- shared dependencies are managed via [pnpm catalogs](https://pnpm.io/catalogs) — versions are defined once in the `catalog:` section of `pnpm-workspace.yaml` and referenced as `"catalog:"` in each package.json. When adding or updating a dependency that is shared across packages, update the catalog entry rather than individual package.json files.
- the `pnpm run test` commands also run the linter after tests
- use `pnpm run -r format` (or individually in packages) to format code with prettier
- use `exports` field in package.json files to define public API surface for packages; use `development` field for nodejs --conditions for development ergonomics (packages/b2c-cli/bin/dev.js will use this condition)

## Documentation

- Update docs in `./docs/` folder and relevant skills in `./skills/b2c-cli/skills/` when updating or adding CLI commands.
- When adding new SDK modules, update `docs/typedoc.json` entry points to include the new module's barrel file so API docs are generated.

See [documentation skill](./.claude/skills/documentation/SKILL.md) for details on updating user guides, CLI reference, and API docs.

```bash
# Run docs dev server (from project root)
pnpm run docs:dev

# Build docs for production (stable mode — root base path)
pnpm run docs:build

# Build docs in dev mode (/dev/ base path)
IS_DEV_BUILD=true pnpm run docs:build
```

### Docs workspace package

The `./docs` directory is a private workspace package (`@salesforce/b2c-dx-docs`) with a dependency on `@salesforce/b2c-tooling-sdk`. This exists to support doc-only releases and changelog tracking via changesets, not for npm publishing. It has no build script — `pnpm -r run build` skips it.

**Deployed URL structure:** stable/released docs are served at the root URL, dev docs (from `main`) live at `/dev/`.

**Doc-only releases:** to release documentation changes without bumping CLI/SDK/MCP, create a changeset that targets only the docs package:

```md
---
'@salesforce/b2c-dx-docs': patch
---

Improved authentication guide with step-by-step examples
```

This produces a `docs@<version>` tag and triggers a docs rebuild on merge of the version PR.

**Automatic cascade:** because the docs package depends on the SDK, when the SDK version is bumped by a changeset, `updateInternalDependencies: "patch"` auto-bumps the docs version too — triggering a docs rebuild (correct since API docs are generated from the SDK).

## Logging

- when logging use the logger instance from `@salesforce/b2c-tooling-sdk/logger` package
- CLI commands have access to this logger via `this.log` method from oclif Command class
- CLI commands can write directly to stdout/stderr if their primary purpose is to output or stream data

## CLI Command Development

See [CLI command development skill](./.claude/skills/cli-command-development/SKILL.md) for patterns.

## Claude Code Skills


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SalesforceCommerceCloud/b2c-developer-tooling](https://github.com/SalesforceCommerceCloud/b2c-developer-tooling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
