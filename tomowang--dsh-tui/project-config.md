---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## What this is

`@tomowang/dsh-tui` is an out-of-tree terminal front door for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (`dsh`). It stacks on `@deepseek-ai/dsh-base` the same way `dsh-web-app`/`dsh-headless` do, but drives the agent from a TTY instead of a browser. It is simultaneously:

- a **Cordis plugin** (`src/index.ts`, name `tui`) that owns terminal input/presentation only — agent lifecycle, session persistence, tool execution, and model policy stay in `dsh-base`;
- a **dsh bundle** (`package.json`'s `dsh.bundle.patch` points at `cordis.patch.yml`) that inserts `tui-startup` and `tui` into the plugin tree and disables `hmr` (module-reload redraws would fight the interactive terminal).

The terminal layer (`src/tui/`) is built on [`@earendil-works/pi-tui`](https://www.npmjs.com/package/@earendil-works/pi-tui), a standalone differential-rendering TUI library — not React/Ink. There is no JSX anywhere in this repo (every file under `src/tui/` is plain `.ts`); components are plain classes implementing pi-tui's `Component` interface (`render(width): string[]`, `invalidate()`, an optional `handleInput(data)`), mutated directly rather than re-rendered from props.

## Commands

```sh
pnpm install
pnpm run build        # tsc -p tsconfig.json → lib/
pnpm run typecheck     # tsc --noEmit
pnpm run lint          # eslint .
pnpm run test          # vitest run
```

A Husky `pre-push` hook (`.husky/pre-push`, wired via the `prepare` script) runs lint, typecheck, test, and build before every push, so failures surface locally instead of in CI. To exercise the TUI end-to-end, point a `dsh` profile's dependency at this checkout and rebuild before each run — profiles load the built `lib/` under plain Node, not `src/`:

```sh
dsh plugin --profile tui add /path/to/dsh-tui
pnpm run build
dsh --profile tui
dsh --profile tui --resume <sessionId>
dsh --profile tui --dump-config   # inspect the composed plugin tree
```

Both stdin and stdout must be real TTYs — `apply()` in `src/index.ts` throws loudly otherwise instead of degrading, so pipes/CI must use `dsh --profile headless`.

## Keep docs in sync

Before committing a feature (new/changed command, keybinding, overlay, or user-visible behavior), check whether `README.md` and this file describe the old behavior and update them in the same commit — not as a follow-up. `README.md` covers user-facing surface (Features, the terminal-commands/keyboard-shortcuts tables); this file's `## Architecture` and `## Current status / roadmap` cover implementation notes and the known-gaps list. A shipped feature still listed as a "known gap" here, or a new `/command`/shortcut missing from README's tables, counts as an incomplete change.

## Commit scopes

Every commit uses a [Conventional Commits](https://www.conventionalcommits.org/) `type(scope): subject`, drawn from this closed list — pick the one covering the primary area touched (a commit spanning several picks the most significant one, not a comma-joined list):

| Scope | Covers |
|---|---|
| `tui` | `src/tui/**` (components, overlays, the store, the prompt editor, theme), `src/render.ts`, `src/markdown.ts` — the terminal UI and the session-log-to-terminal formatting it renders |
| `plugin` | `src/index.ts` — Cordis plugin lifecycle, session/agent wiring, in-terminal approval/question answerers |
| `startup` | `src/startup.ts`, `cordis.patch.yml` — CLI flag parsing and the bundle's plugin-tree patch |
| `release` | Versioning, `CHANGELOG.md`, `cliff.toml`, the npm-publish workflow |
| `deps` | Dependency-only changes not tied to a feature/fix |
| `ci` | GitHub Actions workflows other than the release one |
| `repo` | Repo-wide contributor/process docs not about one code area (e.g. this scope table) |

A docs-only commit still scopes to the subject it documents (`docs(tui): …` for a README/AGENTS.md update about the terminal UI), not a generic catch-all. `git-cliff` groups the changelog by `type` (see `cliff.toml`'s `commit_parsers`), not `scope`, so scope is for readability/`git log --grep`, not changelog sectioning.

## Architecture

**Two-plugin split (`src/startup.ts` → `src/index.ts`).** `tui-startup` parses this app's CLI flags (everything after the launcher's own args) via `dsh-cmdline`/commander and publishes them as an ordinary Cordis service (`TUI_STARTUP_SERVICE` / `tuiStartup`). The `tui` plugin injects that service rather than parsing argv itself — this mirrors how `dsh-headless` is structured and keeps startup-value resolution lazy/testable. The wiring between the two is declared in `cordis.patch.yml`, not in code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomowang/dsh-tui](https://github.com/tomowang/dsh-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
