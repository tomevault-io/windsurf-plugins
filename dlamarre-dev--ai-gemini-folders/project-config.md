---
trigger: always_on
description: This file is the onboarding brief for an agent (or contributor) picking up a task
---

# CLAUDE.md — Working guide for this repository

This file is the onboarding brief for an agent (or contributor) picking up a task
here. It captures the project structure, the build/test/release procedures, the
tools, and the decided constraints — so you can start work without re-discovering
the codebase. Keep it accurate: update it when procedures or constraints change.

---

## 1. What this repository is

Two Manifest V3 browser extensions (Chrome **and** Firefox) that organize AI
conversations into folders and provide a reusable prompt library:

- **Gemini Folders (GF)** — Google Gemini only. Current version **4.6.0**.
- **AI Folders (AF)** — 18 web platforms (Gemini, Claude, ChatGPT, Copilot,
  DeepSeek, Grok, Perplexity, Baidu, Z.ai, Kimi, Qwen, Meta AI, Mistral, Poe,
  Duck.ai, You.com, Pi, Character.AI) **+ a user-configured local LLM**.
  Current version **1.7.0**. The popup's per-site "new conversation" buttons
  are generated from the `SITES` registry (site-config.js) into wrapping
  grid rows — adding a site does not touch popup.html.
  **Site logos**: the extension ships pre-rasterized PNGs
  (`extensions/ai-folders/icons/`, some with a `-light` theme variant) —
  inline SVG `url(#gradient)` fills do NOT render in the popup, don't go back
  to them. The vector sources live in `assets/site-logos/` (reference for the
  website/screenshots/videos); regenerate the PNGs with
  `node tools/generate-site-icons.js` (needs Chrome) after changing one.
  Gemini Folders has an `icons/` directory too, holding only `gemini.png` — the
  welcome page's site row needs the real Gemini mark, which is not the same image
  as GF's own `icon.svg` (§10). Keep the two `gemini.png` copies identical.

Both are built from one shared codebase in `src/`, with a thin per-extension
overlay in `extensions/<name>/`. The build merges the two.

Public site / store-referenced privacy policy: **https://aifolders.xyz**
(served from `docs/`, GitHub Pages).

---

## 2. Repository structure

```
src/                         Shared code (copied into every build)
  utils.js                   Storage (loadData/saveData), LZString compression +
                             chunking (makeChunks/assembleChunks), bookmark mobile
                             sync (syncToBookmarksTree), prompt injection
                             (injectPromptIntoEditor / insertSuggestionsInEditor),
                             title extraction, sort helpers, isSafeUrl/normalizeUrl,
                             import merge (mergeImportData/normalizePromptData)
  folders.js                 Folder/conversation rendering + actions (rename, move,
                             delete, pin, tab groups)
  prompts.js                 Prompt library UI (list, inline edit/auto-save, per-row
                             actions, search/sort)
  popup-core.js              Shared popup wiring: i18n pass (applyCommonI18n),
                             clearable search, save-conversation flow, mode toggle,
                             sort menu, mobile-sync toggle, import/export
  ui.js                      showCustomModal (Enter/Escape/backdrop), storage bar,
                             review banner
  bulk-actions.js            Multi-select bar (move/delete)
  prompt-trigger.js          Content script: `#name` + Space trigger (isolated world)
  import.js / import.html    Standalone import page (Firefox can't open a file
                             picker from a popup)
  welcome.html / .js / .css  First-run page, opened once on fresh install (see §10).
                             Shared; text from each extension's _locales, site logos
                             from its site-config.js. Styled after aifolders.xyz
  popup.css                  Shared styles
  lz-string.min.js           Vendored LZString (excluded from coverage)

extensions/ai-folders/       AF overlay (overrides/adds files on top of src/)
  manifest.json  popup.html  popup.js  background.js  site-config.js
  popup-extra.css            AF-only CSS (inherits src/popup.css, adds tweaks)
  _locales/                  43 locales (messages.json)
  icon*.png / *.svg
extensions/gemini-folders/   GF overlay (same set, no popup-extra.css)

tests/                       Jest suites (jsdom). setup.js mocks chrome.* + LZString.
                             ~270 tests, ~65% coverage. Pure-logic + DOM behaviour.
                             Subdirs: stats-collector/, store-publisher/.

Marketing/
  ai-folders/  gemini-folders/   Promo<LANG>.txt (43 each) = store listing text,
                                  screenshots/, DEVELOPMENT_STORY.md
  (Generators were removed — edit Promo*.txt and _locales by hand.)

docs/                        Static GitHub Pages site (aifolders.xyz)
  privacy.html               Renders from site/privacy-i18n.js via site/app.js
  site/privacy-i18n.js       Privacy policy text, 43 languages (window.AF_PRIVACY)
  site/app.js  styles.css    Page renderer + styles
  site/i18n-data.js  i18n-manual.js  logos.js
  uninstall-ai-folders.html  Uninstall feedback survey, one page per extension
  uninstall-gemini-folders.html   (noindex; see §9)
  site/uninstall.js  uninstall-i18n.js  uninstall-forms.js


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dlamarre-dev/AI-Gemini-Folders](https://github.com/dlamarre-dev/AI-Gemini-Folders) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
