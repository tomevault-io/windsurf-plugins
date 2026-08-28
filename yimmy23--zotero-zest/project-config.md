---
trigger: always_on
description: Zest is a Zotero 10 plugin (TypeScript, esbuild, zotero-plugin-scaffold). This file is the contract:
---

# AGENTS.md — working on Zest

Zest is a Zotero 10 plugin (TypeScript, esbuild, zotero-plugin-scaffold). This file is the contract:
what the pieces are, what must stay true, and how to prove a change works. `plan.md` holds the design
history and the verified Zotero API facts; the README is for users.

## Layout

```
src/
  index.ts hooks.ts addon.ts   startup / shutdown / per-window bind / pref observers
  api.ts                       `Zotero.Zest.api` — read-only surface for Better Notes templates,
                               Actions & Tags scripts and Run JavaScript. Never throws, never
                               returns an internal type, never writes
  columns/                     every item-tree column + registry helpers + title decoration
  authors/pipeline.ts          resolveRoles → normalize → select → format → decorate
  cite/                        citation counts: Extra format, sources (Crossref/OpenAlex/S2)
  rank/                        journal ranks: local dataset → easyScholar → OpenAlex, cache, Map rewrite
  reading/                     tracker, store (zest.sqlite), heat, read status (manual layer in
                               Extra + the automatic layer derived from the record and Zotero's own
                               last-read stamp), the status picker popup, automation, import/export
  annots/density.ts            annotation summaries for the column and the panel cards
  tags/                        nested tag tree, scope pass, rules, tag context menu
  views/                       getItems filter pipeline, column views, collection counts, reveal guard
  reader/                      the readerCustomThemes repair only — Zest adds nothing to the reader
                               (maintainer's call, 2026-08-23)
  panes/                       item-pane sections, statistics window, annotation matrix
  tabs/                        vertical tab sidebar + groups/sessions model
  graph/                       d3-force panel + author identity (authorIdentity: name
                               clustering + cached OpenAlex ids; authorFetch: bounded
                               authorship top-up); authors/authorMenu.ts is the click menu
                               (library filter + online search) both the panel and the
                               graph open
  core/                        config store, JSON caches, sqlite, HTTP, secrets
  ui/                          stylesheet + accent tokens, icon set, palette, batch runner
  utils/                       Extra lines, CSV, guard, prefs, timers, locale, item helpers
addon/                         manifest, prefs.js, locales (en-US, zh-CN), preferences pane, dialog host
scripts/                       dev-eval.sh, dev-shot.py, phase-c/d/e probes,
                               upgrade-probe.js (destructive; upgrade ordering)
assets/                        icon sources; NOT shipped (build packs addon/** only).
                               favicon.svg regenerates addon/content/icons/favicon{,@0.5x}.png
                               via `rsvg-convert -w 96 -h 96` / `-w 48 -h 48`
```

Everything under `addon/` ends up in the xpi, so an unreferenced file there is dead weight that
users download. Design sources live in `assets/`.

## Invariants

These are not style preferences. Breaking one is a bug even if everything still compiles.

1. **Zest extends Zotero; it never degrades it.** Anything that replaces a native surface (nested tag
   tree, vertical tabs, collection counts) is off by default, reversible, and disables itself when
   its feature probe fails. Title decoration only adds to the Title cell (heat, bold) and ships on;
   it is still reversible and probe-gated. **Zest does not duplicate what Zotero 10 already ships**:
   no reader colour presets, no Creator-column copy, no item-type filter, nothing
   inside the reader — a native feature is the answer, not a second copy (see the 2026-08-23 audit
   in session-notes). But "derived from, consolidates or visualises a native feature" is NOT
   duplication — the maintainer's rule (2026-08-23): Zest's job is exactly that. So the Venue
   column (one column across item types), the Authors columns (Creator column with rules), the
   panel's title / one-line author list / abstract (with the translation plugin's translations shown
   whole — read from Extra, never written) and its row of outbound links all stay. Filters compose with Zotero's own search rather than
   overriding it. Native gestures stay native: a modified click (Shift/Cmd/Ctrl/Alt, non-primary
   button) belongs to Zotero's selection handling — see `isPlainClick` in `columns/registry.ts`.
2. **`Extra` is the user's field.** Write only on an explicit user action, only the line you own, and
   never reformat, reorder or delete anything else — including blank lines and other plugins' records
   (`GSCC:`, `ZSCC:`, `openalex.cit_count:`). All writes go through `utils/extra.ts` or
   `cite/extraFormat.ts`; never use toolkit's `replaceExtraFields`.
3. **Secrets never leave the login manager.** No key in prefs (except the explicit legacy fallback), in
   a log line, in Zotero's URL cache, in an exported bundle, or in an error message. Secret-bearing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yimmy23/zotero-zest](https://github.com/yimmy23/zotero-zest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
