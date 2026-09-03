---
trigger: always_on
description: validates a document, because the wire is one object per line and never anything else.
---

# Flea

The fastest GUI file manager on Linux, keyboard first, native to Omarchy.
P0 is the local browser, and remotes, search and disk operations have since landed on
top of it. Encryption and Flea's own terminal interface are later phases and are not in
this tree yet: `flea --tui` says so and exits 2.

## The five load-bearing rules

1. **Every per-file operation stays scoped to the viewport.** Icons, MIME sniffing and
   thumbnails all stay inside the visible rows. The whole margin over the field is that
   the competition stats every entry in the directory and Flea stats only the ones a
   screen can hold. Measured: the field stats 100,000 entries, Flea's first window
   stats 350. `ui/Pane.qml` owns the viewport-sized held window and calls
   `ui/Backend.qml`'s `window(start, count)` request only when that window drifts.

2. **The QML model is an integer count, never a list.** `ui/Pane.qml` binds its
   `ListView` to `model: root.total`, so it instantiates and recycles only the delegates
   the viewport needs. A `Repeater` over a fetched window was tried first and dropped
   220 of 480 frames, because swapping the window destroys every delegate it held.

3. **`list` answers with the first screenful, unasked.** `ui/Backend.qml` sends one
   `list` request and emits its `listed` and initial `rows` replies; `ui/Pane.qml`
   consumes those replies as a pair before accepting the new held window. Making the
   client ask for it separately costs a full round trip that lands mid first-paint.
   Measured: 64 ms if asked for afterward, 4 ms riding along with the listing.

4. **Prewarm stays disabled until it is both safe and faster.** The `flea --prewarm`
   producer exists, but the proposed UI reader measured a 485 ms median against 439 ms
   for the live path. After its removal, ignored experimental generation measured
   469 ms against 437 ms live. Production `ui/shell.qml` intentionally calls only
   `pane.open(start)`, and `tools/flea-first-paint` preserves the comparison. The reader
   was rejected as slower and stale-capable; it remains disabled until the wire carries
   the requested path and a new measurement proves a real win.

5. **Vulkan now, lazy multimedia later.** `ui/shell.qml` sets
   `QSG_RHI_BACKEND=vulkan`, which costs 2.4x less memory than the OpenGL default and
   initialises 35 ms faster, with identical frame timing. Preview and QtMultimedia
   are now in the tree and the laziness held: `ui/PreviewMedia.qml` is the only file
   that imports QtMultimedia, reached through a `Loader` built by the first press of
   play, because QtMultimedia costs 20 MB before it plays anything.

6. **A hidden view is not a free view.** `visible: false` does NOT stop a QML view doing
   model work: it keeps its geometry, stays bound to the listing, and pays per row. All
   three views were instantiated unconditionally and hidden with `visible:` alone, and
   the cost was measured at **110 bytes a row with one view instantiated, 232 with two
   and 262 with three**, on the 100,000 file fixture. Gating the two non-default views
   behind `Loader`s was MEASURED to recover 14.7 MB at 100k and take the listing from 264
   to 147 bytes a row, and was then **REVERTED and is not in the tree**. The repair needs
   `Pane.qml` split first, because two `Loader`s inline exceed the 400-line hard cap, and
   that split broke `tests/ui.sh` to 20 of 30 and then 27 of 30: worse after a fix is the
   signal to stop. It is a deferred ticket with a measurement attached rather than a
   shipped change. The recovery figure is also a list-view number and not a whole-product
   one, taken on a build whose other two views did not render. **The same fact
   had already appeared as a keyboard bug** and been patched at the symptom: a hidden
   view holding focus swallowed the keys the visible one should have had, which is why
   `focus:` is toggled explicitly beside `visible:`. A hidden view that can steal a
   keystroke is a hidden view that is fully alive.

7. **The media bracket got a real rival on 2026-09-03, and two columns felt it.** Measuring
   `strata`'s work properly moved it from unranked into the table, and Flea's placements moved
   with it: memory went from fifth of six to **sixth of seven** at 112.6 MiB against strata's
   **70.3 MiB**, and the CPU lead, which read 3.61x over `nemo`, is now **1.28x over strata**,
   1.42 s against 1.82 s. Both columns Flea wins are still won, settled listing first of six and
   CPU first of seven, and the 100,000 file bracket did not move at all. The operator ruled this
   acceptable with an optimization pass to follow, so it is a deferred ticket and not a defect.
   Item 6 above is the first place to look: its `Loader` gating was MEASURED to recover 14.7 MB at
   100k and is not in the tree, and memory is the column that actually lost a place. Read its
   caveat first, because that figure is a list-view number rather than a whole-product one, and
   the split it needs broke `tests/ui.sh` twice.

## Two-phase listing

Phase 1 (`scan.rs`) reads only names and `file_type()`, which is backed by `d_type` in
the `getdents64` buffer libstd already read to produce the directory iterator, so

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thisisgm/flea](https://github.com/thisisgm/flea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
