---
trigger: always_on
description: This is a pnpm monorepo of native Pi packages. Each directory under `packages/`
---

# Repository Guidelines

This is a pnpm monorepo of native Pi packages. Each directory under `packages/`
has its own manifest and tests; follow the nearest scoped guide, including
`packages/context/AGENTS.md`.

## Project Structure & Module Organization

- Workspace packages currently include `agent-teams`, `btw`, `context`,
  `keyboard`, `mattpocock`, `memory`, `monitor`, `pi-kit`, `plan-mode`,
  `recap`, `utils`, and `vision`.
- Extension code lives in `src/`, `extensions/`, or a package-root `index.ts`;
  skills, procedures, references, and bundled agents use their named folders.
- BDD scenarios are in each package's `features/`; executable tests are in
  `tests/`. Release metadata lives in `.changeset/` and `.github/workflows/`.
- `pi-kit` is the shared runtime and intentionally has no Pi manifest. `plan-mode`
  is a workspace package but is not in the current `scripts/publish-release.mjs`
  allowlist; check that script before changing release behavior.

## Build, Test, and Development Commands

```bash
pnpm install
pnpm test
npx tsc --noEmit -p tsconfig.extensions.json
```

`pnpm test` runs pytest across `packages/`; run one package with
`python3 -m pytest packages/<name>/tests/ -q`. Some packages also document a
strict per-file TypeScript command in their README. There is no separate build
step: Pi loads the TypeScript extensions and packaged resources directly.
Use `pnpm --dir packages/<name> pack --dry-run` to inspect package contents;
`pnpm pack --dry-run` at the root packs the private workspace root instead.
For the SDK example, run `pnpm example:sdk`.

## Coding Style & Naming Conventions

Use ESM TypeScript targeting Node 20 or newer, follow surrounding indentation,
and keep package, command, and tool names explicit and stable. Pi package
manifests use the `pi-package` keyword, an explicit `pi` resource declaration
when applicable, and complete `files` entries; imported Pi core packages are
peer dependencies. `pi-kit` has no `pi` field or runtime dependencies. Do not
add Claude Code plugin artifacts or Claude-only skill frontmatter. Use pnpm
commands such as `pnpm add` instead of hand-editing dependency manifests.

## Shared Runtime: pi-kit

Prefer the internal `@fradser/pi-kit` runtime for reusable helpers and shared
Pi-package infrastructure before adding duplicate code to a package. It is a
workspace runtime dependency, not a Pi package: consumer manifests use
`"@fradser/pi-kit": "workspace:*"` under `dependencies`, never
`peerDependencies`, and it has no `pi` manifest. Keep its dependency direction
one-way: pi-kit may use Node built-ins, but must not import consumer packages.
If the package is not present in the current checkout, do not invent a local
replacement or external registry dependency; record the gap and coordinate the
shared package first. Keep the release allowlist and pack/install checks in sync
when pi-kit is introduced or changed.

## Testing Guidelines

Write or update a `.feature` scenario before behavior changes, then add tests
under the package's `tests/` directory. Python tests use pytest and modules
follow `test_*.py`; runtime TypeScript harnesses belong under `tests/`, not the
repository root. Run affected package tests, `pnpm test`, and the strict
TypeScript check before opening a pull request. Include package dry-run output
when changing manifests or published files.

## Pi UI and Extension Rules

- Interactive popups use `ctx.ui.custom`; do not intercept terminal input globally.
  Respect the established `packages/btw` wrapping, scrolling, theme, and cleanup
  patterns. Keep passive widgets display-only.
- `package.json`: `"keywords": ["pi-package"]`, `"pi": { "skills": [...], "extensions": [...] }`; extensions packages declare `"peerDependencies": { "@earendil-works/pi-coding-agent": "*" }`.
- `files` must include everything that ships (`skills`/`extensions`/`procedures`/`references`/`scripts`).
- **Never** add `.claude-plugin`, `${CLAUDE_PLUGIN_ROOT}`, or Claude-only skill frontmatter (`allowed-tools`, `user-invocable`, `argument-hint`, `model`). Skill frontmatter: `name`, `description`, optional `disable-model-invocation`.

## Command menus vs skills (settled UX)

- `memory`/`btw` expose workflows as **pi menu commands** (`/memory`, `/btw`), not skills: `pi.registerCommand(...)` + `ctx.ui.select` + the full procedure embedded via `pi.sendUserMessage(..., { deliverAs: "followUp" })` with `{{PKG_DIR}}` substituted at send time. Keep this pattern; do not reintroduce per-workflow skills. (The former `git-agent` package follows the same pattern from `~/Developer/FradSer/git-agent/git-agent-pi-package`; the former `git`/`github` packages moved to pure skills in `~/Developer/FradSer/skills`.)
- Skill names are global — avoid collisions (the old `commit`/`commit-and-push` clash between `git` and `git-agent` was resolved by moving to menus; the git/github skills now live in `~/Developer/FradSer/skills`).
- Natural-language routing ("commit this", "create a PR") is preserved with small `before_agent_start` GUIDANCE blocks, not skills.

## TUI pattern — follow `@packages/btw` (canonical)

Interactive extension UI mirrors `packages/btw/src/overlay.ts`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FradSer/pi-packages](https://github.com/FradSer/pi-packages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
