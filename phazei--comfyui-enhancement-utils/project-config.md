---
trigger: always_on
description: Guidelines for working on this project. Read this before making changes.
---

# CLAUDE.md

Guidelines for working on this project. Read this before making changes.

## Project Goals

This is a curated collection of ComfyUI enhancement utilities. The philosophy:

- **Quality over quantity** -- take proven features from aging community packages, combine and improve them
- **Well-organized, well-documented code** -- every module, class, and function gets a docstring or JSDoc comment
- **Modern ComfyUI patterns** -- V3 schema for nodes, current best practices for JS extensions
- **Robust error handling** -- graceful degradation when optional dependencies are missing, no crashes from malformed data

This is NOT a kitchen-sink package. Features are included because they fill real gaps or meaningfully improve on existing implementations.

## Architecture

### Python (Nodes + Monitor Backend)

- **V3 schema** (`comfy_api.latest`) for all node definitions -- `io.ComfyNode`, `define_schema()`, `io.NodeOutput`
- Do NOT use V1 schema (`NODE_CLASS_MAPPINGS`, `INPUT_TYPES` dict) for new nodes
- Do NOT use Nodes 2.0 -- that's a separate, unrelated thing
- `NODE_CLASS_MAPPINGS` must NOT be present in `__init__.py` -- its presence triggers the V1 code path in ComfyUI's loader (`if/elif` fork in `nodes.py`) and silently blocks the V3 `comfy_entrypoint()`
- `WEB_DIRECTORY = "./web/js"` tells ComfyUI where to find JS/CSS extensions
- Node IDs are prefixed with `EnhancementUtils_` (e.g., `EnhancementUtils_PlaySound`)
- Node categories use existing ComfyUI categories (`utils`, `image`), not a custom top-level category
- Python logging via `logging.getLogger("enhutils.module.name")`
- New nodes: add to `nodes/`, import in `nodes/__init__.py`, add to `ALL_NODES` list

### JavaScript (Frontend Extensions)

- **Plain JavaScript** -- no TypeScript, no build step, no npm
- JS files in `web/js/` are auto-loaded by ComfyUI via the `WEB_DIRECTORY` setting (recursive `**/*.js` glob)
- Extensions register with `app.registerExtension({ name: "phazei.ExtensionName", ... })`
- Import pattern: `import { app } from "../../scripts/app.js"` and `import { api } from "../../scripts/api.js"`
- Console warnings use prefix: `[EnhancementUtils]`
- Vendored JS libraries go in `web/js/lib/` -- they get auto-loaded too (UMD bundles set globals like `window.dagre`, `window.ELK`)
- CSS is loaded manually via `<link>` tag injection from JS (ComfyUI only auto-loads `.js` files, not `.css`)
- Shared utilities in `web/js/utils.js` -- `getUniqueIdFromNode()`, `nodeMatchesUniqueId()`, `findNodeByExecutionId()`, `findNodePath()`
- New JS extensions: add to `web/js/`, follow the `app.registerExtension` pattern

### Monitor

- Background daemon thread with `threading.Event` for clean shutdown
- Uses `asyncio.new_event_loop()` per thread -- NEVER `asyncio.run()` (causes deadlocks)
- Stats pushed via WebSocket: `server.PromptServer.instance.send_sync("enhutils.monitor", data)`
- HTTP routes registered via `@server.PromptServer.instance.routes.patch(...)` decorators
- GPU monitoring: `pynvml` is optional, every single pynvml call is wrapped in try/except
- GPU names decoded with `errors='replace'` (some drivers return non-UTF-8)

### Profiler

- Monkey-patches `execution.execute` and `PromptServer.send_sync` at import time
- `send_sync` intercept captures `execution_start`, `executing` (node start times), and `executing` with `node=None` (execution end)
- `execution.execute` wrapper fires after each node to compute elapsed time
- Handles both sync and async `execution.execute` via `inspect.iscoroutinefunction`
- Emits `enhutils.profiler.executed` (per-node) and `enhutils.profiler.execution_end` (total) via WebSocket
- Console summary logged via `logging.getLogger("enhutils.profiler")`
- Frontend stores profiling data in an external `Map<execId, data>` (not on node objects) so data survives graph/subgraph navigation
- Subgraph container nodes show aggregated total time of their internal nodes
- Live elapsed-time counter on the currently executing node (100ms canvas refresh)

## Code Style

### Python

- Module-level docstring explaining what the file does and where features originated
- Docstrings on all classes and public functions
- Type hints where practical (function signatures, dataclass fields)
- `from __future__ import annotations` is not used -- keep it simple
- Imports: standard library first, then third-party, then ComfyUI/local
- Use `@classmethod` for V3 node methods (`execute`, `define_schema`, `fingerprint_inputs`, `validate_inputs`)

### JavaScript

- JSDoc comments on all functions with `@param` and `@returns`
- Section headers use `// ── Section Name ──────────` (single-line box-drawing chars)
- Major sections use `// ═══════════════════` (double-line)
- `const` by default, `let` only when reassignment is needed, never `var`
- Arrow functions for callbacks and short lambdas
- `async/await` for ELK (which returns Promises), regular functions for everything else
- Defensive checks: `node.inputs || []`, `out.links?.length > 0`, etc.

## Key Gotchas

Things that caused bugs or required non-obvious solutions:

### ComfyUI Loader Fork

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phazei/ComfyUI-Enhancement-Utils](https://github.com/phazei/ComfyUI-Enhancement-Utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
