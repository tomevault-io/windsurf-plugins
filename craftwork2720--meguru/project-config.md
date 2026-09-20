---
trigger: always_on
description: A KOReader plugin that turns OPDS-PSE page streams (Kavita, Suwayomi) into
---

# meguru

A KOReader plugin that turns OPDS-PSE page streams (Kavita, Suwayomi) into
ordinary KOReader "books". Each book is a small on-disk **marker** file; the
pages come off the network one at a time as they are read. It is a from-scratch
successor to the `meguru.koplugin` beside it — which is the reference for
behaviour, the fallback if this one misbehaves, and **not to be modified**.

## Environment

These are fixed and shape most of the design:

- **Lua 5.1 / LuaJIT.** No `//`, no bitwise operators, no `goto`. The device is
  the only place this code runs.
- **No test framework and no linter.** Verification is manual, in a running
  KOReader. `tools/check.py` (see Development) is the automated guard, covering
  eleven failure modes.
- **Reuse KOReader's own machinery** rather than rebuilding it: `LuaSettings`,
  `DocSettings`, `DocumentRegistry`, and the built-in `plugins/opds.koplugin` for
  Atom parsing and the browser UI. That plugin is **read only** — wrapped at
  runtime, never edited.
- **Module names are global**, so everything lives under `meguru/`. Always
  `require("meguru/feed")`, never `require("meguru.feed")`: both resolve to the
  same file but occupy two different `package.loaded` keys.
- `require` of `opdsbrowser` / `opdsparser` must be **lazy, at the call site** —
  `pluginloader.lua` only adds plugin directories to `package.path` after the
  plugin itself has loaded.

## Layout

```
_meta.lua                 plugin metadata
main.lua                  plugin class: provider registration, menu dispatch, reader install

meguru/
  paths.lua               every path: markers, and the last-resort folder
  fs.lua                  filesystem predicates, directory creation, one raw write
  settings.lua            plugin-wide preferences in G_reader_settings
  association.lua         Meguru's claim on .cbz: the file-type reader association
  sources.lua             read-only view on settings/opds.lua (catalogs + credentials)
  net.lua                 HTTP: GET, the one PATCH, feed fetch + parse
  naming.lua              sanitizeComponent / deriveSeries / glyph / identity digest
  local.lua               the series a .cbz's folder and file name imply
  comicinfo.lua           the metadata a .cbz carries about itself
  marker.lua              marker read/write, naming, collision resolution, series context
  credential.lua          what a credential looks like in a URL: redact / restore
  seriescover.lua         the series' artwork, written once into its folder
  rowcover.lua            the "Meguru this series" row's own artwork, decoded once
  progress.lua            the reader's position, sent back to the server
  pse.lua                 OPDS-PSE: link extraction, template -> URL, page fetch
  feed.lua                reading a series feed: the rel=next walk, identity, order,
                          neighbour
  panel.lua               the panels on a page, and the order they are read in
  viewport.lua            the window over a page, for the panel view that crops nothing
  hook.lua                runtime wraps on OPDSBrowser (sniff, "Meguru this series")
  updater.lua             GitHub releases: check for one, download it, install it

  driver/
    base.lua              driver registry + pure shared helpers
    suwayomi.lua
    kavita.lua
    komga.lua

  doc/
    document.lua          Document subclass: the reading engine
    image.lua             MuPDF decoding with a size cap
    defaults.lua          per-book seeding of kopt_* from plugin preferences

  ui/
    open.lua              "Meguru this series": resume dialog, marker write, open
    reader.lua            everything grafted onto a running ReaderUI
    panelzoom.lua         the panel sequence viewer: nav, pre-warm, page boundary
    menu.lua              the two menu surfaces

assets/
  meguru-this-series.png  optional; the cover drawn on the series row

.github/workflows/release.yml   a tag builds meguru.koplugin.zip and publishes it
```

`assets/meguru-this-series.png` is the one file the plugin ships rather than
writes, and it is **optional** — `meguru/rowcover` answers nil without it and the
browser draws its ordinary placeholder. It is portrait, authored at 2:3 (what
zen-os fits a cover into by default); any size decodes, and a larger one costs
only bytes on disk. The plugin has had artwork before and it was deleted on
purpose — an error-page drawing whose headline named the wrong fault — so the
distinction is worth keeping: this file is the row's *identity*, not a claim
about something that went wrong.

`tools/check.py` is a development aid, not part of the plugin.

Not yet written: `driver/generic.lua` — the `kind = NULL` driver that can
only discover a series by title heuristic and cannot build a canonical
`catalogURL`, so there is no feed to walk for a neighbour. Until `generic.lua`
exists, an unrecognised server is handled by the absence of a driver rather than
by a driver that returns nothing useful.

**Nothing is written to disk but markers** — and one exception, named here
because the sentence above it is the kind that gets quoted: `meguru/seriescover`
leaves a `.cover.jpg` in a series folder, for whatever *outside* KOReader reads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Craftwork2720/meguru](https://github.com/Craftwork2720/meguru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
