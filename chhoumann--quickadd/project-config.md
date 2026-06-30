---
trigger: always_on
description: QuickAdd is an Obsidian community plugin that provides four choice types:
---

# Repository Guidelines

## Project Overview
QuickAdd is an Obsidian community plugin that provides four choice types:
templates, captures, macros, and multis.

## Project Structure & Module Organization
QuickAdd is an Obsidian community plugin. Source code lives in `src/`: core logic under `engine/`, `services/`, and `utils/`; Svelte UI in `src/gui`; shared types in `src/types`; settings entry in `src/quickAddSettingsTab.ts`. Bundled artifacts `main.js` and `styles.css` stay at the repo root and should be generated, not hand-edited. Place tests and stubs in `tests/`, and keep user-facing docs in `docs/`.

## Tooling & GitHub
- Use `pnpm` for package management and scripts. Avoid npm/yarn/bun.
- Use the GitHub CLI (`gh`) for issues, PRs, and releases.
- When resolving a GitHub issue, use `gh issue develop <issue-number>` to
  create/link the working branch before implementation.
- GitHub does not allow approving your own PR from the same account; do not
  block merge waiting for self-approval.

## Build, Test, and Development Commands
- `pnpm run dev`: watch-mode bundle via `esbuild.config.mjs`, regenerating `main.js` as you edit.
- `pnpm run build`: run `tsc --noEmit` then produce the production bundle.
- `pnpm run build-with-lint`: type-check, run ESLint (`pnpm lint`), then produce the production build; use for release packaging.
- `pnpm run lint`: apply ESLint to TypeScript sources to catch type and usage issues.
- `pnpm run test`: execute Vitest with `--passWithNoTests` for fast local verification.

## Coding Style & Naming Conventions
The project uses tab indentation and LF endings (see `.editorconfig`); align editor settings. Use camelCase for variables and functions, PascalCase for classes and Svelte components, and kebab-case for directories and utilities. Preserve the hand-ordered imports in `src/main.ts`; disable auto-sorting there. Prefer type-only imports and route logging through the `logger` utilities for consistent output.

## Testing Guidelines
Vitest (configured in `vitest.config.mts`) runs under jsdom and cannot load real Obsidian modules. Structure production code so Obsidian dependencies are injected behind interfaces; unit tests target pure logic and swap in adapters or `tests/obsidian-stub.ts`. Co-locate specs with their source or group them under `tests/feature-name`. Add regression coverage for bug fixes, and ensure `pnpm run test` passes before pushing.

## Commit & Pull Request Guidelines
Follow Conventional Commits (`feat:`, `fix:`, `test:`, `release(version): ...`) so semantic-release can determine versions. Keep generated files in the same commit as the changes that produced them. Pull requests must include a concise summary, reproduction steps or screenshots for UI changes, linked issues when relevant, and explicit notes on release or migration impact. Request review from maintainers closest to the touched area.

## Documentation Versioning
Docs live in `docs/` and use Docusaurus with versioned documentation. The current (unreleased) docs are in `docs/docs/`, while stable snapshots live in `docs/versioned_docs/version-X.Y.Z/`.

**When releasing a new version:**
```bash
cd docs && pnpm run docusaurus docs:version X.Y.Z
```
This snapshots `docs/docs/` as the new stable version. Update `docs/docusaurus.config.js` to set `lastVersion` to the new version and add an entry under `versions`.

**Structure:**
- `docs/docs/` → "Next" (unreleased, shows warning banner)
- `docs/versioned_docs/version-X.Y.Z/` → stable release docs
- `docs/versions.json` → list of versioned snapshots
- `docs/versioned_sidebars/` → sidebar configs for each version

Keep docs in sync: update `docs/docs/` when adding features, and snapshot when releasing.

## Agent Playbook
Automation or scripted work should surface disruptive operations in the PR description and rerun `pnpm run build-with-lint` to keep `main.js`, `manifest.json`, and `versions.json` synchronized. Treat unexpected diffs in those artifacts as blockers until a maintainer approves.

## Dev workflow
Always use the `obsidian` CLI to test changes. Use the shared `dev` vault in
the main checkout, and use the isolated worktree wrapper in Codex worktrees.

Obsidian CLI is a command line interface that lets you control Obsidian from your terminal for scripting, automation, and integration with external tools.

Anything you can do in Obsidian can be done from the command line. Obsidian CLI even includes developer commands to access developer tools, inspect elements, take screenshots, reload plugins, and more.

## Obsidian Dev Vault Workflow
- Always target the `dev` vault when using the Obsidian CLI by passing
  `vault=dev` as a prefix argument before the command:
  `obsidian vault=dev <command> ...`.
- Critical: do not use suffix form (`obsidian <command> vault=dev ...`).
  It may resolve to the wrong vault due to CLI parsing behavior.
- Dev vault root path: `/Users/christian/Developer/dev_vault/dev/`.
- QuickAdd plugin path in the vault:
  `/Users/christian/Developer/dev_vault/dev/.obsidian/plugins/quickadd`.
- Run `pnpm run dev` in this repository to generate/update `main.js` for
  development.
- Reload QuickAdd after build/deploy with:
  `obsidian vault=dev plugin:reload id=quickadd`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chhoumann/quickadd](https://github.com/chhoumann/quickadd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
