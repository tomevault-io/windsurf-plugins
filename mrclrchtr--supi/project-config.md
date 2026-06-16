---
trigger: always_on
description: SuPi (**Super Pi**) is a curated extension repo for the [pi coding agent](https://github.com/earendil-works/pi) (`@earendil-works/pi-coding-agent`).
---

# CLAUDE.md

SuPi (**Super Pi**) is a curated extension repo for the [pi coding agent](https://github.com/earendil-works/pi) (`@earendil-works/pi-coding-agent`).

It is a pnpm workspace monorepo of installable pi extensions. pi loads the extensions directly as TypeScript — there is no build step.

## Development status

SuPi is pre-release and not API-stable. Intentional breaking changes to package APIs, commands, configuration formats, and extension behavior are allowed when they improve the design. Do not treat backwards compatibility as a blocker unless a task explicitly asks for it.

## Pi docs-first rule

- Never assume pi harness APIs, behavior, or conventions from memory or model priors.
- Before changing code or giving guidance about pi-specific behavior, read the relevant installed pi docs (`README.md`, matching files in `docs/`, and `examples/`) and follow linked `.md` cross-references.
- Start with `docs/index.md` for getting an overview of PI's docs.

## Documentation expectations

- Add JSDoc for exported APIs, config surfaces, and non-obvious behavior; skip boilerplate for trivial private code.
- Add inline JSDoc for complex internal logic where a short explanation would help maintainers.

## Package layout convention

- Follow `docs/package-layout.md` for repo-wide package structure.
- Standardize package boundaries with `src/api.ts`, `src/index.ts`, and `src/extension.ts` when the package role requires them.
- Prefer package-level tests in `__tests__/unit/` and `__tests__/integration/`, with `__tests__/helpers/` and `__tests__/fixtures/` as needed.
- Prefer domain folders over catch-all names like `core/`, `shared/`, or `misc/`.
- Keep small packages flat; add `config/`, `tool/`, `ui/`, `session/`, or other domain folders only when the package size and responsibilities clearly justify them.
- Current anchor examples: `supi-lsp` uses the hybrid large-package model; `supi-insights` uses the standard package-level test layout.
- This convention is the default for new packages and for existing packages when they receive structural work.
- Packages that should stay flat unless they grow: `supi-bash-timeout`, `supi-context`, `supi-debug`, `supi-rtk`, `supi-test-utils`.
- `supi-web` should stay mostly flat, but may use `src/tool/` for per-tool guidance files and other narrowly scoped tool-specific wiring.

## Commands

See `pnpm run` for routine build/lint/test. Toolchain versions pinned in `.mise.toml`.

- When both standard and `*:ai` scripts exist, prefer the `*:ai` variant for agent runs — they produce lower-noise, more token-efficient output.
- Current root examples: `biome:ai`, `typecheck:ai`, `test:ai`, `check:ai`, `verify:ai`.
- Use the non-`:ai` variant when you specifically want prettier or interactive local output.
- **After changes, run `pnpm verify:ai`** — typecheck, lint, tests in one pass. Prefer over individual checks.

## Architecture

This repo has two install surfaces:
- repository root `package.json` exposes a `pi` manifest for local-path and git installs — supports `extensions`, `prompts`, `skills`, `themes` keys
- each `packages/supi-*` is installable independently

## Package tiers

All packages are published independently. There is no meta-package — each package ships its own dependencies directly.

- Packages that depend on other `@mrclrchtr/supi-*` packages must list them in both `dependencies` and `bundledDependencies`. This applies to packages that still ship `pi.extensions` (installable pi packages). Library-only packages (no `pi.extensions`, no `./extension` export) are regular npm dependencies and do not need bundling — transitive npm resolution is sufficient for them.
- Packages that bundle `@mrclrchtr/supi-*` dependencies must reference their extension entrypoints in `pi.extensions`.

New packages should be added to the root `package.json` `pi.extensions` array for development convenience.

## Packaging conventions

- Every published SuPi pi-package exposes an explicit `./extension` export. Packages with a reusable library API expose an explicit `./api` export (optional — omit when there is no library surface). Do not rely on package-root (`.`) imports or cross-package `src/...` deep imports.
- `supi-core` is the exception — it is a library-only package with no pi extension surface, no `./extension` export, and no `pi.extensions` entry. Other SuPi packages bundle it for the library API only.
- `pi.extensions` / `pi.prompts` / `pi.skills` / `pi.themes` manifest entries must remain **real package-relative file paths**. Do not replace them with `exports` aliases.
- Any SuPi package that depends on another `@mrclrchtr/supi-*` package must list it in both `dependencies` and `bundledDependencies`. Per [pi packages docs](https://github.com/earendil-works/pi/blob/main/docs/packages.md), pi packages that depend on other pi packages must be bundled in the tarball — npm transitive dependency resolution is not guaranteed by pi's module isolation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrclrchtr/supi](https://github.com/mrclrchtr/supi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
