---
trigger: always_on
description: Project-local guidance for working in this repo. Read alongside `README.md`
---

# CLAUDE.md — Noteback engineering notes

Project-local guidance for working in this repo. Read alongside `README.md`
(what/why), `CONTRACTS.md` (the runtime module API + behavioral invariants), and
`docs/design.md` (the original design).
This file records the **non-obvious gotchas** — things you can't infer by reading
the code, that have already bitten us once.

## Hard constraints (do not break)

- **Zero RUNTIME dependencies, no build step, no TypeScript.** The shipped code
  (`bin`, `src`, `skills`) loads unpacked exactly as written — never add a bundler,
  a framework, or a `dependencies` entry, and never `require` a package from
  `src/`. Tests run on the **Node built-in runner** (`npm test` → `node --test`).
  The **one** allowed exception is `devDependencies`: Playwright backs the browser
  e2e (`test/e2e/`, `npm run test:e2e`) that covers overlay DOM behaviour the Node
  suite can't. It is test-only and never reaches users (`files` ships `bin`/`src`/
  `skills` only). Needs the browser binary once: `npx playwright install chromium`.
- **One runtime, two modes.** The annotation engine in `src/runtime/` runs both
  as the extension content script (`ChromeStorageAdapter`) and inlined into a
  saved canvas file (`InFileStateAdapter`). Anything in `src/runtime/` must work
  in **both** — no `chrome.*` access, no extension-only globals. Mode-specific
  code lives in `src/content/`, `src/adapters/`, `src/canvas/`.
- **Pure-logic modules** (`anchor`, `state`, `markdown`) must run under Node *and*
  the browser (UMD-lite dual export) so they stay unit-testable. Keep them
  DOM-free.

## Gotchas that already bit us

- **CSS transition out of `display:none` does not reliably fire.** The comment
  chip's entrance is a **keyframe animation restarted by a forced reflow**:
  `el.classList.remove('nb-in'); void el.offsetWidth; el.classList.add('nb-in')`.
  Don't "simplify" it back to a `transition` — it'll snap in with no animation.
- **The comment chip is debounced (~340 ms).** A `setTimeout` is re-armed on each
  `selectionchange` and the anchor is re-resolved on `mouseup`. Two consequences:
  (1) live/Playwright tests must wait ~380 ms after selecting before the chip is
  clickable; (2) `commitPopover` is **async** (`await persist`) — a test that
  creates two comments synchronously will have the second reuse the first
  anchor (because `onSelectionChange` early-returns while a popover is open).
  Await the first commit.
- **Composer vs. sidebar outside-click are opposite on purpose.** The composer
  closes **only** via Cancel / Save / Escape (never outside-click); the sidebar
  **does** close on outside-click (guarded). See `CONTRACTS.md` §3.5. Don't
  "unify" them.
- **Markdown line refs are computed from the document markup**, not the DOM. The
  full (uncondensed) quote is located in `docHtml`; long quotes are condensed for
  *display* only. If a line ref and the quote ever disagree, **the quote wins** —
  it's the anchor; the line number is a convenience.
- **Line-number semantics differ by mode.** Embedded canvas → doc-content-relative
  (`#noteback-doc-root` innerHTML, line 1 = first body line). Extension →
  `documentElement.outerHTML` (file-absolute, tracks the opened file). Same
  `toMarkdown`, different `docHtml` origin. This is a deliberate, documented
  tradeoff — don't try to "fix" one to match the other.
- **Doc identity is the BAKED doc-id; a version is hashed from the CLEAN, pre-paint
  content root.** A draft's identity is the explicit `data-noteback-doc-id` baked on
  `#noteback-doc-root` (extension pages Noteback didn't author fall back to a per-URL
  minted id under `nb:url:<href>`). Within that doc-id, a *version* is keyed by a
  content hash over `#noteback-doc-root` `textContent` (`createHistoryStateAdapter`'s
  `contentText`), read before highlights are painted — never recompute it from the live
  DOM after `<mark>` wrappers are added, or the hash shifts (and the draft splinters
  into a new version). When the text is too short to hash, the version key falls back to
  `h0:<docId>`.
- **`window.localStorage` access can THROW (not just be absent) on `file://`** or
  when storage is blocked — and `file://` is the primary canvas use case. The
  `EMBEDDED_BOOT` builds the localStorage-backed kv store (`lsStore`) inside a
  `try/catch`; on failure `lsStore` is `null` and `createHistoryStateAdapter` degrades
  to the in-file `InFileStateAdapter` (comments still work, just no version history).
  Never reference `window.localStorage` raw in the boot guard, or a blocked store
  crashes the whole canvas mount (it did once — the overlay never appeared).
- **`file://` localStorage is one shared bucket** across all local canvases (Chrome).
  Keys are namespaced and keyed by the explicit doc-id (`nb:doc:<docId>`) /
  content-hashed version key (`nb:ver:<versionKey>`), with `nb:url:<href>` for
  per-URL minted ids (extension only), precisely so distinct documents don't collide in
  that shared bucket.
- **History snapshots the WHOLE clean document ONCE, at a version's first comment —
  there is no per-comment fragment/"section" extraction.** `snapshot-capture.js`
  `captureCleanDoc` clones `documentElement`, strips `[data-noteback-ui]`, **unwraps**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alekkowalczyk/noteback](https://github.com/alekkowalczyk/noteback) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
