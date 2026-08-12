---
trigger: always_on
description: validates each against that subtype's **own engine `Handler.validate_payload`**
---

# CLAUDE.md — project conventions

File teach future Claude (and human) contributors load-bearing conventions of codebase. Keep short, honest.

## Stack

- Python ≥ 3.10 (target 3.10/3.11/3.12 in CI; embedded runtime 3.12.10).
- ComfyUI V3 API (`comfy_api.latest`) — no V1 `NODE_CLASS_MAPPINGS`.
- Vue 3 + Vite 5 + pnpm. Strict TypeScript.
- Vitest for frontend, pytest for Python.

## Directory contract

- `engine/` — pure Python, **zero ComfyUI imports**. Testable with vanilla pytest.
- `wp_nodes/` — ComfyUI V3 nodes. Named `wp_nodes` (not `nodes`) because ComfyUI repo-level `nodes.py` shadow any top-level `nodes/` package on `sys.path`.
- `src/` — Vue/TS frontend. Two Vite targets: `extension` (critical path, budget-gated) + `manager` (post-MVP SPA).
- `tests/` — Python tests. Use `conftest.py` sys.path shim.
- `scripts/` — Node helpers (bundle-size gate, size-delta diff, release helpers).
- `docs/widget-context-contract.md` — single canonical engine ↔ widget JSON shape doc. Update both Python (`engine/modules.py`) + TS (`src/widgets/_shared.ts`) sides in one PR when shape change.
- `docs/superpowers/{specs,plans}/` — gitignored per-contributor design docs + impl plans.

## Commands

```bash
pnpm build:extension   # produces js/main.js
pnpm build:manager     # produces web_dist/
pnpm test              # Vitest
pnpm test:coverage     # Vitest + v8 coverage (with thresholds)
pnpm typecheck         # vue-tsc --noEmit
pnpm lint              # ESLint flat config
pnpm size              # bundle-size gate (--json <path> for manifest)
pnpm check:css-isolation  # flag non-wp-prefixed top-level CSS selectors
pytest                 # Python unit tests
ruff check .           # Python lint
```

## Load-bearing conventions

- **Engine isolation** — never import from `wp_nodes/`, `comfy_api`, or `torch` in `engine/`. Engine dict-free of ComfyUI globals. Break this break tests.
- **Internal keys** — context keys starting with `__` engine-internal; strip at socket boundaries (see `wp_nodes/types.py:strip_internals`).
- **Module IDs** — 8-hex-char short UUIDs. Match wildcard `@{uuid}` reference syntax planned for next spec.
- **V3 node IDs** — prefix `WP_`, category `wildcard-pipeline`.
- **Custom widget types** — `WP_CONTEXT_MODULES`, `WP_DEBUG_VIEWER`. Registered via `app.registerExtension({ getCustomWidgets })`. Assembler helper widget injected via `beforeRegisterNodeDef → onNodeCreated`, not `getCustomWidgets`, because sit alongside native STRING template widget.
- **Toast singleton** — one Vue app mounted to body-level div renders all toasts pushed via `src/components/shared/toast-store.ts`. Every Context node import same store so toasts surface from anywhere. Position mirror ComfyUI native PrimeVue toast anchor (rect-relative to `.graph-canvas-container`).
- **Subgraph traversal** — `collectUpstreamVariables` / `collectUpstreamValues` / `findDownstreamAssemblers` cross subgraph boundaries via `-10` SubgraphInputNode + `-20` SubgraphOutputNode sentinel ids. Walkers expect `app.graph` (root); per-call `buildSubgraphParents` map handle step-out. Use `walkAllNodes(rootGraph)` for graph-wide scans (e.g. pre-run validation).
- **Subgraph badges** — `extension/subgraph-badge.ts` attach `LGraphBadge` (via `window.LGraphBadge`) to any SubgraphNode containing WP nodes. Badge surface worst inner conflict severity with human label (e.g. `missing $foo`, `2 conflicts`). Wire via `nodeCreated` + `loadedGraphNode` hooks — `beforeRegisterNodeDef` does NOT fire for SubgraphNodes.
- **Conflict scanner** — advisory only; never block execution. Runtime last-write-wins. Rule types: `shadows_upstream` (info), `duplicate_variable` (warning), `missing_template_variable` (warning), `constraint_source_missing` / `constraint_target_missing` (warning — uuid not in catalog), `constraint_orphan_source` (warning — no source instance UPSTREAM), `constraint_orphan_target` (warning — the constraint's reach selector covers ZERO reachable downstream instances; per-selector, NOT count-aware — overlapping constraints may freely share an instance). Engine emits runtime warnings `constraint_never_applied` (the reach matched nothing at runtime) and `constraint_partial_reach` (a `next N` / `pick` reach matched fewer instances than requested).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DumiFlex/ComfyUI-Wildcard-Pipeline](https://github.com/DumiFlex/ComfyUI-Wildcard-Pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
