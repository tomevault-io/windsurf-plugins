---
trigger: always_on
description: This file provides guidance to AI coding agents (Codex, Claude Code, Kimi Code, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Codex, Claude Code, Kimi Code, etc.) when working with code in this repository.

## Overview

Lecture Presenter（演讲宝）is a Tauri 2 desktop app for presenting course materials: PDF, video, Markdown, source code, and **PPTE (PPT-EXTRA)** — HTML files that simulate PowerPoint slides. All app code lives in `lecture-app/`.

The desktop app is the open-source half of a two-repo product split:

- **This repo (desktop, open source)** = editor + presenter shell. It is the landing container for content generated elsewhere.
- **Lecture Web / LectureAI (separate closed-source repo)** = generation backend (FastAPI). Only operational notes about it live here, in `memory/2026-07-24-lecture-web-backend.md`; its code is NOT in this repo. The empty `backend/app/{api,models,schemas,services}/` directories at this repo's root are unused placeholders — do not put code there.

The desktop app talks to the cloud server (`https://design.hz-study-system.com`, configurable via `app-config.json`) for auth/membership, update checks, notifications, and login-only analytics.

## Commands

All commands run from `lecture-app/`:

```bash
npm install            # install deps (also requires Rust stable + Tauri 2 system deps)
npm run dev            # tauri dev — launches the desktop app with live frontend
npm run build          # tauri build — bundles to src-tauri/target/release/bundle/
npm run build:release  # obfuscated production build (scripts/build-obfuscated.js)

# Frontend logic tests (plain node, no framework — each script loads a src/js file in a vm sandbox):
npm run test:ppte            # scripts/test-ppt-extra-viewer.js (slide URL/platform logic)
npm run test:annotator       # scripts/test-ppte-annotator.js (annotation overlay)
npm run test:ppte-shared     # scripts/test-ppte-shared-groups.js (shared page groups)
npm run test:resource-center # scripts/test-resource-center.js (resource center logic)
npm run test:captions        # scripts/test-live-caption.js (caption transcript logic)
npm run test:course-manager  # scripts/test-course-manager.js (course grouping/switcher logic)
node scripts/test-auth.js    # auth UI logic (no npm script alias)

cargo check            # run inside src-tauri/ to type-check Rust
cargo test             # run inside src-tauri/ — unit tests in lib.rs plus tests/info_plist.rs
```

There is no bundler, framework, or lint step: the frontend is plain HTML/CSS/JS loaded directly from `lecture-app/src/` (`frontendDist: ../src`). A quick sanity check for frontend edits is `node --check src/js/<file>.js`. `npm run build:release` copies `src/` to `dist/`, obfuscates non-vendor JS, temporarily patches `tauri.conf.json`, then restores it.

**Packaging gotcha (macOS Apple Silicon):** the default Rust toolchain may be `stable-x86_64-apple-darwin` (installed under Rosetta), so a plain `npm run build` produces an x86_64 unsigned bundle. Build with `npx tauri build --target aarch64-apple-darwin` (do NOT use `npm run build -- -- --target`, the double `--` is unreliable); output lands in `target/aarch64-apple-darwin/release/bundle/`. The result may carry a broken linker signature — re-sign with `codesign --force --deep --sign -` and verify with `codesign --verify --deep --strict` before replacing `/Applications`.

## Architecture

### Frontend (lecture-app/src/)

- `index.html` — single page containing ALL UI: sidebar, content area, and every modal (PDF/video/code/HTML/PPTE viewers, course creator, settings, auth, resource center). Modals are `.hidden`-toggled divs.
- `audience.html` — separate Tauri window shown to the audience in speaker mode; receives `slide-change` events and emits `audience-navigate` events back.
- `js/*.js` — plain scripts, each defining one global object (e.g. `PptExtraViewer`, `CourseLoader`, `Content`). No modules/imports; load order is set by `<script>` tags in `index.html`. `app.js` installs global error logging (`window.errorLogs`) and calls each component's `init()` on DOMContentLoaded.
- Frontend ↔ backend via `window.__TAURI__.core.invoke(...)` and `window.__TAURI__.event` — guarded by `if (window.__TAURI__)` so browser dev mode degrades to `fetch`.
- `vendor/` holds third-party browser libs (pdf.js, marked, highlight.js, pptxgenjs bundle) loaded via `<script>`/`<link>`; these are not obfuscated.

Key frontend modules:

- `settings.js` — theme, font size, view switching (course view / PPTE management / PPTE editor). It was split up; satellite modules are `ai-settings.js` (AI provider config), `gitee-settings.js` (Gitee backup UI), `dev-settings.js` (developer settings, template center, updater config), `caption-settings.js` (live-caption credentials), `ppte-recent.js`, `ppte-create.js`, and `ppte-shared-groups.js` (`Object.assign`ed into `Settings`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangjingyi2012/lecture-presenter](https://github.com/wangjingyi2012/lecture-presenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
