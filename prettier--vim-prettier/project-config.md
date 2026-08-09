---
trigger: always_on
description: This repository has not had sustained maintenance, so do not assume the current
---

# AGENTS.md

## Compatibility Revival Spec

This repository has not had sustained maintenance, so do not assume the current
`package.json`, lockfile, Docker image, or docs accurately describe supported
Prettier, Vim, Neovim, Node, or parser-plugin versions.

The immediate goal is to turn compatibility from guesswork into a tracked,
tested support policy. Prefer small, evidence-backed changes over broad
modernization.

## Operating Rules

- Work on `dev/compatibility-revival` for this effort unless directed otherwise.
- Keep runtime fixes small and separately reviewable.
- Do not silently drop legacy support. If support is removed, document the reason
  and provide migration notes.
- Treat async formatting as data-loss-sensitive. Add tests before changing async
  buffer replacement, write behavior, or buffer-switch handling.
- Preserve project-local Prettier behavior. A user's project-local Prettier and
  config should take precedence over bundled fallback tooling.
- Do not use dependency upgrades as proof of compatibility. Every Prettier/plugin
  major bump needs fixture coverage.
- Keep README and `doc/prettier.txt` in sync when user-facing behavior changes.

## Proposed Support Matrix

Use this as the starting hypothesis, not as a final promise.

### Blocking Editor Targets

- Vim 8.2 latest patch with `+job` and `+channel`.
- Vim 9.1 or latest stable Vim.
- Neovim 0.9 latest patch.
- Neovim 0.10 or latest stable Neovim.

### Discovery-Only Editor Targets

- Vim 7.4.
- Vim 8.0 and 8.1.
- Neovim 0.4.x.

These should not be advertised as supported unless CI proves they work. The code
already uses APIs and method-call syntax that are unlikely to be Vim 7 safe.

### Blocking Prettier Targets

- Prettier 3 latest as the primary target.
- Prettier 3.0.3 to validate the current 1.0.0-era package baseline.
- Prettier 2.8.8 as a project-local legacy target.

### Discovery-Only Prettier Targets

- Prettier 1.x. Prefer the existing `release/0.x` path for users that need this
  unless a maintainer explicitly chooses to restore active support.

### Language Coverage

Core Prettier languages to test across Prettier 2.8.8 and 3.x:

- JavaScript, JSX, MJS, CJS.
- TypeScript and TSX.
- CSS, SCSS, Less.
- JSON and YAML.
- HTML and Vue.
- GraphQL.
- Markdown and MDX.

External plugin languages to validate primarily against Prettier 3-compatible
plugin versions:

- PHP.
- Ruby.
- XML.
- Lua.
- Svelte.

## Known Baseline Findings

- `yarn install --frozen-lockfile` succeeds, but current PHP and Svelte parser
  plugin versions report Prettier 1/2 peer compatibility while the package ships
  Prettier 3.0.3.
- `yarn test` on Vim 9.1 originally failed all formatting tests with
  `E930: Cannot use :redir inside execute()`, caused by version detection in
  `autoload/prettier/resolver/config.vim` calling `prettier#PrettierCli()` under
  `redir`.
- `yarn lint` now uses pinned local Python requirements and a checkout-local
  virtualenv wrapper for `vim-vint`, rather than relying on global `vint`.
- No GitHub Actions workflow currently defines a compatibility matrix.
- The Dockerfile is old and uses Alpine 3.8 plus unpinned `testbed/vim:latest`.
- Docs and runtime filetype support have drifted.

## Current Branch Progress

- Fixed the Vim 9 `redir` failure by making Prettier CLI version detection call
  the resolved executable directly instead of routing through `:PrettierCli`
  inside `redir`.
- Added a focused vim-driver regression test for config version detection inside
  `execute()`.
- Made the Jest/Vim harness use `tests/vimrc` with `-u NONE` so local tests load
  this checkout instead of a user's installed vim-prettier runtime.
- After those fixes, `yarn test` on local Vim 9.1 reaches formatting assertions:
  15 tests pass and 16 fail.
- Remaining core snapshot failures are compatibility deltas rather than the
  original Vim 9 harness failure: GraphQL, SCSS, YAML, and Vue differ from the
  stored snapshots under the current Prettier 3.0.3 toolchain.
- Remaining external plugin failures are no-op formatting for Lua, PHP, Ruby,
  and XML, consistent with stale or unresolved bundled parser plugins.
- Added an initial GitHub Actions smoke workflow. It runs the Vim 9 config
  version regression as a blocking check and runs the full formatting suite as
  non-blocking discovery until plugin compatibility is restored.
- Updated targeted Prettier 3.0.3 snapshots for GraphQL, SCSS, Vue, and YAML
  after confirming those were core formatter deltas.
- Added explicit `plugins` CLI config support and scoped automatic bundled
  plugin loading for PHP and XML to Prettier executables under this checkout.
  PHP and XML fixture tests now pass with the current bundled packages.
- Lua and Ruby fixture tests still fail as no-op formatting. Current bundled Lua
  and Ruby plugin versions are not viable Prettier 3 targets and need separate
  support decisions before being advertised.
- Split formatting tests into a blocking known-passing lane and a quarantined
  Lua/Ruby lane, so CI no longer hides regressions in already-passing fixtures
  behind expected plugin-language failures.
- CI now runs an explicit `vim --version` capability check for `+job` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prettier/vim-prettier](https://github.com/prettier/vim-prettier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
