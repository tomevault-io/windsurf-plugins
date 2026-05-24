---
trigger: always_on
description: **Working on:** v1.0 release preparation.
---

# FusionConstraints — ConstraintLens Add-in

## Current Status

**Working on:** v1.0 release preparation.
**Version:** 1.0.0 (manifest + commit must always match) — entire v1 Polish Backlog (#1–#21) complete.
**Next step:** PC test v0.2.12 end-to-end; tag v1.0 and merge to main.
**Convention:** Every commit that bumps the version string must also update `ConstraintLens/ConstraintLens.manifest` `"version"` field so Fusion shows the correct version.
**Blocked by:** Nothing.

### What's verified working (all PC tests + session history)
- Add-in loads, palette docks, populates without Refresh click.
- Geometric constraints list with click-to-select (row body) and select-constraint (icon glyph click).
- Dimensions list (Angular, Linear, Diameter, etc.) with parameter expression in accent color.
- Inline dimension expression edit via pencil icon; Enter commits, Esc cancels.
- Double-click row → opens native Fusion edit dialog (all dimension types via `SketchEditDimensionCmdDef`; Offset/Pattern via dedicated command IDs).
- Implicit endpoint joins as pseudo-rows with implicit badge AND ⊘ lock icon.
- Tangent spline+line row highlights both objects on click (token-based selection).
- OffsetConstraint row lists curve chips; label: `Offset (1→1 curves, 30 mm)`.
- SketchOffsetCurvesDimension row lists curve chips; double-click → offset edit dialog.
- Sketch status banner (name on own row, fully/under-constrained state with color).
- M-1 defensive guard (MidPoint accessor) — both rows render; no crash.
- Button in `SketchConstraintsPanel` (Sketch tab → Constraints panel).
- "Show underconstraint elements" button — calls `executeTextCommand("Sketch.ShowUnderconstrained")`; result as toast.
- Filter bar — client-side by label/kind/entity chips; section headers show `(N of M)`.
- Find button — canvas selection → palette highlight (blue border + scroll); entity readout strip.
- Chip click → sets filter AND selects entity on canvas.
- Bulk delete — checkboxes, "Delete N" + "Clear" buttons, confirm dialog with Ctrl+Z note.
- Collapsible sections — chevron toggle; state preserved across refreshes.
- Invisible entity chips — dimmed + dashed border + "hidden" badge.
- Native Fusion icons (dark variants) — constraints, patterns, polygon, dimensions; 24×24 px.
- Circular/Rectangular pattern inline edit (count, spacing/angle via `ModelParameter.expression`).
- Polygon in "Patterns and figures" section; center chip + line chips; fallback label if center inaccessible.
- Filter matches entity chip labels (e.g. typing "Line 3" finds all constraints involving Line 3).
- Find works for dimension rows (indexes `row.token` not just chip tokens).
- GUI: sketch name on its own top row; buttons on separate second row with `flex-wrap`.
- PolygonConstraint `lines` iterated via `_iter_curves_into_chips()` (SketchLineVector has no `.count`).

### Known sub-issues to keep on radar
- Offset-of-spline creates internal control geometry that Fusion doesn't render. The row still appears in ConstraintLens with a dimmed "hidden" chip; clicking the row still selects the hidden entity — Fusion's native behaviour. (backlog #8 resolved)
- `OffsetConstraint.distance` returns None in the January 2026 build. Fully mitigated — label uses expression from the matched `SketchOffsetCurvesDimension` parameter. (backlog #9 resolved)
- `PolygonConstraint.centerSketchPoint` returns None in the January 2026 build. Mitigated — fallback chain tries `center`/`centerPoint`; if all fail, label is `"Polygon (N sides)"` with no error shown.
- `SketchLineVector` (returned by `PolygonConstraint.lines`) has no `.count` property. Must use direct iteration.

---

## Project Overview

A Fusion 360 Python add-in that docks a panel listing every constraint in the active sketch — with click-to-select, delete, and over/under-constrained status. Fills the UX gap of having to hunt tiny on-canvas glyphs to audit a sketch.

- **Language:** Python 3.14 (Fusion January 2026 build), vanilla JS palette (no framework).
- **Distribution:** GitHub Releases only (zipped `ConstraintLens/` folder). No App Store.
- **Workspace:** Solid workspace only (MVP). Button lives in `SketchConstraintsPanel` (Sketch tab → Constraints panel).
- **Spec:** `SPEC.md` — complete, all 5 open questions resolved.

---

## Architecture

```
ConstraintLens/
├── ConstraintLens.manifest       Fusion add-in manifest (id, version, runOnStartup)
├── ConstraintLens.py             Entry point — delegates to lib/lifecycle only
└── lib/
    ├── lifecycle.py              Command + palette creation, message routing
    ├── events.py                 GC-safe event handler registry (M-7 guard)
    ├── dispatch.py               21-row constraint type dispatch table + dimension dispatch
    ├── scanner.py                Sketch enumeration → JSON payload
    ├── labels.py                 EntityLabeler: token→"Line 3" map per scan
    ├── selection.py              ui.activeSelections helpers
    ├── actions.py                delete_constraint() with isDeletable check
    ├── tokens.py                 token_of() / resolve() wrappers
    └── messaging.py              palette.sendInfoToHTML / parse_incoming + M-8 guard
palette/
    ├── index.html                Shell; initial "Loading…" state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Botrops1/ConstraintsLens](https://github.com/Botrops1/ConstraintsLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
