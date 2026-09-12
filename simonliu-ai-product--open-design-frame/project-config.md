---
trigger: always_on
description: You are working on the **open-design-frame framework** — the runtime, Vite plugins, and CLI.
---

# open-design-frame — Framework Repo Guide

You are working on the **open-design-frame framework** — the runtime, Vite plugins, and CLI.

open-design-frame is to design artboards what open-doc is to documents and open-slide is
to decks: the work is written as React components, and the viewer is a canvas
rather than a page or a projector.

## Layout

pnpm + Turbo monorepo.

| Path | Package | Role |
| --- | --- | --- |
| `packages/core` | `@open-design-frame/core` | Runtime (canvas, frame rail, inspector, assets), Vite plugins, `open-design-frame` dev/build CLI. |
| `packages/mcp` | `@open-design-frame/mcp` | MCP server over the same ops. Opt-in: `open-design-frame dev --mcp`. |
| `apps/demo` | private | Local consumer via `workspace:*`. Dogfood target — `pnpm dev`. |

## Workflow

```bash
pnpm dev          # turbo: runs the demo against local core
pnpm build        # build all packages
pnpm typecheck    # tsc across the graph
pnpm check        # biome (format + lint)
pnpm check:fix    # auto-fix what biome can
pnpm test         # vitest
```

**After changing `packages/core/src`, rebuild it (`pnpm --filter @open-design-frame/core build`)
before testing the demo** — the demo's `open-design-frame` binary runs the built `dist`.

## What a frame is

`frames/<id>/index.tsx` default-exports an array of frames. A frame is a
component that carries its own size:

```tsx
const Dashboard: Frame = () => <div>…</div>;
Dashboard.frameName = 'Dashboard · Desktop';
Dashboard.size = SIZES.DESKTOP;
export default [Dashboard];
```

`meta` names the file, `design` supplies the tokens (`--odf-*`). Wrapping a part
of the frame in `<Layer name="…">` is what puts it in the inspector's tree —
unwrapped markup still renders, it just cannot be selected.

## Architecture notes

- **The size travels with the frame, not the project.** open-doc fixes A4 and
  open-slide fixes 16:9 because the medium decides; a design has no medium, so
  `Frame.size` is per-frame and `SIZES` only names the common ones.
- **One renderer for three surfaces.** The home cover, the rail thumbnail and
  the canvas are all `FrameCanvas` at different scales — a thumbnail that
  disagrees with what opens is not a state this can reach. The canvas scales the
  container, never the contents, so every length inside a frame is the length
  that will be exported.
- **A fitted canvas measures its own box.** `FrameCanvas` without `scale` reads
  its container with a ResizeObserver, so the container must have a size of its
  own — a scrollable parent using `place-items: center` grows to fit the frame,
  and the fit then reads that larger number and never shrinks.
- **The layer tree is read from the DOM, not the source.** A frame is ordinary
  JSX — helpers, `map`, conditionals — so the component tree and the picture
  disagree the moment anything is generated. `collectLayers` walks rendered
  `[data-odf-layer]` elements, and the effect that runs it depends on the loaded
  module: keyed only by the frame id it runs once against a null ref.
- **One implementation, two callers.** Everything that touches the workspace
  lives in `src/ops` and raises `OpsError` rather than writing status codes.
  The dev server's routes turn that into HTTP; the MCP tools turn it into a
  tool error. A second implementation of "rename a frame" would drift the first
  time one of them learned something the other did not.
- **A write carries the revision it read.** `write_frame` hashes the source it
  is replacing and refuses a stale one, because the other half of this
  workspace is a person with the file open.
- **Picking happens on the canvas.** Most of a frame is never wrapped in a
  named `Layer`, and the thing a designer is looking at is the thing they point
  at — so the click handler takes the innermost element carrying a source tag,
  and the layer list is a second way in, not the only one. A frame's own links
  and buttons do not follow on click; this is a canvas, not a preview.
- **A selection is a description, not a reference.** An edit reloads the module
  and every element in the frame becomes a new object, so `Pick` records the
  layer id, else the source location plus which of that site's renders it was —
  a held node would go stale exactly when the panel is meant to show the result.
- **An edit previews itself.** Changing a value writes the element's own inline
  style, which is what the save will write into the source, so what the canvas
  shows between the change and the save is the result rather than a guess at it.
  Discard reloads: undoing a preview by hand would be a second implementation of
  the same thing, and the source is already the answer.
- **The inspector edits source, not the DOM.** `loc-tags-plugin` stamps
  `data-odf-loc="line:col"` onto host JSX in `frames/**` (dev only); the panel
  posts ops to `/__edit`, and `editing/edit-ops.ts` rewrites the file with
  Babel. Edits are staged until saved — a write and an HMR reload behind each
  digit of a font size is not an edit, it is a stutter.
- **`Layer` forwards its loc tag.** Capitalised JSX is normally skipped because
  a component's location is its call site, but `Layer` renders one host element

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simonliu-ai-product/open-design-frame](https://github.com/simonliu-ai-product/open-design-frame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
