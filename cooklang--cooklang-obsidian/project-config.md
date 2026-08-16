---
trigger: always_on
description: These instructions apply to the entire repository.
---

# AGENTS.md

## Scope and source of truth

These instructions apply to the entire repository.

Use the current source, tests, and build configuration as the source of truth. The files under `docs/superpowers/`, plus root-level `plan.md` and `research.md`, are historical design material and can be stale. In particular, `README.md`, old design notes, and `test-recipes/curry.0.png` still show a zero-based step-image example; the current implementation, tests, and changelog use one-based names.

## Project overview

Cooklang Editor is a TypeScript Obsidian plugin for editing and rendering Cooklang recipes. It supports `.cook` files, Markdown files whose frontmatter contains the Boolean `recipe: true`, and fenced `cook`/`cooklang` blocks rendered by a Markdown reading-mode postprocessor. Rollup produces a CommonJS Obsidian bundle from `src/main.ts`.

- Runtime baseline: Obsidian 1.8.0 or newer.
- Supported platforms: desktop and mobile (`isDesktopOnly` is `false`). Do not add Node-only runtime assumptions.
- Package manager: npm with `package-lock.json`.
- CI runtime: Node 20.x.

## Setup and commands

Use Node 20.x to match CI.

| Task | Command | Notes |
| --- | --- | --- |
| Reproducible install | `npm ci` | Preferred when using the existing lockfile. |
| Add or update dependencies | `npm install` | Commit the resulting `package-lock.json` change. |
| Development build/watch | `npm run dev` | Rebuilds with Rollup on changes. |
| Full test suite | `npm test` | Runs Vitest once in the Node environment. |
| Focused test | `npm test -- src/utils/scaling.test.ts` | A filename or Vitest filter is acceptable. |
| Test watch mode | `npm run test:watch` | Interactive local use. |
| Production/type gate | `npm run build` | Runs Rollup with TypeScript `noEmitOnError`; emits `main.js` and `styles.css`. |

There is no source lint or formatting command. Do not claim one was run, and do not reformat unrelated code. A standalone `tsc --noEmit` is not the authoritative gate and can expose declaration/dependency errors outside the Rollup build; use `npm run build` for project validation.

The misspelled `instal-deps` package script is legacy and is not the canonical setup command.

## Repository map

- `src/main.ts`: plugin lifecycle, settings loading, view/extension registration, commands, menus, Markdown code-block processors, and `recipe: true` auto-detection.
- `src/cookView.ts`: the `TextFileView`, CodeMirror editor, source/preview switching, per-view checklist state, persisted mode/scale/current-step state, and rich-preview entry point.
- `src/services/ParserService.ts`: singleton initialization and access to the Cooklang WASM parser.
- `src/services/TimerService.ts`: timer intervals, Howler audio, notices, and cleanup.
- `src/renderers/`: focused Obsidian DOM renderers. `PreviewRenderer` composes the full recipe; `MarkdownRecipeRenderer` composes compact fenced-block output. Shared state and callbacks live in `renderers/types.ts`.
- `src/utils/`: mostly pure transformations and view models, with colocated `*.test.ts` files.
- `src/mode/cook/`: CodeMirror `StreamLanguage` highlighting and its tests.
- `src/settings.ts`: settings defaults and the Obsidian settings UI.
- `src/styles.scss`: editor, full-preview, and Markdown-embed styling.
- `test-recipes/`: fixtures for manual testing in Obsidian.
- `manifest.json`, `rollup.config.js`, `vitest.config.ts`, and `tsconfig.json`: runtime and build configuration.

The two rendering flows are:

```text
`.cook` or whole-file Markdown recipe
  -> CookPlugin registration
  -> ParserService.initialize() / parse()
  -> CookView interactive state
  -> PreviewRenderer -> focused child renderers
  -> Obsidian DOM and vault APIs

fenced `cook` / `cooklang` block in Markdown reading mode
  -> registered code-block processor
  -> ParserService.initialize() / parse()
  -> MarkdownRecipeRenderer static context
  -> IngredientListRenderer + MethodStepsRenderer
  -> Obsidian DOM and vault APIs
```

## Architecture contracts

- Keep `ParserService` a singleton and keep initialization idempotent. The WASM bindings use shared global state; creating independent parser/WASM instances can corrupt it. Await `initialize()` before parsing in flows that may run before plugin startup completes.
- `CookView` owns interactive full-preview state. Pass state and callbacks through `RenderContext`; focused renderers should not become competing state owners. Interactions normally update `CookView` and trigger a full preview render.
- Only `mode`, `scale`, and `currentStep` are persisted in leaf state. Checked ingredients and active timers are in-memory session state and are cleared during view cleanup.
- Scaling intentionally parses twice: parse with the current scale for displayed quantities, and parse unscaled data to derive base servings. Never derive base servings from an already scaled recipe.
- Section step tracking uses one global zero-based index across the recipe. Step-image filenames are one-based: `Recipe.1.jpg` is the first step. Convert between them explicitly where needed.
- Recipe images are siblings of the recipe. The main image shares its basename; step images add a numeric suffix. Use `app.vault.getResourcePath()` for display URLs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cooklang/cooklang-obsidian](https://github.com/cooklang/cooklang-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
