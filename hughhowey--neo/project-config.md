---
trigger: always_on
description: NEO is a local word processor for books. It is an Electron app made of plain JavaScript, HTML, and CSS. There is no bundler, no framework, and no compile step. `npm start` runs `electron .`.
---

# NEO — notes for agents

NEO is a local word processor for books. It is an Electron app made of plain JavaScript, HTML, and CSS. There is no bundler, no framework, and no compile step. `npm start` runs `electron .`.

Read [CONTRIBUTING.md](CONTRIBUTING.md) before adding a feature. The product is opinionated on purpose. A change that helps someone finish a book belongs here. A change that adds a panel, a prompt, or a dependency usually does not.

## Rules that override convenience

- Nothing interrupts a writer mid-sentence. No popups, no squiggles, no notifications while typing. Spellcheck is off until the writer asks for a pass.
- Controls stay hidden until hover or keyboard focus.
- Words are never discarded. Deleting text, unbinding a shelf, or losing a trash operation must leave the words recoverable (Darlings, a sibling chapter, or the system trash). `book:delete` uses `shell.trashItem`. If trash fails, leave the folder and show it.
- Books are plain files. No database, no proprietary format.
- The renderer never touches the filesystem. Disk access goes through `window.neo` (`preload.js`) to handlers in `main.js`.
- Do not save UI decoration into chapter HTML. Search highlights, spellcheck underlines, and focus dimming use the CSS Highlight API so they stay out of the file.
- Do not rewrite a chapter that has not changed. Libraries are synced with iCloud and Syncthing. A timer that writes every chapter on an interval will fight the other device.

## Where the code is

| File | Role |
|---|---|
| `main.js` | Window, menus, every filesystem operation, import parsing, PDF, backups |
| `preload.js` | The entire renderer API, `window.neo` |
| `index.html` | Two views: `#bookshelf-view` and `#editor-view`. CSP is `script-src 'self'` |
| `app.js` | The whole UI, in banner-marked sections. Search for the banner before reading the file |
| `styles.css` | All styling. Tokens are CSS variables at the top |
| `covers.js` | Shelf covers in the window: seeded canvas art plus real title type. `window.NeoCovers` |
| `art.js` | Painted covers in the main process. OpenAI only. Title and author are never sent to the image model |
| `i18n.js` | `t()` / `tk()`, shared by main and the window. English source text is the key |
| `spell-worker.js` | Hunspell WASM, forked with `utilityProcess`. Messages: `load`, `check`, `suggest`, `add` |
| `spell-ro.js` | Romanian diacritics, used by the worker. Does not alter the manuscript |
| `locales/<code>.json` | One language. Regional files (`fr-CA.json`) hold only the strings that differ |
| `pocket/` | Capacitor shell. It does not contain its own editor |

`app.js` section banners look like `/*  SAVING  */`. Start there: bookshelf, bound shelves, editor open, typing, poetry, screenplays, placeholders, nav, tabs, outline, outline cards, darlings, counters, saving, refresh, structural undo, find, import, spellcheck, focus, goals, export.

Menus are built in `buildMenu()` in `main.js`. A menu click sends `{ type, ... }` to the window; `app.js` handles it on `window.neo.onMenu`.

## Outline cards

The Outline tab shows the book as index cards (OUTLINE CARDS in `app.js`) unless `library.outlineView` is `'list'`. A script's Outline is always cards, one per scene (`scriptScenes`): heading, length in eighths, cast, and a note in `book.sceneNotes`, keyed by `data-scene-id` on the heading line. Dragging a scene card calls `spMoveScene`.

- Cards come from the manuscript, not a separate structure: each chapter is cut at its `p.scene-break` lines (`chapterSegments`). A section that holds a `p[data-sec-id]` belongs to that note in `book.sectionNotes`; the first section to carry an id owns it, because paragraphs split from a written ghost inherit the id. Others show their first line.
- Moving a card moves its paragraphs and *** between chapter bodies, then `syncChapter` and `orderSectionNotes`. Every move takes a `snapshotStructure` first.
- `syncGhosts` leaves a ghost where it stands and places a note the page lacks before the next section that's there. It never reorders ghosts.
- Loose cards are `book.looseCards`, shown in the right-hand pane while the Outline is up.
- `joinChapter` makes a chapter a section of another (List Tab on a chapter line, or a chapter card dropped on the middle of another). It moves the lines, persists the receiving chapter, and only then deletes the emptied one.
- In the List, Enter always makes a chapter and Tab always makes a section.

Brighter Interface is two settings: `library.uiBright` while writing (and on the shelf), `library.uiBrightAside` on the other tabs, bright unless turned off. `applyBright` picks one on every tab switch.
- The walking note (`walkNoteUpdate`) is an overlay inside `.chapter`, plus a `data-walk` mark on the caret's paragraph that `captureBody` strips. `note.dismissed` hides it for good.

## Screenplays

A book whose `book.json` says `"format": "screenplay"` is a script. Right-click (long-press) a shelf's + for New Script. The SCREENPLAYS section of `app.js` holds the feature; `scripts/screenplay.test.js` tests its rules.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hughhowey/neo](https://github.com/hughhowey/neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
