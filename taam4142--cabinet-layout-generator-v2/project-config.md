---
trigger: always_on
description: `SKILL.md` says *what the system is and how it's wired*. This file says *what must always be true* —
---

# CLAUDE.md — rules for building & extending the Cabinet Layout Generator

`SKILL.md` says *what the system is and how it's wired*. This file says *what must always be true* —
the laws the code and (later) the AI obey. Read both before writing code. Both live at the repo root
and are the canonical copies for anyone (human or AI) working in this codebase.

---

## 0. The one law (above all others)

> **The AI never invents geometry, connectivity, or part data. It only structures/interprets input.
> Deterministic code draws. A human reviews the result in CAD.**

Placement, dimensions, and part identity must be exactly what the validated input says — never a
model's guess. A confidently-wrong terminal or clearance ships to a panel shop. If a value is unknown,
it is **flagged for human confirmation**, never silently filled.

---

## 1. AI in v1 — socketed but OFF (provider-agnostic)

- **v1 uses no AI.** The tool is fully functional manual-first. Do not add an AI dependency to any
  build/run path.
- **Socket design** for later:
  - A single **`assistant` interface** with a **deterministic default implementation** (manual entry /
    the future type-to-place parser). All AI lives behind this one module.
  - An **`AI_ENABLED=false`** flag and a **`provider`** field (never hard-wire a vendor).
  - When enabled later, route through a **serverless proxy** with the key in an env var (direct
    from-browser calls hit CORS / key-exposure problems). Plan AI as a separate, likely-paid,
    provider-agnostic key.
- **AI's only ever jobs** when on: structure fuzzy input into the validated schema (parse a messy BOQ
  into typed rows, normalize tag names, infer a type from a part number), interpret extracted data, map
  intent to parameters. **Never** emit coordinates, sizes, or connectivity as ground truth.

---

## 2. Validation rules (the data model is the contract)

Validate on every load, edit-commit, and before every export. Reject or flag — never silently coerce.

**Structural**
- Every `element.lib_key` / `group.lib_key` resolves to a library item; unresolved = error.
- Every `label.anchor` (`element:<id>` / `group:<id>`) points to an existing element/group.
- IDs are unique within their arrays; `group_id` on an element references a real group.
- `plate.origin == "top_left"` (the only editor convention; conversion to DXF bottom-left is §4).

**Dimensional / physical**
- `width_mm`, `height_mm`, `length_mm` > 0. Duct `width_mm` ∈ {30,40,60} faces unless a custom dialog
  value was set; `label_h_mm` is label-only (no geometry — never let it affect layout).
- Equipment size is read from the library, **not** editable by drag. Size changes only via typed mm.
  Reject any attempt to free-resize an equipment element.
- Rotation uses the **rotated bounding box** for spacing (90°/270° swap W×H).
- `gap_between_equipment_mm` default 0.1; `clearance_equipment_to_duct_mm` default ≥3. Per-element
  overrides allowed; clearance below default is **flagged "too tight"**, not blocked.

**Layout integrity**
- Editing one `gap_before_mm` re-flows **only downstream elements in that local run** — assert no other
  run and no upstream element moved.
- Plate fit: on overflow, return an **itemized** message (what didn't fit, by how much). Boundary is
  **warn-but-allow** — flag, never silently crop or overlap.
- Overlap detection ignores **non-physical markers** (text labels, and `label_plate` parts that sit
  coincident on their host by design) — flag real collisions, not intentional coincidence.
- A `pair_id` (e.g. stopper + label) is a locked unit: move / rotate / delete act on all pair-mates,
  but each stays a distinct part for the BOM. `locked` pins a part so auto-pack flows others around it.
- Auto-pack must honor `locked` elements and flow the rest around them.

**Export integrity**
- Exports render from the JSON model, never the canvas. Assert no editor artifacts (selection handles,
  grid, snap guides) appear in output.
- DXF: **every part is a named block** `EQ_<lib_key>` (uploaded DXFs re-embedded; rect/symbol parts a
  unit rectangle) inserted with the correct transform so CAD *Count Block* / a BOM can tally each part.
  Tags and label/custom centre text are separate TEXT entities. Layers `DUCT` / `EQUIP` / `TEXT` / `GROUND`.
- At scale 1:100, geometry is written at 1/100 size but **dimension text strings carry the real value**.
- Coordinate origin converted in **exactly one place** (§4).

---

## 3. AI structuring rules (for when the socket is turned on — write the prompt to these)

When/if AI is enabled, its output is **always** the validated JSON schema (SKILL.md §4) and must pass
§2 validation before anything is drawn. Prompt rules:
- **Extract, don't invent.** Map only values present in the input. Unknown dimension/part → emit a
  `confirm: true` marker and a human-readable note; never fabricate a number.
- **Normalize, flag conflicts.** Standardize tag names to the observed vocabulary; when two inputs
  disagree, surface both and flag — do not pick silently.
- **Type inference is a suggestion, not a fact.** Inferring a component type from a part number is
  allowed, but the type and any datasheet dimension is presented for human confirmation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Taam4142/cabinet-layout-generator-v2](https://github.com/Taam4142/cabinet-layout-generator-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
