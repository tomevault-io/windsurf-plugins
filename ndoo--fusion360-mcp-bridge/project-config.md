---
trigger: always_on
description: This file is read automatically by Claude at the start of every session.
---

# Fusion 360 MCP — Scripting Knowledge

This file is read automatically by Claude at the start of every session.
It encodes hard-won knowledge about the Fusion 360 Python API so that
`fusion_execute` scripts work first time rather than through trial and error.

---

## Architecture

```
Claude  ←→  server.py (MCP, stdio)  ←→  HTTP :7654  ←→  FusionMCPBridge add-in  ←→  Fusion API
```

Two tools:
- **`fusion_execute(script)`** — run arbitrary Python inside Fusion's process
- **`fusion_screenshot(...)`** — capture the active viewport as a base64 PNG

All geometry work goes through `fusion_execute`. Write scripts that `print()` their results back.

---

## Units

Fusion stores all geometry in **centimetres (cm)**. When calling API methods that accept a `ValueInput`, pass strings like `"3cm"`, `"10mm"`, `"45 deg"` — Fusion converts them. Raw floats are always in cm.

---

## Script structure

```python
import adsk.core, adsk.fusion, math

def run(_context):
    comp = design.activeComponent
    # ... work here ...
    print("done")
```

The context provides `adsk`, `app`, `ui`, and `design` (may be `None` if no design is open). Defining `run(_context)` is optional but conventional. Always `print()` results — return values are ignored.

---

## Design mode

Check before scripting anything parametric:

```python
import adsk.fusion
def run(_context):
    mode = design.designType
    if mode == adsk.fusion.DesignTypes.ParametricDesignType:
        print("parametric")
    else:
        print("direct")
```

- **Parametric mode**: features are recorded in the timeline. `TemporaryBRepManager` bodies need a `BaseFeature` wrapper (see below).
- **Direct mode**: operations happen immediately, no timeline. Some parametric APIs are unavailable.

---

## Revolve / profile rules

**The single most common error: profile crossing the revolve axis.**

Fusion's modelling kernel rejects any revolve where the sketch profile has geometry on both sides of the axis — even if the crossing point is just a line endpoint at the origin.

### Rules
1. The profile must lie **entirely on one side** of the revolve axis (x ≥ 0 or x ≤ 0).
2. Profile edges **touching** the axis are fine (the poles of a sphere land on the axis).
3. A closing line that runs **along** the axis is fine.
4. A line that **passes through** the axis (e.g. from `(r,0,0)` to `(-r,0,0)`) is not fine.

### Correct sphere pattern (use this every time)

Revolve on the **XY plane around the Y axis**. Use `addByThreePoints` — it's unambiguous about arc direction.

```python
import adsk.core, adsk.fusion, math

def run(_context):
    comp = design.activeComponent
    r = 3.0  # 3 cm radius → 6 cm diameter

    sk = comp.sketches.add(comp.xYConstructionPlane)

    # Arc: top pole → equator → bottom pole (all x >= 0)
    sk.sketchCurves.sketchArcs.addByThreePoints(
        adsk.core.Point3D.create(0,  r, 0),
        adsk.core.Point3D.create(r,  0, 0),
        adsk.core.Point3D.create(0, -r, 0))

    # Closing line along Y axis (ON the axis — allowed)
    sk.sketchCurves.sketchLines.addByTwoPoints(
        adsk.core.Point3D.create(0, -r, 0),
        adsk.core.Point3D.create(0,  r, 0))

    # Sanity check
    if sk.profiles.count == 0:
        print("Error: no profile formed")
        return

    rev = comp.features.revolveFeatures.createInput(
        sk.profiles.item(0),
        comp.yConstructionAxis,
        adsk.fusion.FeatureOperations.NewBodyFeatureOperation)
    rev.setAngleExtent(False, adsk.core.ValueInput.createByString('360 deg'))
    feat = comp.features.revolveFeatures.add(rev)
    print(f"Sphere created: {feat.name}")
```

### Why not `addByCenterStartSweep`?

`addByCenterStartSweep(center, start, sweep_angle)` sweeps in the sketch plane's "positive" rotation direction. The direction is easy to get wrong, and if the start point is on the axis the profile may fail silently. `addByThreePoints` is always unambiguous.

---

## Profile formation checklist

If `sk.profiles.count == 0` after adding sketch curves:

1. **Open contour** — every curve endpoint must connect to exactly one other curve endpoint. Check for gaps.
2. **Duplicate edge** — adding the same line twice creates a zero-area region. Remove duplicates.
3. **Endpoints on revolve axis** — can sometimes prevent profile closure. Shift the arc to avoid placing both endpoints on the axis, or switch to `addByThreePoints`.
4. **Wrong sketch plane** — verify `comp.xYConstructionPlane` vs `xZConstructionPlane` vs `yZConstructionPlane`.

Always add an explicit check before using `profiles.item(0)`:

```python
if sk.profiles.count == 0:
    print(f"No profile — curves added: {sk.sketchCurves.count}")
    return
```

---

## TemporaryBRepManager pattern (complex shapes)

Use this for polyhedra, convex hulls, and anything that can't be built from a single sketch profile. It constructs BRep topology directly without needing Fusion's feature history.

```python
import adsk.core, adsk.fusion

def run(_context):
    tbm = adsk.fusion.TemporaryBRepManager.get()
    # ... build bodies with tbm ...

    if design.designType == adsk.fusion.DesignTypes.ParametricDesignType:
        # Parametric mode: must wrap in a BaseFeature
        base_feat = design.activeComponent.features.baseFeatures.add()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndoo/fusion360-mcp-bridge](https://github.com/ndoo/fusion360-mcp-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
