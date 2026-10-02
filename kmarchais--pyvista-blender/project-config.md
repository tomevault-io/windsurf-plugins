---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Cursor, Aider, ...) working on
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Aider, ...) working on
`pyvista-blender`. This file is the project's **single source of truth** for
conventions; `CLAUDE.md` points here.

## What this project is

A Python library that translates a live `pyvista.Plotter` scene into a
Blender (`bpy`) scene and renders it through Cycles or Eevee Next. Users
build scenes with PyVista's API; rendering becomes a backend choice. No
file round-trip between the two — translation is in-memory with an
identity-keyed cache.

The architectural surface is documented in [`docs/architecture.md`](./docs/architecture.md):

- Translation pipeline (PyVista actor → bpy mesh / material / light).
- Identity-keyed Level-1 cache (mesh data-blocks and materials reused
  across renders on the same plotter).
- Volumetric dispatch (closed-cube + Cycles Volume Principled atlas).
- Interactive viewport architecture (`pl.blender.show()` overlay +
  Trame backend).
- Animation export channels (camera, deformation, scalars, lights,
  glyphs).

## API shape

The bridge attaches to PyVista's `BasePlotter` via
`@pv.register_plotter_component("blender")` plus an entry point in
`pyproject.toml`:

```toml
[project.entry-points."pyvista.plotter_components"]
blender = "pyvista_blender._component"
```

Users **never `import pyvista_blender`** — installing the package makes
`pl.blender.render(...)` work via PyVista 0.48's plotter-component
auto-discovery. The accessor acceptance test
(`tests/test_accessor.py::test_blender_accessor_resolves_without_explicit_import`)
guards that contract.

Three-tier config resolution: **per-call kwarg → component attribute →
module default**. `pl.blender.resolve_config(attr, call_value)` is the public
way to inspect what would resolve.

## Version policy

| Python             | bpy wheel  | Blender                  |
| ------------------ | ---------- | ------------------------ |
| 3.11               | `>=4.5,<5` | 4.5 LTS                  |
| 3.13               | `>=5.0,<6` | 5.0 / 5.1+               |
| 3.10 / 3.12 / 3.14 | none       | no matching wheel exists |

`requires-python = ">=3.11,!=3.12.*,<3.14"`. The split is dispatched via
PEP-508 markers in `[project] dependencies`. `fake-bpy-module` mirrors the
same split in the `dev` group (5.0 stubs cover the 5.x line since 5.1
stubs aren't on PyPI yet).

## License: GPL-3.0-or-later

Forced by `bpy`'s GPL license. Every Python file under `src/` and
`tests/` starts with an SPDX header:

```python
# SPDX-FileCopyrightText: 2026 Kevin Marchais
# SPDX-License-Identifier: GPL-3.0-or-later
```

Ruff recognises SPDX headers via `[tool.ruff.lint.flake8-copyright]
notice-rgx`. Do not switch to "Copyright (C) ..." — SPDX is the modern
convention.

## Linting philosophy: fix code, not ignore rules

`select = ["ALL"]` with `preview = true`. The top-level `ignore` list
contains exactly three entries, each a **structural conflict in ruff
itself** that cannot be satisfied:

- `D203` ↔ `D211` (mutually exclusive class-docstring blank-line rules)
- `D213` ↔ `D212` (mutually exclusive multi-line docstring summary rules)
- `COM812` (ruff's formatter docs require disabling)

**No project-specific ignores. No per-file ignores.** When a rule fires:

1. Try to fix the code first (rename, restructure, add docstring section, change type).
2. If the rule literally cannot be satisfied (a framework contract or
   external convention), use an **inline `# noqa: RULE`** with a comment
   explaining why.

Load-bearing inline `noqa`s fall into a small number of families:

- **Framework contracts** (`PLW3201`, `ANN401` on
  `pyvista_blender.jupyter.handler`): PyVista's component registry
  requires the exact dunder name `__plotter_close__`, and PyVista's
  jupyter-backend protocol calls the handler with arbitrary kwargs
  (the user's `pl.show(**user_kwargs)` flows through pyvista's own
  `window_size` / `return_img` / `cpos` / ... wrapper). The handler's
  `**kwargs: Any` is the canonical way to accept that open-ended
  contract — TypedDict / Unpack would lock us to whatever subset of
  pyvista's evolving signature we chose to enumerate.
- **Lazy bpy imports** (`PLC0415`): each public entry point on
  `BlenderComponent` (`render`, `animate`, `show`, `export_blend`,
  `export_animation_blend`) and the Jupyter / web handlers
  lazy-import the bpy-touching submodule so PyVista's entry-point
  discovery (which imports `_component` the moment a user touches
  `pl.blender`) doesn't pay bpy's ~200 MB / ~3 s startup cost
  upfront.
- **Deliberately flat public API** (`PLR0913`): `show()` carries 15
  user-facing kwargs spanning the desktop + web viewport + sample
  tiers + render config + HUD toggles. That's the bridge's surface
  area for the interactive viewport, matching PyVista's house style
  (cf. `pl.add_mesh` with ~30 kwargs). The kwargs _are_ the API.
- **Typed-bypass for under-typed stubs** (`B010` for
  `setattr(node, "operation", ...)`): fake-bpy-module's stubs miss
  dynamic `operation` / `domain` / ... attributes on Math / Mix /
  Store-Attribute nodes that exist at runtime. `setattr` is the
  smallest escape.
- **Genuine physical constants in test asserts** (`PLR2004`): a
  handful of magic numbers (`< 2` for "at least two frames",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kmarchais/pyvista-blender](https://github.com/kmarchais/pyvista-blender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
