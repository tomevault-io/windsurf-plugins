---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, etc.) when working
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, etc.) when working
with code in this repository.

## Common Commands

The package manager is **yarn** — every script in `package.json` and every CI
workflow uses it.

- `yarn test` — run the full test suite (vitest, see `vitest.config.ts`).
- `yarn test src/lib/__tests__/commit-message.test.ts` — run a single test file.
- `yarn test -t "<name pattern>"` — run a single test case by name.
- `yarn test:e2e` — end-to-end test: build the bundle, launch VS Code in a temp
  git repo, drive the conventional-commits flow non-interactively, assert a real
  commit landed. Sources live under `e2e-test/`. The first run downloads a VS
  Code build into `.vscode-test/` (network access required).
- `yarn watch` — `tsc -watch` for incremental type-checking during development.
- `yarn webpack` — bundle the extension to `dist/extension.js` in development
  mode.
- `yarn webpack-dev` — same as `yarn webpack` but with `--watch`.
- `yarn build` — runs `node prepare.js` then `vsce package --yarn`. **Side
  effect:** `prepare.js` downloads the upstream `gitmojis.json` from GitHub into
  `src/vendors/gitmojis.json`, so the first build needs network access.
- `yarn deploy` — `vsce publish --yarn` (Marketplace publish).

### Launching the extension host (manual testing)

Use VS Code's **Run Extension** launch task. Per the README's Contribution note:

1. Install the
   [`vscode-tsl-problem-matcher`](https://github.com/eamodio/vscode-tsl-problem-matcher)
   VS Code extension once — the task depends on it.
2. After editing source, **restart the task** (kill and relaunch) so the rebuilt
   bundle is picked up. A simple reload of the host window is not enough.

## Repo Conventions

- Husky hooks live in `.husky/`:
  - `pre-commit` runs `yarn lint-staged` (prettier on staged files matching
    `*.{js,ts,css,less,json,md,html,yml,yaml,pcss,jsx,tsx}`).
  - `commit-msg` runs `yarn commitlint --edit $1` against the conventional
    config (`@commitlint/config-conventional`).
- Releases are driven by edits to `CHANGELOG.md` on `master`; do not hand-bump
  `package.json` independently of the release flow.
- Tests live in `src/**/__tests__/**/*.test.ts` (vitest config restricts the
  glob to that path).

## Architecture

### Activation flow

`src/extension.ts` is the entry point declared by `package.json`'s `main`
(`./dist/extension.js`, produced by webpack). On `activate`:

1. Initialize logging (`src/lib/output.ts`) and localization
   (`src/lib/localize.ts`).
2. Register three commands. The main one (`extension.conventionalCommits`) wires
   `createConventionalCommits` from `src/lib/conventional-commits.ts`.
3. Register a `commit-message:` virtual filesystem provider (see Editor /
   virtual-FS below).

`createConventionalCommits` (in `src/lib/conventional-commits.ts`) is the
runtime workflow:

1. Locate the VS Code Git API and pick a repository (`getRepository` — handles
   single repo, multi-repo quick-pick, and explicit URI argument from the SCM
   menu).
2. `commitlint.loadRuleConfigs(repository.rootUri.fsPath)` — load rules + prompt
   config (see Commitlint below).
3. `prompts({...})` — drive the prompt step machine to collect a `CommitMessage`
   (see Prompt machine below).
4. `serialize(commitMessage)` from `src/lib/commit-message.ts` — turn the
   structured message into a single git commit string
   (`type(scope): gitmoji subject [skip ci]\n\nbody\n\nfooter`).
5. Write to `repository.inputBox.value`. Either auto-commit via `git.commit`, or
   hand off to the editor virtual-FS when `showEditor` is enabled.

### Prompt step machine

`src/lib/prompts.ts` builds an ordered list of `Prompt` questions (`type`,
`scope`, `gitmoji`, `ci`, `subject`, `body`, `footer`), filters by user
configuration (e.g. `promptScopes`, `gitmoji`, `showEditor` hides body/footer
because the editor handles them), and walks them with an index-based loop that
supports a Back button:

- Each step dispatches to one of three handlers in
  `src/lib/prompts/prompt-types.ts`: `QUICK_PICK` (selectable items, optional
  `noneItem`), `INPUT_BOX` (free text + `validate`), and
  `CONFIGURABLE_QUICK_PICK` (quick-pick over a workspace/global setting list
  with new-item input — used for scopes).
- A previous-step's `PromptStatus` (its `value` and `activeItems`) is preserved
  so the Back button (`QuickInputButtons.Back`) can restore it. The handlers
  throw a `{ button, value }` object when Back is pressed; the loop catches it,
  decrements `index`, and replays the prior step seeded with its previous
  answer.
- After the loop, results are folded into a `CommitMessage`
  (`src/lib/commit-message.ts`) with optional per-field `format` (e.g.
  line-break substitution).

### Commitlint integration

`src/lib/commitlint.ts` exposes a singleton `commitlint`. Key responsibilities:

- `loadRuleConfigs(cwd)` calls `@commitlint/load`'s `load({}, { cwd })` to read
  the project's commitlint config (rules + optional `prompt` metadata used to
  drive UI titles/descriptions).
- `getTypeEnum()` / `getScopeEnum()` extract enum values from the `type-enum` /
  `scope-enum` rules so the prompts can offer them as picker items.
- `lintType` / `lintScope` / `lintSubject` / `lintHeader` / `lintBody` /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vivaxy/vscode-conventional-commits](https://github.com/vivaxy/vscode-conventional-commits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
