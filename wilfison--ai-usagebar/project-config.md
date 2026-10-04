---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **GNOME Shell extension** (GJS / ES modules, `shell-version` 47–51, developed and tested on 50) that
shows AI plan usage in the top panel for seven vendors (**Anthropic, OpenAI,
Z.AI/GLM, OpenRouter, DeepSeek, Kimi, Ollama Cloud**) plus a user-mapped
**custom provider**. It is a port of the Rust Waybar widget
[ai-usagebar](https://github.com/akitaonrails/ai-usagebar) by akitaonrails.

The extension is fully built: panel indicator, per-vendor fetch + OAuth refresh,
scroll-to-cycle, a collapsible multi-vendor popup, and a prefs window. This repo
IS the installed extension (it lives under
`~/.local/share/gnome-shell/extensions/ai-usagebar@wilfison`); gnome-shell loads
the JS directly; there is no build step.

## Develop / test

The `Makefile` is the canonical dev loop: prefer these targets over raw
`gnome-extensions` / `journalctl` / `gjs`. Run `make` to list them.

```bash
make test      # gjs -m tests/run.js: pure-JS unit suite (compiles schema first)
make lint      # tools/lint.sh: trailing-newline + no imports.* + no SPDX header
make eslint    # npx eslint .: GNOME Shell flat config (needs `npm ci` first)
make validate  # tools/validate.sh: metadata.json + schema --strict
make watch     # re-run `make test` on changes under lib/ ui/ tests/ (needs inotify-tools)
make reload    # disable + enable (Wayland still needs a full relog to pick up changes)
make run       # launch a throwaway nested gnome-shell (Wayland) to test live
make logs      # journalctl -f -o cat /usr/bin/gnome-shell
make pot       # tools/i18n.sh pot: re-extract strings into po/<uuid>.pot
make update-po # msgmerge each po/*.po against the refreshed template
make compile-locale  # msgfmt po/*.po → locale/<lang>/LC_MESSAGES/*.mo (dev)
make i18n-check      # fail on a stale .pot or a malformed po/*.po (CI gate)
make pack      # gnome-extensions pack . --podir=po --force → upload zip (with .mo)
make info      # gnome-extensions info ai-usagebar@wilfison
```

Run a **single test file** directly: `gjs -m tests/lib/vendors/anthropic/parser.test.js`.
Each `*.test.js` is self-contained (calls `system.exit(summary())`), and
`tests/run.js` discovers them **recursively** (so tests may nest in
subdirectories that mirror the source path) and runs each as an isolated
`gjs -m` subprocess.

The runner colorizes output with ANSI codes; `tests/run.js` honors the
[`NO_COLOR`](https://no-color.org) env var, so run `NO_COLOR=1 make test` to get
plain (uncolored) output; use this when capturing logs or running non-interactively.

CI (`.github/workflows/ci.yml`) runs `make schemas`, `make test`, `tools/lint.sh`,
`npx eslint .`, `make validate`, and `make i18n-check` on every PR (the runner
installs `gettext` for the i18n tools).

## Architecture

`extension.js` is thin: `enable()` constructs the `Indicator`
(`ui/indicator.js`) with `getSettings()` + an `openPreferences` callback and adds
it to the panel; `disable()` calls `this._indicator.destroy()`. **Everything
created in `enable()` must be released in `disable()`** (widgets, GLib timeout
sources via `GLib.Source.remove`, signal handlers, the Soup session, the
Cancellable); reviewers reject extensions that leak on disable. `Indicator`
owns this discipline in its `destroy()`.

### Vendor-adapter pattern (the core abstraction)

The indicator never branches per vendor. `lib/vendors/registry.js` maps a vendor
id → a uniform `Adapter` (`{ id, cacheId, icon, vendorShort, fetchSnapshot,
severity, peakUsage, placeholders, notifyRows, resetCredits, buildSection }`,
plus optional `fakeSnapshot` and `shortCode(config)`); the indicator calls
`getAdapter(id)` and drives it generically. `notifyRows(snapshot, _)` lists one
`{key, label, percent, resetsAt}` per usage window and `resetCredits(snapshot)`
the banked resets `{title, expiresAt}`; both feed `lib/notify.js`. Each vendor
lives in its own directory
`lib/vendors/<vendor>/` as a **module quad**:

- `lib/vendors/<vendor>/adapter.js`: wires the triple into the uniform `Adapter`
  object and exports it (e.g. `anthropicAdapter`). Derives the creds path /
  resolved API key from config, catching resolution errors into the standard
  error result, and exposes the pure `buildSection` as-is (the indicator injects
  the real `gettext` at the call site). Imports `main.js` (transitively `Gio`)
  but **no** `resource://`, so the whole `ADAPTERS` graph loads under `gjs -m`;
  its shape is checked in `tests/lib/vendors/registry.test.js`.
- `lib/vendors/<vendor>/main.js`: fetch state machine (`fetchSnapshot`). Reads
  creds/key, maybe-refreshes the OAuth token, GETs the usage endpoint, caches,
  and falls back to stale cache on failure. Transitively imports `Gio` (via
  cache/http), so it is **not** unit-tested directly. Never throws: always
  resolves to a `FetchResult`.
- `lib/vendors/<vendor>/parser.js` (**pure**): `parseUsage(jsonBytes) → snapshot`,
  plus `severity`, `peakUsage`, `placeholders` (Map for `bar-format`
  substitution; takes an injected `ngettext`), `notifyRows`, `resetCredits`,
  `snapshotToCacheJson`/`parseCacheJson`, `ICON`, `VENDOR_SHORT`. No `gi://`;
  fully unit-tested.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilfison/ai-usagebar](https://github.com/wilfison/ai-usagebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
