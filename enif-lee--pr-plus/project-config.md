---
trigger: always_on
description: Agent-oriented guide for build, verify, architecture, and tests. Humans can use it too.
---

# AGENTS.md — pr+ (GitHub PR review extension)

Agent-oriented guide for build, verify, architecture, and tests. Humans can use it too.

## What this repo is

**pr+** is a Chromium MV3 extension for GitHub PRs: stack tree on `/pulls`, and a fast in-page **modal / embed shell** for Conversation, Diff, and merge. UI is React (modal); network and extension plumbing live in background + content bridge + host.

Demo / e2e target repo: `enif-lee/pr-plus`. Conversation/Diff fixture: merged PR `#19` (stack root DEMO-300; former `#7` closed) via `/pull/N`. List click: open `#13` (`LIST_PR`). Meta write-through: open `#1` (`META_PR`).

---

## Verification workflow (build + extension reload)

After code changes, **do not rely on a full Chrome restart** for every iteration. Prefer:

1. **Rebuild** the extension artifacts from the repo root.
2. **Reload the extension** in Chrome via `chrome://extensions` (circular reload button) — keeps the browser process and tabs; picks up a new service worker + content scripts on next navigation/injection.
3. Re-open or soft-reload the GitHub tab under test.

### Build

```bash
# Full pipeline (pure → content-ts → fetch → background → sw → host → bridge → app-parts → modal)
npm run build

# Faster when you know the surface area:
npm run build:pure          # modal/lib/*.ts → pure IIFE (injected in SW / pages)
npm run build:fetch         # fetch-api.ts → fetch-pulls.js
npm run build:sw            # background.sw.js (+ bundle dual-write)
npm run build:host          # host modules → pr-modal-host.js
npm run build:content-bridge
npm run build:modal         # React modal bundle + CSS
```

Service worker entry (manifest): **`src/background.sw.js`**. Stale SW is a common source of “I fixed it but still see old GraphQL/behavior” — always reload the extension after `build:sw` / `build:fetch` / `build:pure`.

### Manual: chrome://extensions reload (no browser restart)

1. Open `chrome://extensions` (or edge://extensions).
2. Enable **Developer mode**.
3. Find **pr+** (loaded from this workspace, typically “Load unpacked” → repo root or packaged dist).
4. Click the **reload** (circular arrow) control on the card.
5. Return to the GitHub tab → hard refresh the page (`Cmd+R` / `F5`) or re-open the PR so content scripts reinject.

Optional: `chrome.runtime.reload()` from an extension context (e2e may dispatch `prp-reload-extension` when `PRP_E2E_RELOAD_EXT=1`). That still does **not** require quitting Chrome.

### When a full browser restart *is* needed

- Extension was never loaded / path changed / “Load unpacked” broken.
- Profile locks or corrupted agent-browser session (`npm run browser:close` then re-open).
- SW refuses to activate after reload (rare); then quit Chrome-for-Testing / agent-browser session and relaunch with the extension load path.

### Agent-browser / e2e note

E2e uses a **shared browser session** and often only **soft-resets** tabs + IDB between suites. After rebuilds that change the SW, either:

- reload the extension via `chrome://extensions`, or  
- set `PRP_E2E_RELOAD_EXT=1` so global setup dispatches extension reload, or  
- `agent-browser close --all` and start a new session (heavier).

Prefer **build + extensions reload** over restarting the whole browser for day-to-day verification.

---

## Architecture (overview)

### Domain / UI SoT (host-data-first)

- **Domain SoT:** host open-session detail store → React reads `detail` prop / `useDomainDetail()` (no `localDetail` mirror).
- **Mutations:** `src/modal/commands/*` — API success → narrow `onPatchDetail` ack (`applied|stale|failed`; void ≠ applied).
- **Settled set-authority:** `mergeCommentsHostFirst` / `src/modal/lib/set-authority.ts`; no durable `_dropPending` latch.
- **UiStore:** Zustand `modal-store` / `ui-store` — layout/drafts/focus only (no PrDetail domain blob).
- **Verify:** `npm run build:pure && build:host && build:modal`, then chrome://extensions reload after SW/host/pure changes.

### UI labels (i18n)

- **User-visible UI labels** (buttons, badges, section titles, empty states, toast copy, aria-labels shown as text, timeline narratives) **must** go through `useT()` / `t('key')` and the catalogs in `src/modal/lib/i18n*.ts`.
- Do **not** hard-code English (or any locale) strings in JSX/TS for UI chrome when adding or changing labels.
- Add new keys to the appropriate catalog (`i18n.ts` core, `i18n-chrome.ts`, `i18n-residual.ts`, …) for **en + ko + ja + zh_CN**.
- Non-UI exceptions: log messages, GraphQL operation names, test fixtures, pure algorithm comments.



```
┌─────────────────────────────────────────────────────────────┐
│  github.com (page)                                           │
│  content scripts → list tree / open toggle / embed host      │
│  content-bridge  → messages to SW (fetch, resolve, …)      │
│  pr-modal-host   → open/close, progress, detail store, props │
│  modal React app → Conversation / Diff / composers           │
└───────────────────────────┬─────────────────────────────────┘
                            │ chrome.runtime messages
┌───────────────────────────▼─────────────────────────────────┐
│  Service worker: src/background.sw.js                         │
│  · GitHub REST + GraphQL (fetch-api / fetch-pulls)           │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enif-lee/pr-plus](https://github.com/enif-lee/pr-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
