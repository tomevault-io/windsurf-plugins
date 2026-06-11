---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Read [docs/agent-notes.md](docs/agent-notes.md) before non-trivial work.** It
> holds the durable, hard-won working knowledge that isn't derivable from the code
> — the maintainer's conventions (tests-with-features, robust-over-cosmetic,
> toggle-label and theming rules), the vault topology, how to verify live-preview
> behaviour via the Obsidian CLI, screenshot procedure, Obsidian styling traps,
> the release flow, and shipped-feature design history. **When you learn something
> durable, append it there in the same change** — that file, not ephemeral session
> memory, is the shared long-term record.

## Project

Obsidian community plugin (`id: journal-folder`) that adds folder-based journaling utilities. Any folder in a vault can act as a journal — notes named `YYYY-MM-DD`, `gggg-[W]ww`, `YYYY-MM`, or `YYYY` are recognized as daily/weekly/monthly/yearly entries. Quarterly notes (`YYYY-Q[1-4]`) are an **opt-in** fifth tier gated by the `quartersEnabled` setting; they slot between yearly and monthly when on. The vault root is *not* supported as a journal folder due to Obsidian link-resolution behavior.

The plugin is built with TypeScript + Svelte 5 (runes API, `$props`, `$state`, `$derived`) and bundled with esbuild into a single `main.js`.

## Commands

```bash
npm run dev        # esbuild in watch mode → main.js (inline sourcemap)
npm run build      # tsc --noEmit type check, then production esbuild
npm test           # Vitest run (one-shot)
npm run test:watch # Vitest in watch mode
npm run lint       # eslint-plugin-obsidianmd — the Obsidian community-review ruleset
npm run lint:fix   # same, auto-fixing what's safe
npm run version    # bump manifest.json + versions.json from package.json version
```

Tests live in `tests/` and mirror the `src/` layout. **Linting uses `eslint-plugin-obsidianmd`** (flat config in `eslint.config.mjs`) — the *same* ruleset the Obsidian community-review scanner runs against a submitted release, so `npm run lint` reproduces those findings locally. It covers both `.ts` and `.svelte` sources, type-aware in both cases: `.ts` against `tsconfig.json` (scoped to `src`; tests are excluded from the tsconfig), `.svelte` via `svelte-eslint-parser` against `tsconfig.eslint.json` (an ESLint-only tsconfig that adds the `.svelte` files — the build's `tsc` never sees them). Note `prefer-active-doc` only flags bare `document`; the matching `activeWindow` convention for viewport reads/listeners is *not* lint-enforced (see `docs/agent-notes.md`). Run lint before tagging a release to avoid surprises in the published review. Prettier config also exists and is run manually if desired.

## Architecture

### Plugin shell → feature set

Entry point is `src/plugin/journal-folder-plugin.ts`. It instantiates a `PluginFeatureSet` and registers its features in order:

- `JournalFolderSettingsFeature` — owns the global settings, persists them via `plugin.saveData`/`loadData`, registers the settings tab, propagates settings to the other features, and applies global side-effects (`applyStartOfWeek`, the body class for *Hide journal-folder.md in file explorer*). Its `saveSettings` is `public readonly` so the sidebar feature can mutate global settings (e.g. *Set as default*) through the same pipeline.
- `JournalHeaderFeature` — registers the `journal-header` markdown code block processor that mounts the in-note header + calendar.
- `JournalTasksFeature` — registers the `journal-tasks` markdown code block processor and owns a shared `TaskCache` that the sidebar's task panel also consumes. See the *Tasks* subsection below and [docs/tasks-design.md](docs/tasks-design.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chfourie/obsidian-journal-folder](https://github.com/chfourie/obsidian-journal-folder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
