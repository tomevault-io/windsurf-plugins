---
trigger: always_on
description: Cross-browser extension (Chrome, Firefox, Edge) that uses Google Gemini API and OpenAI-compatible APIs to summarize and translate web pages. It also supports YouTube caption summarization, image/PDF summarization, follow-up questions, custom actions, and streaming LLM output.
---

# AGENTS.md

## Project overview

Cross-browser extension (Chrome, Firefox, Edge) that uses Google Gemini API and OpenAI-compatible APIs to summarize and translate web pages. It also supports YouTube caption summarization, image/PDF summarization, follow-up questions, custom actions, and streaming LLM output.

- **Platform:** Chrome Extension Manifest V3
- **Language:** Vanilla JavaScript (ES modules) — no TypeScript, no bundler, no framework
- **API backend:** Google Gemini API + OpenAI-compatible APIs
- **Version source:** `extension/manifest.json` and `firefox/manifest.json`

## Core rules

- Conversation content is stored in a provider-agnostic format using Gemini-style `parts` arrays with `role` fields (`"system"`, `"user"`, `"model"`).
- `generateContent()` and `streamGenerateContent()` in `extension/utils.js` are the only entry points for LLM calls.
- Keep production-source changes inside `extension/` unless the task is specifically about `firefox/` manifests or the translation helper scripts in `utils/`. Update `test/`, `docs/`, root configuration files, and `AGENTS.md` when required by the task.
- Do not edit files in `extension/lib/` except when updating a vendored library according to the procedure below.
- Always use block braces `{}` for control statements such as `if`, `else`, `for`, and `while` (brace-less single-line statements like `if (cond) return;` are strictly prohibited).

## Task routing

- Popup UI, page extraction, image/PDF input: `extension/popup.html` and `extension/popup.js`
- Results tab, follow-up conversation, streaming display: `extension/results.html` and `extension/results.js`
- Options UI, provider settings, model/language settings: `extension/options.html` and `extension/options.js`
- Background logic, context menus, shortcuts, cached results: `extension/service-worker.js`
- LLM provider logic, shared utilities, error handling, theme helpers: `extension/utils.js`
- Dropdown templates: `extension/templates.html`
- Localized strings: `extension/_locales/*/messages.json`
- Firefox-specific changes: `firefox/manifest.json`

## Source file organization

Each JavaScript source file under `extension/` is divided into named sections with `// ── Section name ──...──` separator comments. Keep the section vocabulary and ordering consistent across files so that dependencies flow from low-level helpers toward entry points.

### Section vocabulary

Reuse the existing section names rather than inventing new ones. The canonical set, in dependency order:

1. `Pure utilities (no DOM access, no side effects)` — functions that depend only on their arguments and return values. No `document.*`, `Image`, `FileReader`, `canvas`, listener registration, or module-state mutation. Plain data constants also belong here.
2. Specialized helpers (e.g. `Content script injection utilities`, `Image processing`, `Tab state & notification`) — grouped by domain when a file has enough related helpers to justify a dedicated section.
3. `UI helpers` — DOM reads/writes, form population, preview rendering, status text. DOM element references (e.g. `const x = document.getElementById(...)`) belong here, not in `Pure utilities`.
4. `Button action handlers` — handlers wired to specific buttons.
5. `Core async logic` — orchestration functions (`main`, `askQuestion`, `waitForResult`, `saveOptions`, etc.). `initialize` is the last function in this section.
6. `Event listeners` — always the last section in the file; contains only listener registration and the initial call to `initialize()`.

`extension/utils.js` uses a library-oriented vocabulary instead: `UI helpers`, `Extension helpers`, `LLM APIs`. Within each section, place internal helpers before the exported entry point they support (bottom-up ordering, plan A).

### Ordering rules

- Place functions so that a function is defined before it is used within the same file, and so that lower-level helpers come before higher-level orchestration.
- Within a section, prefer `internal helper → exported API` ordering. If an exported function is self-contained, it may sit at the top of its section.
- Keep `initialize` as the last function in `Core async logic`, and keep `Event listeners` as the last section.
- Do not place DOM-touching or side-effectful functions in `Pure utilities`. Move them to `UI helpers` or a specialized helper section.
- When adding a function, choose the section by what the function does, not by where it happens to be called from.

## Validation

- After code changes, run `npm run lint` and `npm test`, and fix relevant errors or test failures before finishing.
- When modifying provider logic, verify both `apiProvider: "gemini"` and `apiProvider: "openai"` paths still work.
- When updating the extension version, update both `extension/manifest.json` and `firefox/manifest.json`.
- `npm run test:e2e` runs the minimal Chromium E2E under `e2e/` (Playwright, local mock API). It is not part of `npm test` and is not a PR-required check; run it on `main` and before releases. See [`docs/TESTING_PHASE_5.md`](docs/TESTING_PHASE_5.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sh2/extension-summarize-translate-gemini](https://github.com/sh2/extension-summarize-translate-gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
