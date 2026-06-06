---
trigger: always_on
description: Tiny Chrome MV3 extension. No build step, no dependencies, no network calls.
---

# CLAUDE.md — claude_extention

Tiny Chrome MV3 extension. No build step, no dependencies, no network calls.
The font is bundled. After any change, the user reloads via
`chrome://extensions` and refreshes the host tab.

## Scope

Runs on three hosts (declared in `manifest.json`):

- `https://claude.ai/*`
- `https://chatgpt.com/*`
- `https://chat.openai.com/*` (legacy domain — redirects to chatgpt.com, kept for safety)

The extension is still named **"Claude.ai Farsi RTL"** even though it covers
ChatGPT too. Do not rename the extension or files unless the user asks —
they explicitly deferred rebranding.

## Features (all in one content script)

1. **Per-block Farsi RTL** — `MutationObserver` over the page; for each prose
   block (`<p>`, `<li>`, headings, etc., *not* `<div>`), sample ~50 chars and
   flip to `dir="rtl" lang="fa"` if Persian letters exceed `THRESHOLD` (0.4).
   Nested English blocks inside a Farsi container get flipped back to LTR.
   `<pre>` / `<code>` are skipped.

2. **Font picker** — popup writes `farsiFont` / `englishFont` to
   `chrome.storage.local`; content script mirrors them onto
   `documentElement` as `--farsi-font` / `--english-font` CSS variables that
   `styles.css` reads.
   - Font lists live in `FARSI_FONTS` / `ENGLISH_FONTS` arrays at the top of
     `popup.js` — keep the defaults aligned with `DEFAULT_FARSI_FONT` /
     `DEFAULT_ENGLISH_FONT` in `content.js`.
   - Custom values from older installs are preserved by `fill()` — it
     appends a "Custom (…)" option when the stored value isn't in the list.
   - The English font is only visible when an English block sits inside a
     Farsi-marked container (`[data-farsi-rtl="0"]`). In English-only
     conversations nothing gets tagged and the dropdown appears inert —
     this is by design. Don't "fix" it by applying the font globally; that
     would override the host site's typography.
   - `applyPreview()` updates the in-popup preview elements; live propagation
     to the chat tab is via the content script's `chrome.storage.onChanged`
     listener — no tab message needed.

3. **Synced prompt library** — popup stores prompts in
   `chrome.storage.sync` (Chrome syncs them across browsers / laptops
   signed into the same Google account — no network code on our side,
   no new permissions) and sends `{type: 'cfr-insert', body, position}`
   via `chrome.tabs.sendMessage` to the active supported tab.
   - Schema: `{id: string, title: string, body: string}` array under key
     `prompts`. New entries are unshifted (newest first). `id` from
     `crypto.randomUUID()` with a `p_<ts>_<rand>` fallback.
   - **One-time migration:** `migratePromptsToSync()` in `popup.js` runs
     before the first render. If `local.prompts` exists and `sync.prompts`
     is empty, it copies up; then it sets `local._promptsMigrated: true`
     and removes `local.prompts`. The marker is idempotent — once set,
     migration never runs again, so a user who later empties their synced
     list on a fresh device won't get it repopulated from a stale local
     copy. Don't remove the marker check.
   - **Sync limits to remember:** `chrome.storage.sync` caps total at
     ~100KB, per-item at ~8KB, and ~120 writes/min. Plenty for a prompt
     library, but a single 9KB prompt body will silently fail to save —
     if users start hitting this, surface the error rather than swallowing.
   - Title uniqueness is enforced case-insensitively in `popup.js` — don't
     remove that check; the list reads as an index, not a log. The inline
     edit form skips self in the duplicate-title check so the user can keep
     the existing title while editing only the body.
   - Cards show a truncated body preview (`truncateBody()` in `popup.js`)
     capped at `previewChars` chars, configurable from Settings under
     `promptPreviewChars` (clamped to `[20, 2000]`, default 100). The full
     body is always what gets inserted and what stays in storage.
   - Each card has a ✎ (inline edit) and × (delete) button on one row;
     ✎ swaps the card into an edit form (`buildPromptEditForm`) tracked by
     module-level `editingId` so a `storage.onChanged` re-render doesn't
     clobber an in-progress edit.
   - `position` is `'top' | 'cursor' | 'end'`. `'cursor'` requires that the
     user clicked into the composer before opening the popup — the popup
     steals focus, so `content.js` tracks `lastEditor` / `lastRange` via a
     `selectionchange` listener. Don't try to read the live selection from
     the popup-spawned context — it'll be empty.
   - `findComposer()` picks the bottom-most `contenteditable="true"` /
     `div[role="textbox"]` wider than 100px. This works on both Claude and
     ChatGPT (both ProseMirror). If a site lands with a different composer
     shape, prefer extending this heuristic over hardcoding a selector.
   - Insertion path: `execCommand('insertText')` first, then a
     `beforeinput` `InputEvent`, then clipboard fallback. All three return
     a structured `{ok, error}` to the popup so `flashStatus()` can show
     the right message.
   - The popup auto-closes ~250ms after a successful insert so focus
     returns to the composer.

4. **Feature toggles (Settings tab)** — master `enabled` plus per-feature

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alirahmani93/claude-farsi-rtl](https://github.com/alirahmani93/claude-farsi-rtl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
