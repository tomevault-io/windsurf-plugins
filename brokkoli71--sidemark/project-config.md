---
trigger: always_on
description: > **Read this file completely before doing anything.** It is short on purpose:
---

# CLAUDE.md — working on Sidemark

> **Read this file completely before doing anything.** It is short on purpose:
> everything in it is something you can break by accident, and a rule you
> skimmed past is a rule you will break. Detail lives in `docs/` — follow the
> link for the area you are touching, and only that one.
>
> **This file has a budget: 400 lines.** CI fails if it grows past it. That is
> deliberate — it was 2254 lines once and sessions stopped reading it, which is
> how the rules below came to be missed. If your addition does not fit, it
> belongs in `docs/<area>.md`, in a code comment at the thing it describes, or
> in `ideas.csv`. **Never** add a changelog entry here.

## The rules

Break any of these and the damage is silent or expensive. They are first
because position matters.

1. **Never run the tests against the live desktop session.** Always
   `./run_tests.sh`, which starts an isolated Weston. A bare `pytest` is
   refused by `conftest.py` for this reason. GTK4 has no offscreen backend —
   never `GDK_BACKEND=offscreen`.
2. **Never show a `Gtk.Popover` in a PRESENTED test window.** It needs a
   Wayland surface headless Weston cannot give, weston dies, and every later
   window test fails at `Gtk couldn't be initialized` looking like a bug in
   itself. Use `_run_in_window(present=False)` and assert on the model.
3. **Both modes, always.** Every feature is for PDF *and* text-first pages
   unless there is a stated reason it cannot be. A feature missing on one side
   reads as a bug. If one side genuinely cannot have it, say so loudly in
   `ideas.csv`. (Only two stated exceptions today: linked page notes and
   bookmarks, both because a text page has no page structure.)
4. **Whether it LOOKS right is the user's call.** Do not screenshot the app to
   judge layout, spacing or a new widget. Build it, then hand over a short
   numbered checklist. Screenshots are for factual yes/no questions only.
5. **You cannot drive gestures on this machine** — there is no
   `wtype`/`ydotool`. Anything gesture-, pen- or undo-shaped is **not
   verified** until the user runs it. Say so rather than implying otherwise.
6. **Test what could break by ACCIDENT**, not what someone would change on
   purpose. An assertion naming a tuned value only fires when somebody edits
   it deliberately. Constants belong in assertions as **bounds** or
   **identifiers**, never as expected values. Asserting a *proxy* for the
   behaviour is the same trap in a different coat.
7. **Do not merge or push the `deck` branch into `master` without asking.**
8. **Editing `ideas.csv` from a script: force LF** (`csv.writer(f,
   lineterminator="\n")`, read with `open(p, newline="")`). Python writes CRLF
   by default and churns all ~190 rows into the diff. The **Issue** and
   **Hash** columns belong to `extras/sync_issues.py`; never hand-edit them.
9. **Maintain the docs you invalidate, in the same change** — this file, the
   right `docs/` guide, `ideas.csv`. But keep this file lean: replace facts,
   never append.

## What this is

A **single-file GTK4/libadwaita Python app** (`sidemark.py`, ~28k lines): a PDF
annotator with a live Markdown notes panel, built for lecture notes and
presenting. One window, two document modes (PDF + text). There is no other
source module on this branch. Deps: PyGObject/GTK4/Adw/GtkSource, PyMuPDF
(`fitz`), cairo, numpy.

Files stay plain: `.pdf` + `.md` sidecar notes, `<name>-ink.json` ink sidecars.
The `.md` names its PDF with an `![[name.pdf]]` embed line at the top.

Launch a checkout standalone: `SIDEMARK_STANDALONE=1 /usr/bin/python3
sidemark.py [FILE]` — the env var bypasses the running single instance.
`sidemark --version` says which copy is running when that is in doubt.

## Where things are

- `PDFCanvas` — the page canvas: ink, lasso, anchors, zoom/pan, images.
- `MarkdownNotesView` — the live-Markdown editor (`\alpha`→α, `x^2` scripts;
  source text stays intact, rendering is display-only).
- `TextPageView` — text-first mode: an A4 Markdown sheet you can draw on.
- `DocumentSession` — one open document (one tab). `PDFEditorWindow` owns an
  `Adw.TabView` of them and **proxies the active session's attributes onto
  itself** (`_session_prop`), so window code reads `self.canvas` and follows
  the active tab. New per-document state goes in `DocumentSession.STATE` /
  `WIDGETS`, kept in sync with that proxy list.
- **Modes**: a tab is PDF or text-first (`doc_mode`). Which header chrome each
  shows is a table (`_MODE_CHROME`), not per-mode `if`s — extend the table.

## Read before you touch

Each guide is the invariants for one area. Read the one you are working in.

| touching | read |
|---|---|
| buttons, chords, stylus, touch, the toolbar | `docs/input.md` |
| the pen, stroke shape, smoothing, latency | `docs/ink.md` |
| the notes editor, maths, links, the sheet, `GtkTextView` | `docs/notes-text.md` |
| page navigation, bookmarks, search, thumbnails, hidden pages | `docs/pages.md` |
| pasted or imported images, the PDF image layer | `docs/images.md` |
| share to phone, the phone as a tablet, the transport | `docs/share.md` |
| Ctrl+R reload, app copies, autosave, logging, the watchdog | `docs/lifecycle.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brokkoli71/sidemark](https://github.com/brokkoli71/sidemark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
