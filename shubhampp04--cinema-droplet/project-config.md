---
trigger: always_on
description: provides; the droplet conforms to the matching protocol for each of them and to nothing it
---

# Tmdb, a Droplet for Droppy

<!-- Written by `droppykit agent`. Add your own notes below; the file is only rewritten with --force. -->

This package is a **Droplet**: an extension that runs inside Droppy, the
Dynamic Island and shelf for Mac, written in SwiftUI against **DroppyKit**.
Droppy loads the built `.droplet` bundle into its own process and draws it on
the notch, the shelf, the lock screen and the menu bar.

- Droplet id: `tmdb`. It is also `TmdbDroplet.id` in Swift and `id` in `droplet.json`; the three must agree or the loader refuses the bundle.
- Swift product: `Tmdb`, a dynamic library. The harness target is `TmdbHarness`.
- SDK checkout: `/Users/jasneet/droppykit` (DroppyKit 1.4.0). Docs online: https://getdroppy.app/docs/droppykit
- Host: Droppy 15.3 or later, or the free Droppy Playground (https://getdroppy.app/download/playground), which loads unsigned bundles.

## The loop

Every change goes through all of this, in order. A droplet can compile,
validate and then draw nothing, so a green build is not the end.

1. Edit `Sources/Tmdb/`. The manifest is `droplet.json`.
2. `droppykit build` writes `.build/Tmdb.droplet`, universal, linked against the
   framework Droppy ships. Never a bare `swift build` for the bundle: it folds a second
   copy of DroppyKit into the droplet, and that bundle loads in the harness and dies
   inside Droppy at dyld with "Symbol not found".
3. `droppykit validate` runs the exact checks the Store's intake runs.
4. `droppykit run -- --shots ./shots --report ./shots/report.json` renders every surface
   to a PNG without opening a window and writes a JSON verdict. Look at the pictures.
   Read `report.json`: `problems` must be empty and every surface you declared must be
   `provided`.
5. Put the bundle into Droppy Playground and confirm it loaded. Copy
   `.build/Tmdb.droplet` to
   `~/Library/Application Support/Droppy Playground/Droplets/tmdb/Tmdb.droplet`,
   relaunch the Playground, and read its Store row: the subtitle is the loader's verdict.

With the DroppyKit MCP server connected, the same steps are the tools `droppykit_build`,
`droppykit_validate`, `droppykit_shots` and `droppykit_install`, and `droppykit_shots`
returns the images inline. This package carries the server in `.mcp.json` (Claude Code)
and `.cursor/mcp.json` (Cursor). Codex: `codex mcp add droppykit -- /Users/jasneet/droppykit/Scripts/droppykit mcp`.
The other tools are `droppykit_manifest` (a static check, no build), `droppykit_docs`
(the guides and a search over the SDK sources), `droppykit_doctor`, `droppykit_new`,
`droppykit_open_harness` and `droppykit_submit`.

`droppykit run` with no arguments opens the harness window for a person: Droppy's own
Settings panel with a page per surface. You cannot see that window. The shots are your
eyes; take them after every visual change.

`droppykit version` says which SDK checkout the scripts come from and which tag this
package pins; `droppykit update` moves both to the newest release. A build that stops with
"no compiled objects" or "DroppyKit.o not found" is an SDK older than 1.2.1: update it.

## Rules

- **Surfaces and conformances agree.** `surfaces` in `droplet.json` lists what the droplet
  provides; the droplet conforms to the matching protocol for each of them and to nothing it
  does not list. Disagreement is the most common reason a droplet validates and then does
  nothing.
- **Every shelf widget declares both widths.** `preferredSoloWidth` and
  `preferredPairedWidth` are required; Droppy refuses a descriptor that leaves either to a
  host fallback. Solo and paired are different compositions, not one view at two widths:
  branch on `context.isPaired`.
- **Layout traits describe the widget's rectangle.** Every number in
  `ShelfWidgetLayoutTraits` is the area the widget draws in, in points at the Regular shelf
  size, exactly what the harness renders; Droppy adds its own chrome around it. `.fixed(150)`
  is a 150-point rectangle, alone and in a row, clamped to 48 through 480. A widget that needs
  more height declares more; it never pads its way out of a clip. A solo widget is never
  narrower than 352 on a notch or 370 on an island, so lay out to `context.availableSize`.
- **No card, no border around the widget.** Droppy paints nothing behind a widget and almost
  every one of its own widgets lays its content directly on the shelf's black. Put no
  background, fill, outline or rounded box on the widget's root view. `notchSurfaceCardFill`
  is for a tile or a chip inside the widget that has to read as raised, never a frame.
- **Lay the widget out like Droppy's.** The root view fills the rectangle
  (`.frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .topLeading)`) with ONE
  padding, `DroppySpacing.mdl`, the same on all four edges and nothing more: the shelf's chrome
  is already outside it. Leading text, trailing `.monospacedDigit()` numbers, rows that span the
  full width, a header row of a 12pt symbol and a 12pt semibold title with the widget's control
  at its trailing end, `DroppySpacing` steps between rows. Declare the height the content
  needs; never leave unused space or fill it with padding.
- **Buttons are Liquid Glass, Droppy's own.** `DroppyCircleButtonStyle` (20pt on an item,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShubhamPP04/cinema-droplet](https://github.com/ShubhamPP04/cinema-droplet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
