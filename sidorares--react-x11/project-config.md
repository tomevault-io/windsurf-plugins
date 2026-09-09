---
trigger: always_on
description: handles are components. Notes below under "Writing a window manager".
---

# AGENTS.md

Guidance for AI agents (and new contributors) working on react-x11.

## What this project is

A custom React renderer whose host environment is an X11 server, with
react-like ergonomics on top of [ntk](https://github.com/sidorares/ntk) /
[node-x11](https://github.com/sidorares/node-x11) — pure JavaScript
implementations of the X11 protocol, no native bridge.

Architecture (see NEXT_STEPS.md for the full rationale): only `<window>`,
`<popup>`, `<glarea>` and `<foreign>` map to real X11 windows (`<glarea>`
because GLX needs its own visual, `<foreign>` because the window is another
process's); everything else is a retained
lightweight node — one yoga-layout node each — painted into the owning
window's double-buffered 2d context on ntk's frame clock, with synthetic
capture/bubble events dispatched via front-to-back hit testing. X11
windows are created **top-down in the commit phase**
(`WindowNode.realize()`): `createInstance` performs no X11 calls (the
render phase is discardable under concurrent React), and every
CreateWindow names its actual parent from the start — no ReparentWindow,
no override-redirect staging (issue #4).

## Layout

- `src/index.js` — public entry (`createRoot`; the legacy
  `render`/`unmountComponentAtNode` pair was retired in #114).
- `src/Reconciler.js` — react-reconciler host config + render entry points.
  Written against react-reconciler 0.33 (React 19). If you upgrade
  react-reconciler, expect host config contract changes; the smoke test is
  the safety net.
- `src/nodes.js` — the retained node tree: `WindowNode` (real X window,
  paint/event/flex root), `BoxNode`, `TextNode` (+ spans/chunks),
  `ImageNode`, `CanvasNode`. Layout (yoga), painting, hit testing.
- `src/glnodes.js` — `<glarea>`: the GL surface. A child X window on a
  GLX visual (ntk's `chooseGLXConfig`), positioned by the parent's yoga
  rect, drawing `onDraw` frames on its own frame clock. First step of
  docs/glx.md.
- `src/foreignnodes.js` — `<foreign>`: another process's window, embedded.
  A second `drawn: false` node, over ntk's `XEmbedSocket` (docs/embedding.md).
  Two things here are not obvious and are commented at length. **Teardown is
  synchronous** — `WindowNode` destroys its own X window in the same turn, and
  `DestroyWindow` takes every inferior with it, so a release that waits for a
  round trip releases a window that is already gone; the client is reparented
  to the root and dropped from the save set before the container goes, and
  **never destroyed**. And **no focus proxy**: the classic XEmbed embedder
  gives the X focus to an InputOnly window that forwards keys, which here
  would read as the toplevel losing focus and would put every key past the
  React tree before any handler saw it. The X focus stays on our window and
  forwarding happens in `defaultKeyDown`/`defaultKeyUp`, which is what makes
  the rule "app chords first, everything else forwards" mechanical.
- `src/frame/` — `<Frame>`: a pane of the application in its own process,
  its window embedded over `<foreign>` (docs/frame.md). Four small files
  along the seams: `protocol.js` (pure — the six messages, the callback
  table with its one-update grace window, argument sanitizing), `env.js`
  (the context bridge: `createFrameContext`, the `FrameEnv` accumulator the
  theme publishes into), `index.js` (the host: fork, supervision,
  coalesced updates, fallback), `childmain.js` + `child.js` (the pane
  bootstrap behind an injectable transport, which is what lets the tests
  run a real pane in-process over a loopback — `test/frame.test.js` forks
  for real exactly once, over a TCP bridge onto the in-process server).
  Two things are deliberate and easy to undo: the **update listener goes up
  before the pane module import starts** (a store the tree subscribes to
  later — a listener scoped to the mounted tree would drop every update
  that landed during the import), and the host's transport listeners
  **outlive the effect cleanup until the exit**, because a `useFrameClose`
  handler flushing through a callback prop sends its `invoke` after the
  unmount message.
- `src/scene3d.js` — the 3D scene tree inside a `<glarea>`: `<mesh>`,
  `<group>`, geometry/material nodes and the renderer that compiles each
  geometry into a server-side **display list** (a frame is matrices +
  material state + one `CallList` per mesh). `src/geometry3d.js` generates
  the primitives, `src/mat4.js` is the matrix math, `src/raycast3d.js` +
  `src/pointer3d.js` are picking and mesh pointer events.
- `src/svgnodes.js` — `<svg>` over ntk's `SvgView`: `SvgNode` (sized from
  its viewBox like `<image>`, cached as coverage when the drawing is one
  colour) and `SvgChildNode`, the declarative SVG elements underneath it,
  serialized into the DOM SvgView consumes. It used to hold `<markdown>`,
  `<html>` and `<tex>` beside it, over ntk's document widgets; those were
  removed in 2.0 — a document rendered as one opaque widget can neither be
  selected across blocks nor re-rendered a block at a time, and the
  successors are `<Markdown>`/`<Formula>` in `@react-x11/components`,
  composed from public host elements.
- `src/yoga.js` — the layout engine. Enums synchronously, WebAssembly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sidorares/react-x11](https://github.com/sidorares/react-x11) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
