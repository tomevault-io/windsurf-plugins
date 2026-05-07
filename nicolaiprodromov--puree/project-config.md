---
trigger: always_on
description: > These instructions are loaded into every Copilot conversation in this workspace.
---

# Puree — Copilot Instructions

> These instructions are loaded into every Copilot conversation in this workspace.
> They provide the essential context for any AI working on Puree.

## What Is Puree

Puree is a **GPU-accelerated UI framework for Blender addons**. It lets developers build rich custom interfaces using YAML (structure), SCSS (styling), and Python (interactivity). It renders via ModernGL compute shaders inside Blender panels. Layout uses Taffy/Stretchable (a Rust flexbox/grid engine compiled to native binaries via PyO3).

**It is NOT a web framework.** Many CSS features don't exist. See "What Puree Doesn't Support" below.

## Repository Structure

```
puree/
├── puree/                    # ← THE ENGINE (Python + Rust + GLSL)
│   ├── parser.py             # YAML → Container tree
│   ├── parser_op.py          # Parser state sync operator
│   ├── compiler.py           # Executes user script.py
│   ├── render.py             # GPU rendering pipeline (ModernGL)
│   ├── transition_manager.py # CSS transition animations
│   ├── hit_op.py             # Hit detection modal operator
│   ├── input_router.py       # Event consumption routing
│   ├── hot_reload.py         # File watcher + live reload
│   ├── hot_reload_ops.py     # Hot reload Blender operators
│   ├── reload_server.py      # Built-in TCP server (127.0.0.1:19746) — reload, ping, logs
│   ├── cli.py                # CLI tool: puree init/build/install/link/unlink/reload
│   ├── native_bindings.py    # Rust FFI: HitDetector, SCSSCompiler, ColorProcessor
│   ├── text_op.py            # Text rendering operator
│   ├── text_input_op.py      # Text input handling operator
│   ├── img_op.py             # Image loading operator
│   ├── mouse_op.py           # Mouse state tracking operator
│   ├── scroll_op.py          # Scroll state tracking operator
│   ├── panel.py              # Blender debug panel
│   ├── components/           # Container class, Style class, defaults
│   ├── puree_core/           # Rust source (compiled → native_binaries/)
│   └── shaders/              # GLSL compute/vertex/fragment shaders
├── dist/                     # ← BUILD SCRIPTS & TOOLS
│   ├── dev_reload.py         # Triggers reload in running Blender
│   ├── build.sh / build.bat  # Platform build scripts
│   └── release.py            # Release automation
├── static/                   # ← BUILT-IN UI (example/default panel)
│   ├── index.yaml
│   ├── style.scss
│   ├── script.py
│   └── components/
├── examples/                 # ← EXAMPLE PANELS
├── docs/                     # ← DOCUMENTATION
│   ├── PUREE_SPEC.md         # Full framework specification
│   ├── PUREE_VS_CSS.md       # CSS compatibility reference
│   ├── API.md                # Python API reference
│   └── COMPONENTS.md         # Component system docs
├── assets/                   # Images (PNG, SVG)
├── fonts/                    # Font files (.ttf, .otf) — NeueMontreal family
├── __init__.py               # Blender addon entry point + reload server lifecycle
└── blender_manifest.toml     # Blender extension manifest
```

## Two Kinds of Work

### 1. UI Work (YAML/SCSS/Python panels)
**Agent**: `puree-coder`

Building or modifying end-user interfaces. Files in `static/`, `examples/`, or addon directories.

### 2. Engine Work (Python/Rust/GLSL internals)
**Agent**: `puree-maintainer`

Fixing bugs or extending the framework itself. Files in `puree/`, `puree/puree_core/`, `puree/shaders/`.

## Critical Rules — Always Remember

### YAML
- Node names: **underscores only** (`my_button` ✓ `my-button` ✗) — parser breaks on hyphens
- Component refs: `data: '[component_name]'` — square brackets required
- Parameters: `"{{param_name, 'default_value'}}"` — both quote types, comma required
- Font/image names omit extensions

### SCSS
- **Only 3 transition-animatable properties**: `background-color`, `border-color`, `opacity` (`color` changes instantly on hover)
- Layout properties in `:hover`/`:active` are **silently ignored** (no width/height/padding/margin changes on hover)
- Units: `px`, `%`, `rem`, `em`, `vw`, `vh`, `vmin`, `vmax`, and `calc()` (no `fr`, `clamp()`, `min()`, `max()`)
- Display: only `flex`, `grid`, `block`, `none` (no inline variants)
- Gradients: only `linear-gradient()` (no radial, conic)
- Selectors: no `::before`/`::after`, attribute selectors (but `:nth-child`, `:not()`, `:first-child`, `:last-child`, sibling combinators `+`/`~` ARE supported)
- `--` extension properties need `#{$var}` interpolation for SCSS variables
- Inherited properties: `color`, `font-size`, `text-align`, `font-family`, `font-weight`, `font-style`, `pointer-events`, `visibility`, `text-transform`, `line-height`, `letter-spacing`, `white-space`
- Font selection uses YAML `font:` attribute, not CSS `font-family`

### Python (script.py)
- Entry: `def main(self, app):` — must `return app`
- After ANY property change: call `mark_dirty()` — GPU won't update without it
- Runtime display values are UPPERCASE: `'FLEX'`, `'NONE'` (CSS uses lowercase)
- Event callbacks: `fn(container)` — all take one argument
- Component children are namespaced: `instance_child_name`
- Blender is NOT thread-safe — use `threading.Thread` for async, `bpy.app.timers` for deferred bpy calls

### Engine (for maintainers)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicolaiprodromov/puree](https://github.com/nicolaiprodromov/puree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
