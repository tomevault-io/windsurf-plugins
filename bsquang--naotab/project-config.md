---
trigger: always_on
description: Chrome Extension (Manifest V3) that turns browser tabs into a **personal knowledge base**. No server, no backend — everything runs inside the extension, data stored in `chrome.storage.local`.
---

# naoTab — Chrome Extension

## Project overview

Chrome Extension (Manifest V3) that turns browser tabs into a **personal knowledge base**. No server, no backend — everything runs inside the extension, data stored in `chrome.storage.local`.

UI language: **English**. Codebase comments: mixed EN/VI.

---

## File structure

```
naoTab/
├── manifest.json        # Manifest V3, permissions: tabs + storage + unlimitedStorage + scripting
├── popup.html/css/js    # Popup — view tabs, save, export
├── app.html/js          # Knowledge Base full-page
├── settings.html/js     # AI provider config page
├── build.sh             # Build script → naotab-vX.Y.Z.zip
├── core/
│   ├── schema.js        # ⚠️ Single source of truth for data structure — never remove fields
│   ├── storage.js       # Bookmarks CRUD + settings (reads only, no business logic)
│   ├── ai.js            # callAI() + suggestTags()
│   └── export.js        # exportJSON / importJSON / exportObsidian / bookmarkToObsidianMd
│   └── sync/            # (future) drive.js, notion.js, etc.
├── vendor/
│   ├── d3.min.js        # D3.js v7.9.0 — bundled locally (CSP compliance)
│   └── jszip.min.js     # JSZip 3.10.1 — bundled locally (CSP compliance)
├── icons/               # icon16/48/128.png
├── README.md            # English README (links to VI version)
├── README.vi.md         # Vietnamese README (links to EN version)
└── CLAUDE.md            # This file
```

---

## Architecture & Data flow

```
popup.js  ──────────────────────┐
settings.js ────────────────────┤──► storage.js ──► chrome.storage.local
app.js  ────────────────────────┘         │
                                          └──► External AI API (optional)
```

### Data schema — bookmark object

```json
{
  "id": "1712345678901",
  "url": "https://...",
  "title": "Page title",
  "reason": "Why user saved this",
  "summary": "AI-generated or manual summary",
  "tags": ["rust", "async", "performance"],
  "favIconUrl": "https://...",
  "pageMeta": {
    "description": "og:description or meta description",
    "ogTitle": "og:title",
    "ogType": "article",
    "keywords": "meta keywords",
    "author": "meta author",
    "siteName": "og:site_name",
    "ogImage": "og:image URL",
    "lang": "html lang attribute",
    "canonical": "canonical URL"
  },
  "savedAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-01T00:00:00.000Z"
}
```

Note: `status` field removed from UI (was: unread/reading/done/revisit). `pageMeta` is `null` if meta read failed or tab was a chrome:// page.

### Data schema — settings object

```json
{
  "aiEnabled": false,
  "aiBaseUrl": "https://api.openai.com/v1",
  "aiApiKey": "sk-...",
  "aiModel": "gpt-4o-mini",
  "featTags": true,
  "featSummary": true
}
```

---

## Key files and responsibilities

### `core/schema.js`
⚠️ **Never remove or rename existing fields.** Only ADD new fields with a default value.

- `SCHEMA_VERSION` — current version integer, increment when adding fields
- `BOOKMARK_DEFAULTS` — canonical bookmark shape with all fields and defaults
- `SETTINGS_DEFAULTS` — canonical settings shape
- `migrateBookmark(raw)` — heals old bookmark objects: fills missing fields, drops removed ones
- `createBookmark(fields)` — builds a new bookmark object with all required fields

### `core/storage.js`
CRUD only. Loaded into popup, app, settings pages.

- `getBookmarks()` — read all bookmarks, runs `migrateBookmark()` on each (safe for old data)
- `saveBookmark(fields)` — save new, dedup by URL, calls `createBookmark()`
- `updateBookmark(id, changes)` — partial update
- `deleteBookmark(id)` — delete by id
- `getSettings()` / `saveSettings(settings)` — read/write settings

### `core/ai.js`
- `callAI(title, url, pageMetaText)` — call AI API, returns `{tags, summary}` or throws
- `suggestTags(title, url)` — offline keyword + domain matching, returns tag array

### `core/export.js`
- `exportJSON()` / `importJSON(jsonString)` — full backup/restore
- `bookmarkToObsidianMd(bookmark)` / `exportObsidian()` — Obsidian .md export

### `popup.js`
- `loadTabs()` — queries all tabs + bookmarks, marks saved URLs
- `getPageMeta(tabId)` — uses `chrome.scripting.executeScript` to read SEO meta tags; returns structured object with `_aiText` field for AI prompt
- `openSaveModal(tab)` — async, reads meta in background, shows/hides AI row
- Window save: calls `getPageMeta()` for each tab, shows `⏳ N/total` progress, saves `pageMeta`
- Single tab save: passes `pageMeta` to `saveBookmark()`
- **"💾 Tab this"** button — saves active tab directly from toolbar

### `app.js`
State: `allBookmarks`, `activeTag`, `excludedTags` (Set), `searchQuery`, `currentView` (default: `'graph'`), `editingId`, `panelId`

Key features:
- **Graph view** (default): D3 force simulation, single-click → node panel + highlight connected nodes, double-click → open URL, click background → reset. Node color: blue (has summary) vs grey (no summary)
- **List view**: card per bookmark, click card → node panel (links/buttons still work normally)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bsquang/naotab](https://github.com/bsquang/naotab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
