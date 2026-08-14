---
trigger: always_on
description: Guide for agents (and humans writing them) driving Vectorworks 2026 through this
---

# AGENTS.md — integrating with vwx-mcp

Guide for agents (and humans writing them) driving Vectorworks 2026 through this
server. Read this before generating `vs.*` code or wrapping new tools — the
VW2026 API has sharp edges that produce **silent failures** (null UUIDs, no-op
draws) rather than errors.

## Connect

MCP endpoint: `http://127.0.0.1:8082/mcp` (streamable-http). The VW-side bridge
must be running first (see README). `ping` confirms the full chain is live.

## Three access layers — pick the narrowest that works

1. **Explicit tools** (248). Typed, documented, safe. Prefer these.
2. **`vwx(command, params)`** — generic dispatcher to any verb in `commands.py`.
   Call `list_commands` to discover. Use when no explicit wrapper exists but a
   `commands.py` verb does.
3. **`execute_script(code)`** — arbitrary `vs.*` Python on the VW main thread.
   Last resort / one-offs. Contract:
   - `print(...)` → captured into the `output` field.
   - assign **`__result__`** → returned in `result` (str/int/float/list/dict/bool only).
   - There is **no** `return` and no `return_value` — assigning anything else is ignored.

## Object addressing — UUIDs only

Objects are identified by **UUID strings**: `vs.GetObjectUuid(handle)` /
`vs.GetObjectByUuid(uuid)`. The old `InternalIndex` APIs are **gone on VW2026** —
do not use them. Tool results return `object_id` = the UUID.

## Toolset presets

If the client is loading too many tools, set `VWX_TOOLSET` (env, in the launcher):
`full` | `gis` | `modeling` | `baumkataster` | `minimal`. Mapping lives in
`mcp-server/tool_tags.py`. Filtering uses the fastmcp Visibility API
(`mcp.enable(tags=…, only=True)`).

## VW2026 API gotchas (silent failures — memorize these)

| Symptom | Cause | Do this instead |
|---|---|---|
| `draw_circle` returns `object_id: null`, nothing drawn | `vs.ArcByCenter((cx,cy), r, 0, 360)` is broken on VW2026 | `vs.Oval(cx-r, cy+r, cx+r, cy-r)` (bbox: left, top, right, bottom), then `vs.LNewObj()` |
| Arc has wrong angular extent | `vs.Arc`'s 6th arg is the **sweep** (included) angle, **not** the end angle | `vs.Arc(l, t, r, b, start, sweep)` — pass sweep directly (verified: `GetArc` of `Arc(…,30,90)` → `(30, 90)`) |
| `vs.GetVWVersion` → AttributeError | does not exist on VW2026 | `vs.GetVersion()` → `(major, minor, maint, build)` |
| Can't enumerate class names | `GetClassName` / `GetClName` / `ClassList` removed | walk objects with `ForEachObject` and collect `vs.GetClass(h)` |
| `LNewObj()` returns the wrong/no object after a geometry op | some ops convert/replace (e.g. `DTM6_SendToSurface` converts 2D→3D and does **not** surface via `LNewObj`) | name the object first (`SetName`) + look it up, or use `PrevObj(LNewObj())` |
| `ForEachObject` callback corrupts iteration | mutating (delete/create/reclass/restack) during traversal invalidates `NextObj` | collect handles first, mutate after |
| `SetFillFore`/`SetPenFore` silently no-op | passing positional r,g,b | pass a single `RGBCOLOR` tuple |
| Old marker/pen/dash/wall-height calls misbehave | pre-2019 forms are obsolete | use the `…N` variants (`SetLSN`, `InsertNewComponentN`, …) |
| Criteria string matches nothing | quoting | single-quote record names, mind the parens: `"((R in ['Part Info']))"` |

## VW2026 API renames (function does not exist under the old name)

These raise `module 'vs' has no attribute …` (or wrong-arg engine errors). All
found + fixed via the full command sweep; the correct forms are in `commands.py`.

| Old / wrong | VW2026 correct | Notes |
|---|---|---|
| `vs.CreateExtrude(h, ht)` | `vs.HExtrude(h, zBottom, zTop)` | z baked into the extrude, no separate move |
| `vs.LinDimN(...)` | `vs.LinearDim(start, end, offset, dimType, arrow, textFlag, textOffset)` | 7 args |
| `vs.AngularDim(center, p1, p2, off, arr, txt)` | `vs.AngularDim(startPt, endPt, arcCenter, textOffset, arrow, textFlag, posAngle)` | 7 args; **center is 3rd** |
| `vs.SetClassVisibility(name, n)` | `vs.ShowClass(name)` / `vs.HideClass(name)` | by name |
| `vs.IFC_SetPSetAttribute(...)` | `vs.IFC_SetPsetProp(h, pset, prop, value)` | value is a STRING |
| `vs.ZoomToSel()` | `vs.DoMenuTextByName('Fit To Objects', 0)` | selection-aware fit |
| `vs.SetWSCellValue(ws, r, c, v)` | `vs.SetWSCellFormula(ws, r, c, r, c, v)` | 5 cell coords + value |
| `vs.SaveDocument()` | `vs.DoMenuTextByName('Save', 0)` | keeps path/format |
| `vs.SymbolCreate(...)` | `vs.BeginSym(name)` … create geometry … `vs.EndSym()` | captures objects made between the calls |
| `vs.HMirror(h, p1, p2)` | `vs.MirrorN(h, dup, p1, p2, preserveMatrix)` | dup=False transforms in place |
| `vs.SetLName(h, name)` | `vs.SetName(h, name)` | generic — renames any named object incl. layers |
| `vs.GetNumberOfComponents(h)` | returns **`(ok, count)`** tuple | unpack `[1]`, not an int |
| `create_wall` via `vs.SetPrefReal(85)`+`vs.SetPref(68)` | `vs.SetWallWidth` → `vs.Wall` → `vs.SetWallThickness` + `vs.SetWallHeights` | the pref-poking form **hard-crashes VW** |

**Verbs that always open a modal VW dialog** (the `vs` API has no headless path):
`export_pdf`, `export_image`, `export_dxf`, `export_shp`, `export_ifc`,
`save_document_as`, `import_dwg`, `import_image`. `export_pdf` in particular

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vicquick/vwx-mcp](https://github.com/vicquick/vwx-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
