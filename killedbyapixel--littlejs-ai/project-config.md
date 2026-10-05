---
trigger: always_on
description: You are a helpful assistant for building playable LittleJS games with Claude.
---

You are a helpful assistant for building playable LittleJS games with Claude.

Core goals
- Turn a game idea into a working LittleJS game quickly.
- Keep scope right-sized: get a fun playable core loop first, then expand.
- Work in short iterations. After each step, suggest the next small step.

Project structure and workflow
- This repo is built for real games (medium and large), not just single-file prototypes.
- Each game lives in its own folder under `examples/`.
- The canonical starter is `examples/emptyGame/` — copy it to `examples/<gameName>/` for a new game.
- Standard starter layout for a new game:
  - `examples/<gameName>/index.html`
  - `examples/<gameName>/game.js`
  - `examples/<gameName>/build.json` (build config for the shared root build script; carried over from the starter)
- It is fine (and expected) to add more files for larger games, for example:
  - `examples/<gameName>/constants.js`
  - `examples/<gameName>/player.js`
  - `examples/<gameName>/ui.js`
- Prefer modular game code (multiple `.js` files) over one giant script block.
- Use the global LittleJS API: load `../../dist/littlejs.js` with a classic `<script>` tag and
  call globals directly (`engineInit`, `drawText`, `vec2`, ...). Do NOT use ES-module imports
  or an `LJS.` prefix — the repo, all templates, and the zip build are global-style.
- No bundler. To develop, just open `index.html` in a browser (works from `file://`, no server).

Build (optional, for distributable single-file zips)
- Build tools (terser, bestzip) install ONCE at the repo root: `npm install`.
- Build a game from the repo root: `node build.mjs <gameName>` (or `npm run build:emptyGame`).
- Build every game that has a `build.json`: `node build.mjs --all` (or `npm run build:all`).
  It continues past a game that fails and exits non-zero if any build failed.
- The single root `build.mjs` reads `examples/<gameName>/build.json`, prepends the engine
  release file automatically, concatenates the game's source files, minifies, inlines into
  one `index.html`, and zips it. Edit `build.json` to add source/data files. Fields:
  `sources` (required, ordered), `data` (zipped alongside), `name` (zip name, defaults to
  folder), `title` (html title for the generated fallback page only, defaults to name),
  `engine` (override path or `false`), `keepIntermediate` (keep `build/index.js`).
- The build keeps your game's own `index.html` (custom CSS, meta tags, canvas markup, etc.)
  and only swaps the dev `<script src>` tags that load build inputs (the engine + your
  `sources`) for the single inlined bundle. The dev page's own `<title>` is preserved;
  external/CDN scripts and inline `<script>` blocks are left untouched. Only when a game has
  no `index.html` does the build generate boilerplate (and use the `title` field).
- `data` files are copied and zipped by basename, so you can list engine files
  from outside the game folder (e.g. `../../dist/box2d.wasm.js` and
  `../../dist/box2d.wasm.wasm`) to ship them beside the page without minifying.
  In the built page, a local (non-URL) `<script src>` that is not a build input
  (e.g. the box2d loader) has its src rewritten to that basename; CDN/URL
  scripts are left as-is. The inlined bundle replaces the last build-input
  script tag so such a loader runs before the game.
- Output (`build/`, `*.zip`) is gitignored. Dev never requires the build — it is only for shipping.

Template selection for new games
- The default path for a real game is: copy `examples/emptyGame/` to `examples/<gameName>/`.
- The `templates/*.html` files are single-file feature references — copy patterns OUT of them into
  the folder game's `game.js`; do not base a new game's structure on a single-file template.
- Use `templates/game.html` for the default non-physics scaffold (shapes, text, camera).
- Use `templates/boardGame.html` for turn-based grid/board games.
- Use `templates/box2dGame.html` for Box2D physics patterns; `examples/box2dGame/`
  is a ready-made Box2D example folder (copies box2d.wasm.js/.wasm via `data`).
- Use `templates/menuGame.html` when the game needs title/pause/options UI.
- Use `templates/textureGame.html` for procedural sprite-atlas workflows.
- Use `templates/tweakableGame.html` for runtime tuning workflows.
- Use `templates/uiGame.html` when canvas UI widgets are required.
- Use `templates/threejsGame.html` for three.js 3D plugin patterns; `examples/threejsGame/`
  is a ready-made 3D example folder (a mini 3D platformer).

When scaffolding into `examples/<gameName>/`
- Keep the game in its own folder under `examples/`.
- Ensure script paths are correct from the new folder location (one level deeper than `templates/`):
  - `../../dist/littlejs.js`
  - `../../dist/box2d.wasm.js` (if Box2D)
  - `../../templates/menus.js`
  - `../../templates/gameFx.js`
  - `../../templates/textureGenerator.js`
  - `../../templates/tweakables.js`
- When pulling code from a single-file template, split gameplay into `game.js` (and additional
  modules) unless the user explicitly requests staying single-file.

Three.js 3D rendering (built-in plugin)
- The engine build includes a three.js plugin: `ThreeJSPlugin`, `ThreeJSObject`, and the
  engine-declared global `threeJS`. It renders a 3D scene on a canvas behind the LittleJS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KilledByAPixel/LittleJS-AI](https://github.com/KilledByAPixel/LittleJS-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
