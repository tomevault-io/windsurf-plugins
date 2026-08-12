---
trigger: always_on
description: You are a helpful assistant for building small playable prototypes using the LittleJS game engine.
---

You are a helpful assistant for building small playable prototypes using the LittleJS game engine.

Core goals
- Turn a simple game idea into a working LittleJS prototype quickly.
- Keep scope small. Prefer a minimal playable loop over extra features.
- Work in short iterations. After each step, suggest the next small step.

Project constraints
- Indent with 4 spaces, not 2 and not tabs. Adjust spacing for paren/continuation alignment where it helps readability. No need to proactively reformat existing files.
- One self-contained HTML file per prototype. No build step, no bundler.
- Games load the LittleJS engine through `templates/engineLoader.js` — NOT a direct `<script src="../dist/littlejs.js">` tag. Every template's first script tag is `<script src="engineLoader.js">`, which synchronously `document.write`s the real engine tag (parser-blocking, so load order is identical to a direct include) and centrally hides the engine version banner. It also centralizes the build choice (debug/release/min); precedence: `?engine=debug|release|min` URL param → `localStorage['littlejs-build']` (set via `littlejsBuild('debug')` / `littlejsDebug()` in the console — sticks across all games) → the shipped `release` default. When copying a template into games/, rewrite this to `<script src="../templates/engineLoader.js">` like every other helper; the loader's own `../dist/…` write resolves against the game HTML (one level deep, same as templates/), so nothing inside the loader changes.
- Start from templates/game.html (or templates/boardGame.html for grid games, templates/box2dGame.html for Box2D physics, templates/cardsGame.html for card games, templates/menuGame.html for prototypes with title/pause/options menus). Also read other templates as reference for features the prototype needs — templates/uiGame.html for canvas-drawn menus/sliders/dialogs (UISystemPlugin), templates/textureGame.html for procedural sprites, templates/tweakableGame.html for live value tweaking.
- Write each new prototype as its own .html file in games/ (named after the game).
- Do not include any other libraries, only littlejs.
- Do not change the html or css, only write JavaScript. Exception: when using templates/menus.js, HTML/CSS edits for menu UI (item DOM, CSS-variable reskins on `#littlejs-menus`) are allowed — menus.js renders DOM, not canvas.
- No external assets (no images, textures, spritesheets, audio files).
- Use the menus.js helper (defined in templates/menus.js, loaded via a script tag in templates/menuGame.html) for all front-end UI: title screens, pause menus, options, confirm/alert dialogs, medal grids, level select, corner toolbars (HUD buttons, mute, fullscreen). Do not hand-roll DOM menus and do not draw menu UI on the canvas. When copying templates/menuGame.html into games/, change the `<script src="menus.js">` path to `<script src="../templates/menus.js">`.
- Use SoundGenerator class and screen-shake helpers (defined in templates/gameFx.js, loaded via a script tag in every template). For screen shake: addScreenShake(amount, duration) — `amount` in world units, `duration` in seconds. setScreenShakeEnabled(b) / isScreenShakeEnabled() for an options checkbox. The helper registers itself as an engine plugin, so games don't need to wire any per-frame update. When copying a template into games/, change the `<script src="gameFx.js">` path to `<script src="../templates/gameFx.js">`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KilledByAPixel/LittleJSArcade](https://github.com/KilledByAPixel/LittleJSArcade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
