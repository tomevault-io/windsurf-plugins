---
trigger: always_on
description: Single Obsidian plugin (TypeScript + esbuild) that bundles a longform writer's suite — projects/scenes, a compile pipeline, word goals, writing sprints, and an invisible-revision decision log — replacing the fragile Longform + Word Goals + Word Sprint stack. Inkswell owns the drafting surface: scenes are written in its own embedded Live-Preview editor (the Write panel), backed by a custom CodeMirror 6 `EditorView` — not Obsidian's Markdown editor. Single-user, local, no backend. Desktop + mobile
---

# Inkswell — Agent Guide

## What this project is
Single Obsidian plugin (TypeScript + esbuild) that bundles a longform writer's suite — projects/scenes, a compile pipeline, word goals, writing sprints, and an invisible-revision decision log — replacing the fragile Longform + Word Goals + Word Sprint stack. Inkswell owns the drafting surface: scenes are written in its own embedded Live-Preview editor (the Write panel), backed by a custom CodeMirror 6 `EditorView` — not Obsidian's Markdown editor. Single-user, local, no backend. Desktop + mobile (pandoc export is desktop-only and feature-detected).

## Stack reference
Obsidian plugin conventions (toolchain, Vault API rules, deferred views, mobile, release runbook, gotchas) → `~/.claude/docs/obsidian-plugin-dev.md`. This repo is the reference implementation of that doc; where they disagree, this file wins here and the doc should be fixed.

## Commands
| Task | Command |
|------|---------|
| Seed test vault | `npm run dev-vault:reset` (copy the pristine sample vault into the git-ignored `examples/dev-vault/`) |
| Dev (watch build) | `npm run dev` (auto-deploys to the dev scratch vault on each rebuild) |
| Production build | `npm run build` (`tsc -noEmit` then esbuild; auto-deploys to the dev scratch vault) |
| Refresh demo binaries | `npm run build:sample` (deploys into the committed `examples/sample-vault/` for packaging) |
| Deploy target | `examples/dev-vault/.obsidian/plugins/inkswell` (git-ignored) — override with `INKSWELL_VAULT`. Test here, NOT in the committed sample vault, and never in a real user vault (DKB runs the published store build). |
| Typecheck only | `npm run typecheck` |
| Lint | `npm run lint` (ESLint 9 flat config running **eslint-plugin-obsidianmd's `recommended` set** — the published encoding of the community-store automated review: all `obsidianmd/*` rules plus the type-aware checks the bot reports, e.g. `no-unsupported-api`, `no-deprecated`, `no-misused-promises`, `prefer-window-timers`. Run it before every release to mirror the review locally. The release CI gates on it. Errors block; a few non-blocking rules are tuned in `eslint.config.mjs` with rationale.) |
| Tests | `npm test` (vitest) |
| Reload plugin in vault | use the `obsidian-cli` skill, not a manual restart |

## Architecture rules
- **Finding projects:** query `app.metadataCache` for the `longform` frontmatter key. Don't walk the filesystem — use [src/projects/](src/projects/).
- **Project persistence:** rewrite ONLY the index note's frontmatter. NEVER mutate scene-file bodies — Longform's core invariant. Use the index writer in [src/projects/](src/projects/), not raw `vault.modify` on scenes.
- **Longform compatibility is the premise.** Scene indentation serializes as nested YAML arrays (see `indentedScenesToArrays`/`arraysToIndentedScenes` ported from Longform). Don't invent a flatter encoding — it breaks drop-in compatibility. Inkswell-only data goes under a `inkswell:` sub-key, never inside `longform`.
- **Frontmatter is a frozen 1.0 contract.** Every key Inkswell reads/writes is documented in [SCHEMA.md](SCHEMA.md). Don't rename or repurpose an existing key (including checkpoint/beat/task/category IDs) in a `1.x` release — new optional keys are fine; renames/removals wait for `2.0`. Update SCHEMA.md when adding a key.
- **Word counting:** import the single counter in [src/lib/wordcount.ts](src/lib/wordcount.ts). Don't write ad-hoc counters — goals/sprints/compile must reconcile.
- **Keep pure logic Obsidian-free.** Testable logic (compile assembly, goals math, revision-list ops) lives in modules with NO `obsidian` import (`assemble.ts`, `goals.ts`, `decisions.ts`); the `obsidian`-importing wrapper sits beside it. Tests can't import a module that pulls `obsidian` (no runtime in vitest).
- **Single host view.** Inkswell is ONE main-area tab (`VIEW_TYPE_INKSWELL`, `src/views/inkswell-view.ts`); Projects/Write/Stats/Revision Log are panel classes swapped inside it via a tab-bar. Add new surfaces as panels, NOT as new `ItemView` types/tabs. All entry points call `openInkswell(mode)` so only one tab ever exists. The Write panel hosts the manuscript editor in-place; `openScene` ([src/scenes/scene-actions.ts](src/scenes/scene-actions.ts)) opens a scene in a new, focused markdown tab (like Ctrl/Cmd-clicking a wikilink) for plain Obsidian editing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leethobbit/obsidian-inkswell-plugin](https://github.com/leethobbit/obsidian-inkswell-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
