---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Cursor, Codex, Aider,
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, Cursor, Codex, Aider,
etc.) when working with code in this repository.

## Branching workflow

This repo follows GitHub flow. For any new task, create a feature branch off
`main`, push it, and open a PR — do not commit directly to `main`.

## Repository Purpose

Prototyping browser extension capabilities for improving browser use agent
performance:

- Token efficiency
- Security
- Compliance: e.g., exposure to PII
- Accuracy

Ideas explored in this repository:

- Masking/Redacting sensitive information on the webpage
- Blocking/Modifying dark patterns on the webpage
- Preprocessing webpage content to be more agent-friendly, e.g., hiding
  irrelevant content, hiding user-generated comments which could contain
  prompt-injection attacks, etc.

## Reference

- Upload extension for use with browserbase:
  [browser-extensions](https://docs.browserbase.com/platform/browser/core-features/browser-extensions#browser-extensions)

## Technology

- Chromium Extension: Manifest V3, TypeScript, bun (bundling) assume Chrome 148+
  for modern JS features
- Python: use `uv` for all package management

## Linting

Two linters run on `extension/`: Biome owns formatting plus its recommended rule
set; ESLint runs only rules Biome doesn't have. The split is mechanical — do not
duplicate a rule between them.

- Biome config: `extension/biome.json`.
- ESLint config: `extension/eslint.config.js` (flat config) — pulls in
  `@eslint/js`, `typescript-eslint`, and `eslint-plugin-unicorn`, then disables
  unicorn rules that overlap with Biome or are too opinionated for this repo.
- Custom rules: `extension/eslint-rules/*.js`. Each rule is one file, exported
  via `extension/eslint-rules/index.js` under the
  `agent-browser-shield/<rule-name>` namespace. Add a rule by dropping a file in
  `eslint-rules/`, exporting it from `index.js`, and enabling it in
  `eslint.config.js`.
- `bun run check` runs Biome then ESLint; `bun run check:fix` runs both with
  `--fix`/`--write`.

`demo-site/` has a smaller ESLint config (`demo-site/eslint.config.js`) that
just enforces `unicorn/prevent-abbreviations` so the React surface stays
grep-friendly with the same naming conventions as the extension. The replacement
allowlist (`props`, `ref`, `args`, loop counters, etc.) mirrors the extension's.
`bun run check` in `demo-site/` runs Biome + ESLint.

## Rule ID naming

Rule IDs follow `<target>-<verb>` and the verb names what the rule does to the
DOM. Five canonical verbs:

- `annotate` — adds agent-readable info; page content unchanged
  (`cart-addon-annotate`, `roach-motel-annotate`)
- `hide` — visually conceals via `display: none`; element stays in DOM
  (`ads-hide`, `cookie-banner-hide`, `chat-widget-hide`,
  `newsletter-modal-hide`)
- `redact` — replaces content with a click-to-reveal placeholder (`pii-redact`,
  `secrets-redact`, `comments-redact`, `prompt-injection-redact`)
- `sanitize` — keeps the element and cleans attributes/text/state
  (`json-ld-sanitize`, `attribute-injection-sanitize`, `confirmshame-sanitize`,
  `checkout-checkbox-sanitize`)
- `strip` — removes the element/node from the DOM entirely (`noscript-strip`,
  `html-comment-strip`, `hidden-text-strip`, `svg-sprite-strip`,
  `svg-text-strip`, `meta-injection-strip`)

`-helper` is reserved for non-defensive agent affordances (`search-url-helper`).
See CONTRIBUTING.md → *Adding a new rule* → *Rule ID naming* for the longer
write-up and the hide-vs-redact decision rule.

## Rule authoring: re-scan SPA mutations

Rule `apply` runs once at `document_idle`. Client-side route changes in SPAs
(React Router, Vue Router, etc.) swap subtrees in and out without a new page
load, so anything that only ran in `apply` will never see post-navigation
content — and most of our targets (PII, secrets, scarcity badges, hidden text)
are exactly the kind of late-mounted content SPAs are built on.

Default to wiring a `createSubtreeWatcher`
(`extension/src/lib/subtree-watcher.ts`) into any rule that mutates the DOM,
with `skipPlaceholderSubtrees: true` when the rule inserts placeholders. Mirror
the pattern used in `pii-redact`, `secrets-redact`, `scarcity-redact`,
`hidden-text-strip`, etc.: a shared `scanAndX(root)` function called by both
`apply` and the watcher's `onSubtrees`, plus a `teardown` that calls
`watcher.stop()`. Skip the watcher only when there is nothing to re-scan after
initial load — e.g., a one-shot landmark injection — and call that out in a
comment.

## DOM marker attributes

Every `data-abs-*` attribute the engine or any rule writes onto the page is
declared and exported from `extension/src/lib/dom-markers.ts` — the single
canonical registry. Engine-level markers are named `<PURPOSE>_ATTR`; per-rule
markers are `<RULE>_<PURPOSE>_ATTR`. Import the constant; do not inline the
literal. An ESLint `no-restricted-syntax` rule (`eslint.config.js`) blocks raw
`"data-abs-…"` string and template literals everywhere except the registry
itself, so name collisions and convention drift surface at lint time.

## Rule defaults

The initial enabled/disabled state for each rule lives in
`extension/data/rule-defaults.json`, not on the rule modules themselves. The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pixiebrix/agent-browser-shield](https://github.com/pixiebrix/agent-browser-shield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
