---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Manifest V3 Chrome/Edge extension ("闲鱼词典" / Xianyu Dictionary) that helps with coded slang (暗语) on the Xianyu / Goofish second-hand marketplace. It does three things on `goofish.com` / `xianyu.com` / `2.taobao.com`:
- **Search suggestions** — type a normal word in the site's search box, get slang variants to click-replace.
- **Inline decoding** — highlights slang in listing titles/descriptions with hover tooltips; a floating button summarizes all slang on the page.
- **Popup tools** — translate panel, browse-by-category, a user-editable custom dictionary, and an opt-in on-device AI fallback (Chrome Built-in AI / Gemini Nano).

Plain vanilla JS — no framework, no bundler, no `package.json`, no tests, no linter. Build tooling is two Node ESM scripts in `tools/`.

## Commands

There is **no build step for development**. Edit files, then reload the extension at `chrome://extensions/` (Developer mode → Load unpacked → repo root).

```bash
# Edit the dictionary (the only "data source" you normally touch):
$EDITOR tools/dictionary.source.json      # then reload extension — no build needed

# Regenerate the encrypted bundle only (data/dictionary.enc.json):
node tools/encrypt-dict.mjs

# Full release package → dist/xianyu-slang-helper/ (encrypt + copy + safety checks):
node tools/package.mjs
cd dist && zip -r xianyu-slang-helper-v<version>.zip xianyu-slang-helper   # upload to Web Store

# Import crowd-sourced slang from search-sharp.com (preview first, never blind-merge):
node tools/fetch-searchsharp.mjs               # fetch + quality/policy filter → tools/searchsharp.staged.json
node tools/fetch-searchsharp.mjs --apply       # also merge safe terms into dictionary.source.json
```

The import flow is driven by the **`sync-searchsharp` project skill** (`.claude/skills/sync-searchsharp/`), which wraps the script with the required human review of the staged report. See that SKILL.md before running an import. Key points: the source is noisy crowd-sourced UGC (memes upvoted alongside real terms) and contains Web Store policy-violating terms — `fetch-searchsharp.mjs` excludes the latter via a `POLICY_RULES` denylist (seeded from `dictionary.grey-area.json` themes) and only *reports* them; `--apply` never writes them. New concepts arrive uncategorized (SearchSharp has no taxonomy) and land in a `社区补充` category for manual filing.

## Architecture

### Three runtime contexts
- **Content script** (`content.js`) — search-suggestion panel + slang highlighting via `MutationObserver` + `TreeWalker`. Loaded after `data/crypto.js` and `data/dict-loader.js` (order matters; see `manifest.json` `content_scripts.js`).
- **Popup** (`popup.html` / `popup.js` / `popup.css`) — 4 tabs (翻译 / 词典 / 我的 / 关于). The **only** place the AI fallback runs.
- **Background** (`background.js`) — service worker; does nothing but set `decodeEnabled: true` on install.

### The dictionary loading pipeline (the core design)
`data/dict-loader.js` builds an immutable, frozen `self.XYDict` query API and is the single gateway to dictionary data. It auto-selects a source at load time:
- **Dev path** — if `tools/dictionary.source.json` exists (plaintext), read it directly. No decryption.
- **Release path** — `package.mjs` strips `tools/`, so the loader falls back to `data/dictionary.enc.json` and decrypts via `self.XYCrypto.loadEncrypted()` (`data/crypto.js`).

Decrypted plaintext lives **only in closure variables** — never attached to `window`/`self`. Both `XYDict` and `XYCrypto` are `Object.freeze`'d and defined non-enumerable. Callers get per-query answers (`suggestSlangsFor`, `decodeText`, `getCategoryEntries`, …), not the whole dictionary at once. **Preserve this** when editing — it is a deliberate anti-scraping boundary, not incidental.

### Data model & indexes
Source shape: `categories: [{ id, name, icon, entries: { "<normal word>": ["<slang>", ...] } }]`. On load, `buildIndexes()` derives a forward map (`normal → slangs`), a reverse map (`slang → normals`), and one combined `RegExp` (slang alternation, longest-first) used to scan page text. The user's custom dictionary (`chrome.storage.local.customDict`) is merged in as a synthetic `custom` category; a `storage.onChanged` listener invalidates and rebuilds the indexes.

### Encryption (anti-scraper, NOT anti-reverse-engineering)
AES-256-GCM, key derived by PBKDF2-SHA256 (100k rounds, fixed salt). **The passphrase is intentionally plaintext** in `data/crypto.js` — the goal is only to stop `curl | grep` scrapers from lifting the word list, not to hide anything from a reviewer or a determined reverse engineer. Do **not** "harden" this by obfuscating the key; that contradicts the documented design and the Web Store reviewer notes. Rationale is spelled out in the header comment of `data/crypto.js` and in `store-assets/reviewer-notes.md`.

## Invariants — easy to break, will bite you

- **Version lives in two files and must match**: `manifest.json` `version` and `tools/dictionary.source.json` `version`. `package.mjs` hard-fails if they differ. Bump both together.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sudoo-dev/xianyu-slang-helper](https://github.com/sudoo-dev/xianyu-slang-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
