---
trigger: always_on
description: scripts repo for rhino 8 / grasshopper automation at raider boats.
---

# CLAUDE.md -- raider-scripts

scripts repo for rhino 8 / grasshopper automation at raider boats.
aluminum boat design, fabrication, and shop workflow tooling.

## what this repo is

standalone rhino python scripts and grasshopper C# script components
used in parametric aluminum boat hull design and CNC fabrication.
scripts are run via `_-RunPythonScript`, rhino aliases, or embedded
in grasshopper C# script components.

MIT licensed. no third-party grasshopper plugin dependencies.

## repo structure

```
/rhino-python/      standalone rhino python scripts (.py)
/grasshopper-cs/    grasshopper C# script components (.cs)
/CLAUDE.md          you are here
/LICENSE            MIT
```

## environment

- rhino 8, windows
- grasshopper: vanilla only (no food4rhino plugins in scripts)
- python: rhino's IronPython 2 (legacy) or CPython 3 (`#! python3` shebang)
  - prefer python 3 for new scripts unless IronPython is required
  - python 3 scripts MUST start with `#! python3`
- C#: .NET framework via grasshopper's C# script component
- units: inches, always
- tolerance: document tolerance, typically 0.001" for modeling, 1/16"-1/8" for fab

## supply chain security -- MANDATORY

these rules are non-negotiable. violations can introduce malicious code.

### before installing ANY dependency:

1. verify the package EXISTS and is what you think it is
   - check npmjs.com directly
   - confirm: correct name spelling, real maintainer, not a typosquat
2. check publish date: `npm view <pkg> time`
   - if the package OR the specific version is less than 7 days old, STOP
   - flag it to the user and wait for explicit approval
3. check download count
   - under 100 weekly downloads on npm = suspicious until proven otherwise
4. prefer well-known, high-download packages over obscure alternatives
5. NEVER invent or hallucinate a package name. if you're not 100% certain
   a package exists with that exact name, say so and let the user verify.

### npm hardening (enforce in all npm operations):

- always use `--ignore-scripts` on install unless user explicitly approves scripts
- always commit package-lock.json
- run `npm audit` after any dependency change
- pin exact versions in package.json (no ^ or ~ prefixes)
- prefer `dependencies` vs `devDependencies` correctly

### general dep philosophy:

- MINIMAL dependencies. every dep is attack surface.
- if something can be done in <50 lines of vanilla js, do that instead of adding a dep
- before suggesting a new dep, ask: does this NEED to be a package?
- vendor small utilities rather than importing them
- no meta-frameworks, no orms, no "convenience" layers

### if unsure about a package:

say "i'm not 100% sure this package exists / is safe -- please verify before installing"
-- this is ALWAYS preferable to confidently recommending something wrong.


## conventions

### python scripts

- use `rhinoscriptsyntax as rs` for UI/selection, `Rhino.Geometry` for math
- use `scriptcontext as sc` for doc access
- entry point pattern: define a main function, call it at bottom:
  ```python
  def my_command():
      # ...
      pass

  if __name__ == "__main__":
      my_command()
  ```
- error handling: fail loudly with `print()` messages. no silent failures.
- user interaction: `rs.GetObject`, `rs.ListBox`, `rs.MessageBox` for simple UI.
  `Rhino.Input.Custom.GetPoint` / `GetObject` for anything needing dynamic draw.
- for scripts with dynamic preview (like dim-3d), subclass `GetPoint` and
  override `OnDynamicDraw`

### grasshopper C# components

- file extension: `.cs`
- follows GH_ScriptInstance pattern with RunScript method
- type hints and output variable names must match component setup
- inputs/outputs documented in the summary comment block
- use `Rhino.Geometry.Unroller`, `Brep.CreateDevelopableLoft`, etc. directly
- NEVER use meshes as geometry output. surfaces/polysurfaces only.
  zero-thickness surfaces OK for intermediary ops if thickness is
  accounted for downstream.

### naming

- filenames: `kebab-case.py` or `kebab-case.cs`
- functions: `snake_case`
- classes: `PascalCase`
- constants: `UPPER_SNAKE`

### general

- geometry is always surfaces/polysurfaces, never meshes
- end state for parts: closed polysurfaces with material thickness
- data output format: CSV when exporting tabular data
- layer names follow numbered convention (01-14), see layer table below
- favor generalizable, reusable solutions over one-off hacks
- comprehensive fixes over incremental patches

## layer conventions

scripts that create/reference layers must use these names exactly:

```
01 - Default
02 - Reference        profiles, grids, scale figures
03 - Bake             GH output geometry
04 - Booleans         intersection geo, used booleans
05 - Clipping         layouts/drawings
06 - Annotations
07 - Jig              steel-framed build jig
08 - Static parts     3D model by assembly (Hull bottoms, Hardtop, etc.)
09 - Ink lines        bend marks per part
10 - Fab 3D           in-situ vs flat plate states
11 - 2D geo           flat patterns, sublayers: Inside cut / Outside cut / Mark
12 - Cutfiles
13 - Title block
14 - zArchive
```

sublayer separator is `::` (e.g., `11 - 2D geo::Inside cut`).
sublayer colors for 2D geo: magenta (inside cut), dark green (mark), blue (outside cut).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [travw/scripts](https://github.com/travw/scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
