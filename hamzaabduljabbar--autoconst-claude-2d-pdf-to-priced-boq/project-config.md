---
trigger: always_on
description: > This file is read by Claude at the start of every session in this project.
---

# CLAUDE.md — 2D PDF Drawing → Priced BOQ
# AutoConst | Antigravity Project Brain

> This file is read by Claude at the start of every session in this project.
> It defines what the project does, the pipeline stages, and the output format.

---

## Quickstart (for a demo / first-time user)

You need two things in place:
1. A drawing PDF in `inputs/` (any name — Claude will pick it up)
2. A rate library at `rates/rate-library.csv` (copy `rate-library.template.csv` and fill in prices)

Then say to Claude:

> "Run the takeoff on my PDF and produce a priced BOQ. Save to `outputs/`."

Claude executes the pipeline and hands back a formatted Excel BOQ.

### For Claude — what to do when the user asks to run the pipeline

When the user asks something like *"run the takeoff"*, *"do the takeoff"*, *"price this drawing set"*, or *"generate the BOQ"* — this is a single request: run the WHOLE sequence below in order without stopping to ask between stages. The user should never have to name a subcommand; you orchestrate them.

**Step 0 — pick the input path.** Look in `inputs/` (or wherever the user pointed):
- **A `.dxf`** → run the **DXF path** (exact). Skip the PDF passes entirely.
- **A `.pdf`** → run the **PDF path**. If several files, ask which; if none, ask for one.
- Confirm the rate library exists at `rates/rate-library.csv` (else tell the user to copy the template and fill it — never invent rates).

### DXF path (exact — 4 commands)
```
py scripts/takeoff.py dxf "inputs/<name>.dxf" --db outputs/<name>.db
py scripts/to_boq.py    --db outputs/<name>.db --out outputs/<name>_q.csv
py scripts/classify.py  outputs/<name>_q.csv outputs/<name>_c.csv
py scripts/price_boq.py --quantities outputs/<name>_c.csv --rates rates/rate-library.csv --out outputs/<name>-priced-boq.xlsx
```
Then report. No schedules/reconcile/measure — the geometry is the truth.

### PDF path (run every stage — don't skip the structural ones)
```
py scripts/takeoff.py build       "inputs/<name>.pdf" --db outputs/<name>.db   # + flags raster pages
py scripts/takeoff.py schedules   "inputs/<name>.pdf" --db outputs/<name>.db --auto
py scripts/takeoff.py dimensions  --db outputs/<name>.db                       # concrete m3 / steel from schedule dims
py scripts/takeoff.py annotations "inputs/<name>.pdf" --db outputs/<name>.db   # inline plan labels (columns/piles/French beams) — the ONLY thing that finds elements on schedule-less structural sets
py scripts/takeoff.py locate      --db outputs/<name>.db --include-numeric     # find scheduled marks on plans
py scripts/takeoff.py reconcile   --db outputs/<name>.db                       # promote MED->HIGH
py scripts/takeoff.py layers      "inputs/<name>.pdf" --db outputs/<name>.db   # OCG type-witness (no-op if flattened)
py scripts/takeoff.py measure     --db outputs/<name>.db                       # gross floor/slab area from printed dims
py scripts/takeoff.py quantities  --db outputs/<name>.db                       # headline qty + confidence per type
py scripts/takeoff.py rebar       "inputs/<name>.pdf" --db outputs/<name>.db   # reinforcement register + tonnage estimate
py scripts/to_boq.py    --db outputs/<name>.db --out outputs/<name>_q.csv
py scripts/classify.py  outputs/<name>_q.csv outputs/<name>_c.csv
py scripts/price_boq.py --quantities outputs/<name>_c.csv --rates rates/rate-library.csv --out outputs/<name>-priced-boq.xlsx
```
(Drop `--include-numeric` only if door marks are clearly not room-numbered. `schedules` is slow ~30s/page — that's expected, not a hang.)

**Report** the grand total, HIGH/MEDIUM/LOW/RATE_NOT_FOUND counts, per-trade quantities, any raster pages flagged, and the output path.

Do not invent quantities. Do not invent rates. Do not skip stages. **Do not eyeball the drawing for a count** — every number must be a query result from the DB. If the user asks how you know a number, back it with `cite`:

```
py scripts/takeoff.py cite --db outputs/<name>.db --type door
py scripts/takeoff.py cite --db outputs/<name>.db --mark 270A --crop
```

---

## Project overview

This project turns a **flat construction PDF** into a **priced Bill of Quantities**
in Excel. It reads the PDF's hidden vector text layer, builds a queryable
SQLite database once, cross-checks schedule counts against plan callouts, and
prices the reconciled quantities against a user-supplied rate library.

The output is a file, not a chat response.

Input is a flat PDF export — the same file a contractor would receive. No BIM
model needed. Works whether the drawings use `D-101`, `210.1`, `270A`, or any
other convention for door marks — see the `locate` step.

---

## The pipeline

```
  drawing.pdf                                                        rate library
      │                                                                    │
      ▼                                                                    ▼
   build ─► schedules ─► locate ─► reconcile ─► layers ─► quantities ─► to_boq ─► classify ─► price
      │        │           │          │           │           │           │           │          │
      ▼        ▼           ▼          ▼           ▼           ▼           ▼           ▼          ▼

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hamzaabduljabbar/autoconst-claude-2d-pdf-to-priced-boq](https://github.com/hamzaabduljabbar/autoconst-claude-2d-pdf-to-priced-boq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
