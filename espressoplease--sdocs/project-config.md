---
trigger: always_on
description: Lightweight stateless markdown editor with live styling. Single Node.js file serves a single HTML file. No build step, no framework, one dependency (`marked` for MD parsing).
---

# SDocs

Lightweight stateless markdown editor with live styling. Single Node.js file serves a single HTML file. No build step, no framework, one dependency (`marked` for MD parsing).

## Stack

- **Server**: `server.js` — pure Node `http` module, small
- **Frontend**: split across `public/`:
  - `index.html` — markup only
  - `css/tokens.css` — CSS custom properties, dark theme, theme transitions
  - `css/layout.css` — reset, body, topbar, main layout, left panel, divider
  - `css/rendered.css` — `#rendered` markdown styles, collapsible sections, copy buttons
  - `css/panel.css` — right panel, controls, statusbar
  - `css/mobile.css` — mobile `@media` breakpoint
  - `sdocs-yaml.js` — YAML front matter parse/serialize, UMD shared with Node
  - `sdocs-slugify.js` — slugify heading text to URL-safe IDs, UMD shared with Node
  - `sdocs-styles.js` — pure style data tables + logic, UMD shared with tests
  - `sdocs-state.js` — shared `window.SDocs` mutable state namespace
  - `sdocs-theme.js` — Google Fonts, font loading, dark mode, theme toggle
  - `sdocs-controls.js` — CSS variable management, color cascade, control wiring
  - `sdocs-export.js` — PDF/Word/MD export, save-default styles
  - `sdocs-app.js` — render orchestration, hash encode/decode, Brotli compression, syncAll, mode switching, drag/drop, file info card, scroll hints, init
- **Tests**: `node test/run.js` — red/green, no test framework, uses Node `assert` + `http`
  - `test/runner.js` — shared harness: `test()`, `testAsync()`, `get()`, `report()`
  - `test/test-yaml.js` — YAML front matter parse/serialize tests
  - `test/test-styles.js` — SDocStyles pure module tests
  - `test/test-cli.js` — CLI parseArgs/buildUrl + style merging tests
  - `test/test-slugify.js` — slugify + heading dedup tests
  - `test/test-base64.js` — browser base64 UTF-8 roundtrip tests
  - `test/test-files.js` — file existence + content assertions
  - `test/test-http.js` — HTTP server tests (async)
- **Playwright tests**: `npx playwright test test/write-mode.spec.js` — write mode editor tests
  - `test/write-mode.spec.js` — 42 tests for toolbar actions, toggles, shortcuts, block exits
  - `playwright.config.js` — Chromium only, auto-starts server on :3000

## Writing style (docs, copy, UI strings, commit messages)

Calm, explicit, honest. Not salesy, not defensive, not cute.

- **State what something does, not how great it is.** "The server stores ciphertext," not "Our server never sees your data, your privacy is protected!"
- **Name trade-offs out loud.** If something costs you privacy, latency, or uptime compared to the alternative, say so plainly. Don't front-load reassurance to bury a caveat.
- **Skip rhetorical questions and self-defense.** "Doesn't this break privacy? It doesn't, and here's why..." is PR framing. Just describe what happens step by step; the reader forms their own view.
- **No hype words.** Avoid "simply", "just", "blazing", "seamless", "best-in-class", "trust us", "rest assured", "don't worry". Remove exclamation points from technical copy.
- **Imperative over aspirational.** "To verify, open devtools and watch the Network tab" beats "You don't have to take our word for it, feel free to verify it yourself!" Same information, half the words, no persuasion.
- **Show the mechanism, let the reader judge.** Diagrams, code, and HTTP traces are more convincing than adjectives. The Privacy section's MDN quote does this; follow that shape.
- **Boring is fine.** If a paragraph reads like documentation instead of marketing, that's the goal.

When you catch yourself writing a sentence that tries to *make the reader feel good about a choice*, delete it and write the one that explains what actually happens.

## Dashes

Never use em dashes (`—`) or en dashes (`–`) anywhere: source files, comments, commit messages, docs. Use a plain hyphen (`-`) instead. This also means no `\u2014` / `\u2013` Unicode escapes.

## Agent integration block

The `sdoc setup` command appends a SDocs explainer to coding-agent config files (`~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`, etc.). The block lives as `AGENT_BLOCK` in `bin/sdocs-dev.js` and is duplicated as per-agent snippets in `public/sdoc.md` (the "Set up your agent" section). **If you reword one, reword the other.** The marker comment `<!-- sdocs-agent-block -->` on the first line is used for idempotent re-runs (skip files that already contain it).

## CLI state

All CLI-side state lives under `~/.sdocs/`:
- `styles.yaml` — user-editable default styles
- `update-check.json` — daily npm version cache
- `setup.json` — agent setup tracking (so `sdoc setup` only auto-prompts once)

## Architecture

The entire app is stateless. The server just serves static files. All state (current markdown content, parsed front matter, style values) lives in the `window.SDocs` namespace in the browser, primarily `SDocs.currentBody` and `SDocs.currentMeta`.

Styles are driven entirely by CSS custom properties on `#rendered`. Every control in the right panel maps to a `--md-*` variable. No style objects are stored separately — `collectStyles()` reads the DOM when exporting.

### JS module communication


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [espressoplease/SDocs](https://github.com/espressoplease/SDocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
