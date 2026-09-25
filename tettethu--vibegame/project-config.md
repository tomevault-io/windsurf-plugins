---
trigger: always_on
description: 1. This project uses **Phaser 3 (CE)**. Never use Phaser 2 APIs.
---

## Vibegame Project

1. This project uses **Phaser 3 (CE)**. Never use Phaser 2 APIs.
2. Read the relevant files before acting on the game project. Do not guess game state or asset inventory.
  - `.vibegame/GDD.md` — Game design document (mechanics, characters, art needs)
  - `.vibegame/assets.md` — Asset inventory (registered sprites, tilesets, audio)
  - `.vibegame/spec/` — Engine guides, system specs, design theory
  - `index.html` — Game entry point
3. When outputting a file path for user review, always use markdown link format `[name](absolute_path)`. Do not use relative paths or `[name](file://absolute_path)`
4. Never modify files outside the game project directory unless the user explicitly permits it. When deleting files, prefer recoverable methods (e.g. `trash` on macOS) over `rm`.
5. Never use `pkill -f 'vibegame'`; it can terminate the entire VibeGame agent team, Dashboard, runtime, and tmux session.

## User Interaction

You must interact with the user in this way:
1. **Always** respond and write docs in: **{{language_name}}**. File names, identifiers, code, and engine vocabulary stay in English.
2. **DO NOT** use `AskUserQuestion` or other equaliant tools. To ask user questions or ask user to choose between options, just output the question and options.


## Glossary

Canonical vocabulary used by all agents when talking to the user and to each other. Use these terms; avoid synonyms. Do not switch terminology between turns. When introducing a term to a non-developer user, give a plain-language analogy on first mention, then use the canonical term consistently after.

### Engine primitives
- **node** — A unit in the scene tree, optionally driven by a script. A node can be placed at scene load time (declared in scene JSON) **or spawned at runtime** via `instantiate()` — both are equally first-class. Bullets, enemies, pickups, projectiles, particles are normally runtime-spawned, not pre-placed. Similar in concept to a Unity GameObject or a Godot Node, but the API is different — do not call it `entity`, `actor`, `GameObject`, or `prefab`.
- **scene** — A `.scene.json` file containing the initial node tree plus startup configuration. The runtime loads one scene at a time. Runtime-spawned nodes are added on top of this initial tree.
- **`.node.json` template** — A reusable node definition file (visual + collider + script + config defaults). The vibegame equivalent of a prefab. Used with `instantiate()` to spawn dynamic content (bullets, enemies, pickups). Procedural content **must** use a template — never assemble nodes from raw objects.
- **instantiate** — The runtime action of spawning a `.node.json` template as a child of an existing node: `await this.instantiate('entities/bullet.node.json', { x, y })`. The standard way to create anything dynamic.
- **script** — A JS file under `scripts/` that extends `Node` (from `engine/Node.js`) and drives one node's behavior. Scripts used by runtime-instantiated nodes must be registered (manually or via a disabled placeholder node) so the engine can find them. Not a Unity MonoBehaviour.
- **project** — A user game directory rooted at `project.json`. Contains scenes, scripts, assets, and config.

### Assets
- **sprite** — A single 2D image, usually one PNG.
- **sprite sheet** — A PNG containing multiple frames laid out in a grid (animation frames or direction variants in one image).
- **animation** — A timed sequence of frames, typically sliced from a sprite sheet.
- **tileset** — A set of equal-sized tile images plus tile metadata (collision, auto-tile rules). One PNG paired with one `.tileset.json`.
- **tilemap** — A grid-based level authored with a tileset. One `.tilemap.json`. Distinct from a free-placed sprite scene.
- **manifest** — `assets/manifest.json`. Declares which assets the runtime should preload.
- **pivot** — Fractional anchor `[x, y]` in `0..1`, `[0,0]` = top-left. Applies to both visual (sprite/atlas) and collider. **Visual pivot** (in manifest or `animations.clips.<n>.pivot`) determines Phaser sprite origin so per-frame bbox size changes don't visually shift the character; default `[0.5, 1]` (bottom-center). Cascade: clip-level > manifest sprite-level (`<asset>.sprites.<frame>.pivot`) > manifest group-level (`<asset>.pivot`) > default. **Collider pivot** (`collider.pivot` in node.json) determines where on the collider box the anchor sits; defaults to inherit the visual sprite's resolved origin, so feet-anchored sprites get feet-aligned bodies for free. Cascade: explicit `colliderDef.pivot` > visual sprite origin > `[0.5, 0.5]`. Circle shape defaults to `[0.5, 0.5]` regardless. See [animation-guide.md#pivot](.vibegame/spec/engine/animation-guide.md) and [collision-guide.md#pivot](.vibegame/spec/engine/collision-guide.md).

### Behavior

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tettethu/VibeGame](https://github.com/tettethu/VibeGame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
