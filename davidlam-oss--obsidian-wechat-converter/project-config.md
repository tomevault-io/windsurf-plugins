---
trigger: always_on
description: Handles markdown-it based conversion, HTML sanitization, callouts, image rendering, code block rendering, and WeChat-friendly output shaping.
---

# AGENTS.md

This file provides guidance to Codex when working with code in this repository.

## Language Preferences
- Detect the language of the user's prompt (English or Chinese). Always reply in the same language unless explicitly asked otherwise.
- When drafting replies or documentation intended for Chinese audiences, ensure natural, native-level phrasing.

## Project Summary
- This is an Obsidian plugin that converts Markdown notes into WeChat Official Account article HTML.
- The core user workflow is: open the converter panel, preview the rendered article live, copy rich HTML into the WeChat editor, and optionally sync directly to the WeChat draft box.
- The plugin is aimed at creators who publish from Obsidian to WeChat and care about fidelity for headings, code blocks, math, callouts, tables, local images, and article metadata.

## Commands
- **Install dependencies**: `npm install`
- **Generate embedded runtime bundles**: `npm run generate:embedded`
- **Check embedded runtime bundles are up to date**: `npm run check:embedded`
- **Build for production**: `npm run build`
- **Start development watcher**: `npm run dev`
- **Run unit tests**: `npm test`
- **Run coverage**: `npm run test:coverage`
- **Release dry run**: `npm run release:dryrun`
- **Release validation**: `npm run release:validate`

## Testing Expectations
- This project still relies on manual visual testing for end-to-end rendering quality and WeChat paste/sync behavior.
- Use [TEST.md](/Users/davidlin/Documents/Obsidian/MyVault/.obsidian/plugins/obsidian-wechat-converter/TEST.md) for general rendering checks.
- Use [TEST_MATH.md](/Users/davidlin/Documents/Obsidian/MyVault/.obsidian/plugins/obsidian-wechat-converter/TEST_MATH.md) when touching formula rendering or math upload behavior.
- The repository also has a substantial Vitest suite under [tests](/Users/davidlin/Documents/Obsidian/MyVault/.obsidian/plugins/obsidian-wechat-converter/tests). For logic changes, especially around rendering, sanitization, path resolution, sync flow, or error handling, add or update unit tests unless the change is purely cosmetic.

## Architecture & Structure
- **Entry source**: `input.js`
  It contains plugin lifecycle, view wiring, settings UI, preview panel logic, and top-level sync actions.
- **Generated runtime artifact**: `main.js`
  Do not hand-edit `main.js`; rebuild from source instead.
- **Markdown conversion core**: `converter.js`
  Handles markdown-it based conversion, HTML sanitization, callouts, image rendering, code block rendering, and WeChat-friendly output shaping.
- **Theme system**: `themes/apple-theme.js`
  There is one theme module with three built-in presets exposed in the UI:
  - `github` -> `简约`
  - `wechat` -> `经典`
  - `serif` -> `优雅`
- **Service layer**: `services/`
  The codebase has evolved beyond a single converter file. Important modules include:
  - `render-pipeline.js`: creates the render pipeline used by the view.
  - `dependency-loader.js`: loads embedded runtime dependencies.
  - `native-renderer.js`: native rendering path.
  - `obsidian-triplet-renderer.js` and `obsidian-triplet-serializer.js`: renderer/serializer helpers used by the newer pipeline.
  - `markdown-source.js`: resolves the active markdown source.
  - `path-utils.js`: vault/path helpers.
  - `wechat-media.js`: image and math asset processing for WeChat.
  - `wechat-html-cleaner.js`: HTML cleanup for draft sync.
  - `wechat-sync.js` and `sync-context.js`: draft sync orchestration and user-facing sync messaging.
- **Generated embedded dependency snapshot**: `services/generated-embedded-deps.js`
  This is generated. If you change embedded runtime sources such as `converter.js` or `themes/apple-theme.js`, regenerate it before building or testing.
- **Math bundle**: `lib/mathjax-plugin.js`
  Built separately via `esbuild.math.mjs` and loaded dynamically.

## Current Product Behavior To Keep In Mind
- Live preview is a first-class feature; the plugin is expected to feel close to WYSIWYG for WeChat publishing.
- The plugin supports copy-to-editor and direct sync to WeChat drafts.
- WeChat draft sync supports multiple accounts, frontmatter-aware cover/excerpt defaults, optional cleanup of a configured output directory after successful sync, and optional proxying through a Cloudflare Worker.
- Local image handling is important: wiki links, relative paths, absolute paths, GIF handling, avatar uploads, and cover selection all matter.
- Chinese punctuation normalization only affects rendered output, copy output, and sync output. It must not mutate the source Markdown note.
- The codebase has moved to a `native-only` rendering direction; be careful around old legacy/parity switches and prefer current pipeline behavior.

## Development Notes
- **Runtime environment**: Obsidian plugin running in Electron / Node-style CommonJS.
- **Languages in repo**: mostly JavaScript plus `.mjs` build scripts and test helpers.
- **External dependencies**: `obsidian`, `electron`, and CodeMirror-related APIs are provided by the Obsidian app/runtime.
- **Security-sensitive areas**:
  - HTML sanitization
  - link validation
  - proxy URL validation
  - file/path cleanup after sync

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DavidLam-oss/obsidian-wechat-converter](https://github.com/DavidLam-oss/obsidian-wechat-converter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
