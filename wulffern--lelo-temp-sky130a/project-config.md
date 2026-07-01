---
trigger: always_on
description: See work/Makefile for commands
---

# Temperature sensor


## Tools 

See work/Makefile for commands
- Schematic : xschem
- Layout : magic 
- layout versus schematic : netgen
- Layout engine : cicpy sch2mag

## Folder
- commands needs to be run in work/

## Layout Hints
- Prefer functional instance names in `.sch` files. Use names like `xpd_input[1:0]`, `xnd_bias_mirror[3:0]`, `xne_load[1:0]` instead of opaque `xca*`/`xpd*`.
- In `cicpy`, placement columns are controlled by the instance group name, which is the non-numeric prefix of the instance name. Choose prefixes intentionally in the schematic so default `place()` produces the desired stack order.
- Treat placement as routing-driven. Align devices that share high-fanout or sensitive nets in the same vertical column so drains/gates can be connected mostly with straight vertical metal.
- For matched devices, keep the members adjacent and on the same row/column style. In this project that usually means stacking bused instances vertically in one column.
- Do not add arbitrary X spacing between `JNW_ATR_SKY130A` transistor stacks. NMOS and PMOS stacks are intended to abut or overlap horizontally for compact layout.
- When choosing column pitch for transistor stacks, start from overlap-first placement and only open X spacing if routing proves it necessary.
- Prefer fixing placement by renaming schematic instances to the right groups before adding custom coordinate moves in `<CELL>.py`.
- For coupled analog branches, prefer `layout.makeCellGroup(...)` and `group.addStack(...)` in the cell Python file. Use the default `cicpy` placement for stack X order, then place groups with `abutTop(...)`, `abutBottom(...)`, `abutLeft(...)`, or `abutRight(...)` and a `space=` argument.
- Use `stack.addTaps()` to place one `CTAPBOT` and one `CTAPTOP` for a whole physical stack. If tap clearance matters for later `abut*()` placement, add the taps before abutting groups so the stack bounding box includes them. Use `group.fillDummyTransistors()` when sibling stacks should be equalized in height with physical-only dummy devices.
- Before drawing custom metal to a device terminal, check whether the device already exposes legal access on that layer. In `cicpy`, prefer `instance.getTerminalAccess("D"|"G"|"S"|"B", target_layer="M1")` over guessing where to place M1.
- Treat terminal access as physical geometry, not just schematic naming. For transistor dummies in particular, reuse the device's existing M1 access rectangles instead of stamping generic bars that may violate local spacing.
- Use `CTAPBOT` and `CTAPTOP` physical cells as end-caps around vertical transistor stacks when needed. They are physical-only, so warnings about missing SPICE subckts are expected when generating layout.
- Put startup or enable devices under the branch they assist if that shortens a critical branch net. Put output pull-up/pull-down devices in a column that keeps the `VO` route short and direct.
- In custom `cicpy` Python placers, disable the default `AVDD/AVSS` paint step with `layout.noPowerRoute = True` when the cell instead uses `VDD_1V8`/`VSS`.
- When debugging routing, prefer the fast route-short report from `cicpy sch2mag`. It now runs automatically and is intended to point back to the Python route statement that caused the short.
- Use full connectivity checking only when needed. `cicpy sch2mag --check-connectivity <LIB> <CELL>` is slower and better suited for broader open/split-net analysis than day-to-day route debugging.
- `sch2mag` writes a top-cell `.cic` plus generated cut cells. Child library cells are not embedded in that file. When rendering or inspecting such a cell outside Magic, include dependent library `.cic` files explicitly.
- `cicpy` commands that read `.cic` now support `--I <lib.cic>` to merge additional library files before processing. Use that for `svg`, `transpile`, `jcell`, `minecraft`, and similar design readers when the top cell references external primitive libraries.
- For `LELOTEMP_CMP` SVG/debug outside Magic, include at least `JNW_ATR_SKY130A.cic` and `JNW_TR_SKY130A.cic`, and use `tech/cic/sky130A.tech` rather than `sky130.tech` because the rendered libraries use layers like `POR`.
- Keep route-debug instrumentation concise. The useful output is: shorted nets, one offending route command, and one `file:line` callsite. Avoid flooding the report with internal dummy-route details.
- Internal dummy routes should be treated as implementation detail. If a report is dominated by `xfill_*_dummy_*` nets, suppress them and focus on user-created routes such as `addConnectivityRoute(...)`.
- For quick route debug, checking generated route geometry against exposed terminal/port access is usually enough. Full recursive geometry expansion is only needed for deeper connectivity analysis.
- Run layout generation from `work/` with `cicpy sch2mag <LIB> <CELL>`, then inspect the generated `.mag` to confirm stack order and tap placement.
- A short that shorts many nets is worse than a few opens
- M1 = locali, M2 = metal1, read sky130A.tech for details
- Keep the Magic vs `cicpy` layer naming straight when inspecting `.mag`: Magic `metal3` = `cicpy M4`, Magic `metal4` = `cicpy M5`. For `JNWTR_CAPX1`, terminal `A` is on Magic `metal3` and terminal `B` is on Magic `metal4`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wulffern/lelo_temp_sky130a](https://github.com/wulffern/lelo_temp_sky130a) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
