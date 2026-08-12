---
trigger: always_on
description: A Lean 4 diagramming library with envelope-based spatial composition,
---

# Illuminate

A Lean 4 diagramming library with envelope-based spatial composition,
SVG rendering, and infoview preview via `#diagram`.

## General instructions

Always use the definition of `pi` in `Basic.lean`, importing if
necessary. Never define your own or use other workarounds.

## Prerequisites

- [elan](https://github.com/leanprover/elan) (manages Lean toolchains
  automatically via `lean-toolchain`)
- [uv](https://docs.astral.sh/uv/) (for Playwright structural tests
  and visual regression runner)
- [Docker](https://docs.docker.com/get-docker/) (for visual regression
  tests; on macOS, [colima](https://github.com/abiosoft/colima) works)

## Building

```sh
lake build --wfail
```

This compiles the library (`src/`) and all dependencies. The build
must complete with **zero warnings** — `linter.missingDocs` is enabled
in `lakefile.lean`, so all public declarations require docstrings. The
`--wfail` flag ensures warnings are treated as errors.

## Testing

Always run tests after every change before reporting success.

### Lean unit tests

```sh
lake test --wfail
```

This builds and runs the test executable. It also writes SVG files
(`smiley.svg`, `commdiag.svg`, `roundedrects.svg`) used by the visual
tests.

### Structural and visual regression tests

```sh
uv run test_playwright.py
```

Runs two kinds of tests:

- **Structural tests** use Playwright (headless Chromium) to inspect
  SVG DOM elements (element counts, text content, bounding boxes).
- **Visual regression tests** render SVGs to PNG via Inkscape inside a
  Docker container (`visual_tests/Dockerfile`) with bundled DejaVu
  fonts, then compare against committed baselines. The Docker image is
  built automatically on first run.

To update expected baselines after intentional visual changes:

```sh
UPDATE_BASELINES=1 uv run test_playwright.py
```

Visual test files live in `visual_tests/`:

- `*.expected.png` — committed baselines (the ground truth)
- `*.actual.png` — generated each run, gitignored
- `Dockerfile` — Inkscape + DejaVu fonts container for rendering
- `fonts/` — bundled DejaVu Sans and DejaVu Sans Mono (v2.37)
- `fonts.conf` — fontconfig rules mapping generic families to DejaVu
- `render.sh` — stdin→stdout SVG-to-PNG wrapper for Inkscape

**Important**: Never run `UPDATE_BASELINES=1` without explicit user
approval. If visual tests fail, investigate and fix the underlying
issue first. Only update baselines when the visual change is
intentional and the user has confirmed it.

### Import minimization

After tests pass, run `lake shake`. If it doesn't succeed, ask the
user whether to run `lake shake --fix`.

### JavaScript type checking

The animation player and widget JavaScript in `player_js/` uses JSDoc
type annotations checked by TypeScript. React types are vendored in
`vendored_js/` (excluded from language stats via `.gitattributes`).

To run the type checker, use this command:

```sh
npx tsc --noEmit -p player_js/jsconfig.json
```

### Running both

```sh
lake test --wfail && uv run test_playwright.py
```

### Formatting

Before finishing work, run Prettier to format non-Lean files
(Markdown, Python, JSON, YAML):

```sh
npx prettier --write .
```

CI has a Prettier bot that auto-commits formatting fixes, but pushes
from `github-actions[bot]` do not re-trigger CI. Running Prettier
locally avoids this problem.

### README consistency

When adding, removing, or renaming public API, check that `README.md`
still accurately describes the library. Verify that referenced
function and type names exist, code examples use correct parameter
names, the module overview lists the right types, and new user-facing
features have a section.

The README is for humans, not machines. Keep it informal and
approachable — give readers enough to understand what something does
and when to reach for it, not an exhaustive catalogue of every
parameter and edge case. Prefer short descriptions with a code example
over long prose. If a section is getting dense, that is a sign to cut
detail, not add more.

## Project structure

```
src/Illuminate/          Library source
  Geometry/
    Basic.lean           Foundational constants (pi)
    Types.lean           Core geometry types (Vec2, Point, Matrix, Envelope, PathData)
    Vec2.lean            2D vector operations (directions and offsets)
    Point.lean           2D point operations (positions)
    Matrix.lean          3x3 affine transform matrix operations
    Envelope.lean        Envelope operations (direction -> extent)
    Trace.lean           Trace (ray-shape intersection for boundary detection)
    PathData.lean        Path drawing commands (line, rect, circle, roundedRect)
  Style/                 Color, Fill, Stroke, TextStyle, FontStyle, StyledText
  Diagram/
    Types.lean           Core Diagram type, Backend class, CorePrimitive, LineEnd
    Basic.lean           Smart constructors (circle, rect, text, styledText, ...)
    Placement.lean       Spatial composition (beside, hcat, vcat, grid, pad, frame)
    Arrow.lean           Curved arrow routing (LineEnd, Arrowhead, connect)
    CurlyBrace.lean      Curly brace annotation (curlyBrace, braceBelow, ...)
    TreeLayout.lean      Automatic tree layout (treeLayout, proofTree)
    Paper.lean           Piece-of-paper diagram element

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leanprover/illuminate](https://github.com/leanprover/illuminate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
