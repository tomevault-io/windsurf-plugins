---
trigger: always_on
description: Guide for coding agents working in this repository. The generated code standards that Ultracite itself enforces live in `.claude/CLAUDE.md`; this file covers how the repo is put together and how to change it safely.
---

# AGENTS.md

Guide for coding agents working in this repository. The generated code standards that Ultracite itself enforces live in `.claude/CLAUDE.md`; this file covers how the repo is put together and how to change it safely.

## What this is

Ultracite is a zero-config linting and formatting preset for JS/TS projects, published to npm as `ultracite`. It ships:

- **Presets** for three linter backends: Oxlint + Oxfmt (recommended), Biome, and ESLint + Prettier + Stylelint, each with core and per-framework variants.
- **A CLI** (`ultracite init | check | fix | doctor | upgrade`) that installs the toolchain, writes config files, wires editors, git hooks, and AI agent rules, and shells out to the chosen linter.
- **An agent skill** (`skills/ultracite`) that ships inside the npm package.

The docs site at https://www.ultracite.ai lives in this repo too.

## Repo map

```
packages/cli/                 The published `ultracite` package
  src/index.ts                Commander entry; every subcommand is registered here
  src/commands/               check, fix, doctor, upgrade
  src/initialize.ts           `ultracite init` orchestration (prompts + flags)
  src/linters/                One adapter per tool: biome, eslint, oxlint, oxfmt, prettier, stylelint
  src/integrations/           husky, lefthook, lint-staged, pre-commit
  src/agent-fix/              `fix --claude` / `fix --codex`: hand remaining diagnostics to an agent CLI
  src/data/                   Static tables: agents, editors, hooks, options, providers, rules (AGENTS.md template)
  src/dependencies.ts         Toolchain versions + peer ranges read from package.json; what `init` installs
  config/                     The presets (see "Presets" below)
  __tests__/                  bun:test suite, one file per source module, plus lint-for-real fixtures
  scripts/                    generate-dts, copy-skill, compare-rule-parity, vendor-anti-slop
  build.ts                    Bun.build -> dist/index.js (single minified ESM file, deps external)
apps/docs/                    Docs + marketing site (blume on Astro, deployed to Cloudflare Workers)
  docs/**/*.mdx               Documentation content; meta.ts files order the sidebar
  blume.config.ts             Site config, content sources, redirects under /docs/*
packages/video/               Remotion release videos (private, not linted: vendored UI)
packages/typescript-config/   Shared tsconfig bases (private)
skills/ultracite/             Source of the agent skill; copied into packages/cli/skills on pack
benchmark/                    PR-time performance regression gate for check/fix across all providers
scripts/                      validate-configs (loads every preset + runs the ESLint/oxlint parity check)
patches/                      bun patchedDependencies (currently oxfmt)
.changeset/                   Changesets; every user-facing change needs one
tmp/                          Gitignored scratch area for hand-testing the CLI against a sample project
```

## Toolchain

- **Bun 1.4.x** is the package manager, test runner, script runner, and bundler. Never use npm/yarn/pnpm here. Lockfile is `bun.lock`.
- **Turbo** fans out `build`, `test`, `types`, `dev` across workspaces.
- **TypeScript** type checks run through `tsgo` (`@typescript/native-preview`), not `tsc`. `bun run types` at the root.
- **The repo lints itself with Oxlint + Oxfmt** via `oxlint.config.ts` and `oxfmt.config.ts` at the root, which extend the shipped core, react, astro and anti-slop presets. Do not run Biome, ESLint or Prettier on repo source; those tools are dev dependencies only so the presets can be validated.
- The published CLI must run on **Node 20, 22, 24 and Bun**, on Linux and Windows. It spawns the linters rather than bundling them, so keep new runtime dependencies to a minimum and avoid Bun-only APIs in `src/`. (Scripts, tests and `build.ts` may use Bun APIs.)

## Commands

Run from the repo root unless noted.

| Task | Command |
| --- | --- |
| Install | `bun install` |
| Build the CLI | `bun run build --filter ultracite` |
| Build everything (incl. docs) | `bun run build` |
| Run all tests | `bun test` |
| Run one test file | `bun test packages/cli/__tests__/oxlint.test.ts` |
| Coverage | `bun run test:coverage` |
| Lint + format check (repo source) | `bun run check` |
| Auto-fix lint + format | `bun run fix` |
| Type check | `bun run types` |
| Validate every preset + rule parity | `bun run validate:configs` |
| Docs dev server | `bun run dev --filter docs` (or `cd apps/docs && bun dev`) |
| Benchmark a packed build | See `benchmark/README.md` |
| Add a changeset | `bun changeset` |

`bun run check` and `bun run fix` execute the CLI straight from source (`packages/cli/src/index.ts`), so they need no build step. The husky pre-commit hook runs, in order: CLI build, tests, check, types, validate:configs. Anything that fails there fails CI too.

### Before you finish a change

1. `bun run fix` (formats and autofixes), then `bun run check` must be clean.
2. `bun run types`
3. `bun test`
4. `bun run validate:configs` if you touched anything under `packages/cli/config`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haydenbleasel/ultracite](https://github.com/haydenbleasel/ultracite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
