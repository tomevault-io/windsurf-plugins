---
trigger: always_on
description: - This repository is a small Node.js generator for `README.md`.
---

# AGENTS.md

## Purpose

- This repository is a small Node.js generator for `README.md`.
- Source data lives in `data/*.yaml`.
- The output document is generated from `tmpl/README.md` plus fetched repository metadata.
- Main code lives in `script/index.js`, `script/utils.js`, and `script/yaml.js`.
- Package manager: `pnpm@9.4.0`.
- Module system: native ESM (`"type": "module"`).

## Rule Files

- No repo-local `.cursorrules` file was found.
- No `.cursor/rules/` directory was found.
- No `.github/copilot-instructions.md` file was found.
- No prior repo-level `AGENTS.md` existed in this repository.
- Follow this file as the repository-specific agent guide.

## Repository Shape

- `data/*.yaml`: source records for app categories.
- `tmpl/README.md`: README template with the `<!-- Apps -->` placeholder.
- `script/index.js`: generator entrypoint.
- `script/utils.js`: repo metadata fetch helpers and markdown helpers.
- `script/yaml.js`: YAML file discovery helper.
- `README.md`: generated output; do not hand-edit unless the task explicitly asks for it.

## Setup

- Install dependencies with `pnpm install`.
- Use Node.js new enough to support ESM, global `fetch`, top-level `await`, and `--env-file`.
- `pnpm build` reads remote GitHub/GitLab metadata; network access is expected.
- `pnpm dev` loads `.env` via `node --env-file .env script/index.js`.
- For GitHub API calls, set `GITHUB_TOKEN` when you need higher rate limits or authenticated access.

## Build, Lint, Test Commands

- Install: `pnpm install`
- Build README: `pnpm build`
- Dev run with `.env`: `pnpm dev`
- Lint whole repo: `pnpm lint`
- Auto-fix lint issues: `pnpm lint:fix`
- Pre-commit hook installs via `pnpm install` and runs `pnpm lint-staged`
- Staged-file lint manually: `pnpm lint-staged`

## Single-File / Targeted Commands

- Lint one file: `pnpm exec eslint script/index.js`
- Lint one YAML file: `pnpm exec eslint data/authentication.yaml`
- Auto-fix one file: `pnpm exec eslint script/utils.js --fix`
- Run generator directly: `node script/index.js`
- Run dev entry directly with env file: `node --env-file .env script/index.js`

## Test Status

- There is currently no dedicated automated test suite configured.
- No `test` script exists in `package.json`.
- No `vitest`, `jest`, `node --test`, `playwright`, or `cypress` config was found.
- There is therefore no true single-test command today.
- If a task asks for validation, use `pnpm lint` and `pnpm build` as the default verification path.
- If you add tests in the future, update this file with exact full-suite and single-test commands.

## Standard Validation Flow

- For code changes in `script/`, run `pnpm lint` first.
- For data or template changes, run `pnpm build` to regenerate `README.md`.
- For broader changes, run both `pnpm lint` and `pnpm build`.
- Inspect the resulting diff in `README.md` to ensure generated output changed as expected.
- Do not claim test coverage exists when only lint/build were run.

## Generated File Rules

- `README.md` is generated output.
- Prefer editing `tmpl/README.md`, `data/*.yaml`, or generator scripts instead of hand-editing `README.md`.
- If `README.md` must change, regenerate it with `pnpm build` unless the task explicitly wants a manual edit.
- Keep the `<!-- Apps -->` placeholder intact in `tmpl/README.md`.

## Data File Conventions

- Category source files use kebab-case names such as `file-sharing.yaml` and `a-i.yaml`.
- Each YAML file contains a top-level array.
- Each item should at minimum contain `name` and `url`.
- `description` is optional and falls back to repository metadata when absent.
- Preserve simple, flat records; do not introduce nested shapes unless the generator is updated first.
- Keep entries readable and consistent across files.

## Import Conventions

- Use ESM `import` / `export` only.
- Order imports as: Node built-ins, third-party packages, then local relative imports.
- Keep one import per line.
- Prefer named exports for shared helpers.
- Match existing style: built-ins use the `node:` prefix.
- Do not switch files to CommonJS.

## Formatting Conventions

- Follow ESLint as the source of truth; the repo uses `@antfu/eslint-config` with formatters enabled.
- Use 2-space indentation.
- Use single quotes.
- Omit semicolons.
- Keep trailing commas in multiline arrays, objects, and calls when lint expects them.
- Prefer concise arrow functions where readability stays high.
- Avoid unnecessary blank lines, but keep logical sections separated.

## Naming Conventions

- Use `camelCase` for variables and functions.
- Use `PascalCase` only for user-facing derived display labels such as category names.
- Use descriptive helper names like `getRepoInfo`, `getBadge`, and `getYAMLFiles`.
- Keep file names lowercase; use hyphenated names for YAML categories.
- Avoid single-letter variable names except in tiny callback scopes.

## Types and Data Shape Guidance

- The codebase is plain JavaScript, not TypeScript.
- Preserve the existing lightweight style; do not introduce TypeScript unless explicitly requested.
- Make data contracts obvious through naming and small helper functions.
- When adding richer data fields, update both YAML producers and markdown generation logic together.
- Prefer returning simple plain objects from helpers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miantiao-me/awesome-homelab](https://github.com/miantiao-me/awesome-homelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
