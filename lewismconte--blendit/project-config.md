---
trigger: always_on
description: This is the governing brief for the repo: a project brief, not a one-shot spec.
---

# Build Brief — `Blendit`

This is the governing brief for the repo: a project brief, not a one-shot spec.
Work through it in phases, **confirm the contract before building on it**, and
produce a runnable demo at the end of each phase.

> The long code listings for the contract live in their real files now — see
> [`bir_contract/`](bir_contract/) and [`docs/contract.md`](docs/contract.md). This file
> keeps the narrative, the constraints, and the working agreement.

---

## 1. What we're building

`Blendit` — a tool that moves a Revit model into Blender for
rendering, fast, and applies a curated, good-out-of-the-box rendering pipeline.
The name is a play on "blend it" (Blender + edit); the goal is an experience as
seamless as **Rhino.Inside.Revit** ("one click, it's in Blender, it looks good").

**v1 target:** a one-click Revit-to-Blender rendering tool — press a button in Revit,
get a high-quality render of the current 3D view in Blender, with photoreal
defaults and several render modes.

**But build for more.** v1 is the smallest useful thing; the longer-term aim is a
general Revit↔Blender bridge (live link, asset injection, animation, analysis
viz, and eventually data flowing *back* into Revit). The transport and data
contract must be clean, documented, stable interfaces — everything else is built
on top of them and is expected to change.

---

## 2. Read this before you architect anything

A literal "Blender running inside the Revit process" (the way Rhino.Inside.Revit
loads RhinoCommon into Revit) is **not achievable the same way**:

- Rhino.Inside works because Rhino's core is a **.NET** assembly loadable into a
  .NET host (Revit).
- Blender is **not** .NET. Its `bpy` module is a heavy native CPython module, and
  pyRevit runs on **IronPython**. `bpy` cannot be loaded into Revit's process.

**Therefore the architecture is a bridge, not an in-process embed.** Do **not**
`DllImport` Blender into .NET or `import bpy` inside a pyRevit script. The two
sides are separate processes that talk only through the transport layer (§6).
Keep the door open for tighter coupling later (a future bpy-in-subprocess
"inside" mode), but v1 is a bridge.

> If you ever feel the design requires loading Blender into Revit, **stop and
> flag it** — the design is wrong, not the constraint.

---

## 3. Architecture overview

```
┌─────────────────────────┐         ┌──────────────────────────────┐
│  REVIT  (IronPython)     │         │  BLENDER  (its own process)  │
│  pyRevit extension       │         │  add-on  +  headless bpy     │
│  Extract:                │  bundle │  Import bundle               │
│   • geometry (tessellate)│ ───────▶│  Map materials → Principled  │
│   • materials/appearance │ (glTF + │  Build world (sun + sky/HDRI)│
│   • active-view camera   │  Scene  │  Set camera + view transform │
│   • sun position/time    │  Spec   │  Apply render-mode preset    │
│   • units / base point   │  JSON)  │  Cycles / EEVEE → render PNG │
└─────────────────────────┘         └──────────────────────────────┘
        │                                          ▲
        └────────────  TRANSPORT (pluggable)  ─────┘
        v1: file-based payload + sidecar scene_spec.json in a watched folder
        future: USD · Speckle · websocket live-link
```

The **transport** is the seam. v1 is the simplest thing that works (files on
disk). Everything else is swappable behind the interface.

---

## 4. Tech stack & pinned constraints

**Revit side**
- pyRevit extension — the repo root IS the extension (`Blendit.tab/` + `lib/` at
  the root, so it's pyRevit-catalog-installable). See [`Blendit.tab/`](Blendit.tab/).
- Target **IronPython 2.7** compatibility unless we explicitly opt into the
  CPython engine. Keep Revit-side deps minimal and pure-Python.
- Revit 2025+ requires .NET 8; support a reasonable range of Revit versions.
- Revit API stubs are for IDE hints **only**. Guard RevitAPI imports so the
  package is testable headless.

**Blender side**
- Target **Blender 4.2 LTS and newer** (EEVEE-Next floor). Test on 4.2 LTS and
  the current LTS.
- Package the interactive add-on with the **Blender 4.2+ Extensions** system.
- Headless `bpy`: **one Python version per Blender release** — pin it, document
  it in [README.md](README.md). `bpy`-as-module loads the factory startup scene
  and cannot be `importlib.reload`-ed → fresh process per render, or
  `read_factory_settings(use_empty=True)`.

**Shared**
- Define the data contract once, validatable on both sides: dataclasses
  ([`bir_contract/scene_spec.py`](bir_contract/scene_spec.py)) + JSON schema
  ([`bir_contract/scene_spec.schema.json`](bir_contract/scene_spec.schema.json)).

---

## 5. Repository layout

See [README.md](README.md#repository-layout) for the table. Full intended tree:

The repo root IS the pyRevit extension (`.tab` at root, catalog-installable); the
Blender pipeline + contract ship inside it.

```
blendit/                         # = the extension root (clone as Blendit.extension)
├─ extension.json                # pyRevit catalog metadata
├─ CLAUDE.md  README.md  LICENSE
├─ Blendit.tab/Render.panel/LoadModel.pushbutton/  # ribbon; + OpenModel / RenderLoadedModel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lewismconte/blendit](https://github.com/lewismconte/blendit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
