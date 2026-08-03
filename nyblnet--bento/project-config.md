---
trigger: always_on
description: enableSharing()). A never-saved starter/template stays dormant so the
---

# Bento — self-contained office documents

One HTML file = the document + viewer + editor. See `README.md` for the vision.
`slides/` is the first app (PowerPoint replacement); docs and sheets come later.

## Architecture (slides/)

- `src/model.ts` — the `bento/slides` JSON document model. This is the format.
- `src/starterdeck.ts` — the showcase starter deck (what a fresh build opens
  with): four 'sd-tile-*' elements morph through EVERY slide (the id-continuity
  demo), one deliberate 'fade' beat exists because entrance staggers/count-ups
  only run on non-morph entries, charts slide + hidden pie state demo the
  bar⇄pie data morph, speaker notes double as the feature tour. Gotchas learned
  building it: line shapes take their color from `fill` (not `stroke` — the
  stroke attr is what morphs tween), and the renderer draws lines horizontally
  across the element box (vertical lines = rotation), keep 96px side margins
  (x ≤ 1184 for right-most content).
- `src/save.ts` — the self-save trick: clone the document at boot (`capturePristine`),
  swap the `#bento-doc` data block, re-serialize. JSON is `<`-escaped (`\u003c`) so it can
  never contain `</script>`. File System Access API first, download fallback.
- `src/autosave.ts` (v0.9.8) — auto-save + local version history, IndexedDB
  (`bento-autosave`, two stores: `recovery` single-latest-per-docId, `versions`
  capped timeline). Editor debounces (2.5s) on `doc` events: writes a recovery
  snapshot (plain doc JSON, NOT the shell) + a throttled version, and — when a
  FSA handle exists — silently rewrites the real file (`writeUpdatedFile`, shows
  a "Saved" tag). On boot `checkRecovery` compares the latest snapshot's
  `docContentKey` (content minus volatile modified/collab fields) to the loaded
  doc; a mismatch shows a Restore/Discard banner. Encrypted decks are NEVER
  snapshotted to IndexedDB (plaintext-to-disk) — their file write-back stays
  encrypted. readonly players skip autosave. Version history UI in the About
  dialog; restore = `store.replaceDoc` (undoable). Keyed by docId, so recovery
  needs a stable docId (saved files) — the fresh-each-load anonymous demo won't
  cross-reload-recover, by design.
- `src/render.ts` — single model→DOM renderer shared by editor canvas, thumbnails, and
  Reveal sections. Elements carry `data-el-id` (editing) and `data-flip-id` (morph).
  **Morph key (v1.0.7)**: `data-flip-id = el.morphId || el.id`. `id` stays the
  stable identity (selection, connector/comment anchors, CRDT node key); the
  DEFAULT morph key is still `id` (the duplicate-a-slide idiom, old files have no
  `morphId`), but an optional `morphId` re-targets the pairing WITHOUT mutating
  `id` — so two independently-created elements on different slides can morph. This
  one line is the whole engine change; everything downstream reads `data-flip-id`.
  Edited in the panel's Morph section (`panels.ts buildMorphProps`): a "Morph id"
  field (writing the element's own id clears the override) + a "Pair with" picker
  that adopts another slide element's key; both reject a key that would collide
  with another element's effective key on the SAME slide (present.ts maps by flip
  id, so same-slide dupes would break pairing).
  **Dynamic fields (v0.9.12, doc-props v1.0.2)**: text resolves `{{page}}`,
  `{{pages}}`, `{{title}}`, `{{date}}`, `{{time}}` plus the document-property
  tokens `{{author}}`, `{{company}}`, `{{subject}}`, `{{event}}` at render time
  (`resolveFields`); page/pages take a zero-pad width (`{{page:2}}`→"06"). The
  doc-props live in optional `doc.meta` (`{author,company,subject,event,keywords}`
  — additive, backward-compatible; old files lack it and tokens resolve empty)
  and are edited in the About dialog's "Document properties" section (title stays
  top-level). `renderSlide` auto-fills `RenderOpts.fields` via `fieldContext(doc,
  slide)` (page = 1-based position among NON-state slides).
  The MODEL stores the raw token; only output is resolved, so inserting/removing
  slides re-numbers everything. Editing gotcha: the canvas renders resolved, but
  `canvas.startTextEdit` swaps the token BACK to raw `el.html` while editing so
  authors edit the field, not the computed value. The starter deck's furniture +
  ghost numerals use `{{page:2}}` (they can't drift). Groundwork for the office
  suite's field/cross-reference system.
- `src/editor/clipboard.ts` (v0.9.9) — system-clipboard copy/paste. Bento content
  is written as JSON tagged `__bento:"clip"` (kind elements|slides) with referenced
  assets/fonts embedded, so it round-trips across decks/tabs; asset-key collisions
  remap. Editor: ⌘C copies selected elements, or the current slide when nothing is
  selected (→ `navigator.clipboard.writeText`); a document `paste` listener handles
  external images (embed as data-URI image element), plain text (→ text element),
  and Bento payloads (insert elements on the current slide / slides after it, fresh
  ids). Pasted slides drop `stateOf`. Guarded to skip when a text field is focused.
  Also v0.9.9: a `?` help overlay (editor.openHelp — shortcuts + tips, topbar ?
  button) and richer toolbar tooltips.
- `src/anim.ts` — in-house animation engine (no GSAP): to/fromTo tweens with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyblnet/bento](https://github.com/nyblnet/bento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
