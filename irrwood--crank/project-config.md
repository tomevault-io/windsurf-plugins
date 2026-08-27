---
trigger: always_on
description: Crank is an independent local-first desktop application for translating editable UI structure between source projects and Figma. It is not part of MOMO and must not assume that MOMO exists on the machine.
---

# Crank project instructions

## Product objective

Crank is an independent local-first desktop application for translating editable UI structure between source projects and Figma. It is not part of MOMO and must not assume that MOMO exists on the machine.

The active product scope is web and Electron projects. Prefer Electron and Chromium's official runtime APIs over source-language layout reconstruction. Existing SwiftUI connections may remain readable for backward compatibility, but do not extend SwiftUI parsing or Design Build unless the product direction is explicitly changed again.

## Architecture rules

1. Never hard-code a customer project path, Figma file, node ID, project name, or pairing code.
2. Treat every connected source folder and Figma file as user-owned external data.
3. Local means nothing about the project leaves the machine: send only normalized visual structure, and only during an explicit sync action. It does not mean capture refuses to load what the page itself loads — see rule 11.
4. Capture third-party renderers in an isolated, sandboxed Electron session with Node integration disabled. Block what can change the project or run someone else's code in it — any non-GET request, and off-host scripts and data calls. Subresources the page merely draws are not that; see rule 11.
5. Stable source identity comes from source semantics and deterministic DOM identity. Runtime instances and frames are observations attached to that identity.
6. Preserve editable Figma layers and remembered frame identity. Do not replace linked frames with screenshots.
7. Validate IPC, bridge payloads, stored registry data, and runtime capture data with Zod.
8. Never substitute a missing font, asset, renderer build, or unsupported dynamic state *silently*. Substituting and naming what was substituted is the required behaviour; refusing is not. Twice this rule was implemented as a refusal and cost far more than it protected: a page with one unavailable font produced no layers at all, and a font Figma did not have left an empty frame on the canvas after the frame had already been created.
9. A selected folder can be a workspace. Discover every independently runnable application package and register each one as its own project.
10. Raster fallbacks must be bounded to the unsupported renderer itself. If a page contains SceneKit, Metal, WebView, video, canvas, or another opaque renderer, capture only that renderer's visible bounds as an image and preserve the rest of the page as editable text, shapes, layout, and vector layers. The presence of an opaque descendant must never cause its ancestor, page, or entire window to be rasterized.
11. A browser is the floor. A page that renders correctly when someone simply opens it must not come back from a scan worse than that — missing its typeface, its images, or its layers. Capture is a browser; producing less than one is a defect, never a trade-off to be weighed. When something genuinely cannot be captured, report the gap and deliver the rest: a partial result names what is missing, an empty one names nothing.
12. Prefer a deterministic anchor over a resemblance. Identity derived from source — an attribute injected at build time, a route, a recorded click path — beats identity inferred from what a node looks like, and the inferred kind is the fallback for projects whose build Crank does not control. Injected anchors must never be written into the user's files.

Rules here are decisions, not axioms. Where evidence contradicts one, change it and say what the evidence was; rules 3, 4 and 8 were narrowed after each was read as a prohibition it never stated.

## How this repository is laid out

| Area | Language | What it owns |
| --- | --- | --- |
| `electron/` | CommonJS `.cjs` on Node | Everything that touches the machine: starting projects, driving Chromium, capture, storage, the Figma bridge. |
| `src/` | TypeScript + React 18, built by Vite | The window. Reaches the machine only through `window.uiSync`. |
| `shared/` | Plain ESM `.js` with a hand-written `.d.ts` | The few decisions both sides must agree on. Loaded by `await import()` in the main process and bundled by Vite in the renderer, so it may use neither Node nor the DOM. |
| `figma-plugin/` | Plain JS against the Figma Plugin API | The other half of a sync. No build step — `code.js` ships as written, typechecked by `npm run typecheck:plugin`. |
| `figma-plugin/listing/` | — | The Community submission: the copy and the artwork it is published with. |
| `swift-tools/`, `swift-sdk/` | Swift | The dormant SwiftUI path. |
| `public/`, `assets/` | — | `public/app-icon.png` is the icon; `assets/` holds the macOS `.icns` variants built from it. |

Read the docblock at the top of a module before reading the module. Nearly every
file in `electron/` opens with one that says what the file is for and, more
usefully, what went wrong before it existed. `asset-store.cjs`,
`node-identity.cjs`, `page-origin.cjs` and `browsing-session.cjs` are the four to
read first. Grepping for a symbol tends to land in the wrong file; those
docblocks are the map.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [irrwood/Crank](https://github.com/irrwood/Crank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
