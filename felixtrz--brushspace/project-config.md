---
trigger: always_on
description: This file provides project context for AI coding assistants working on IWSDK (Immersive Web SDK) projects.
---

# IWSDK Project

This file provides project context for AI coding assistants working on IWSDK (Immersive Web SDK) projects.

## Project Structure

```
my-iwsdk-project/
├── src/
│   ├── index.ts              # World.create() entry point
│   ├── systems/              # Custom systems
│   └── components/           # Custom components
├── public/
│   ├── gltf/                 # 3D models
│   ├── audio/                # Sound files
│   ├── glxf/                 # Scene files
│   └── ui/                   # Compiled UI
├── ui/
│   └── *.uikitml             # UI markup source
├── metaspatial/              # Meta Spatial Editor project
└── vite.config.ts
```

**Convention:** One system per file, with its related components. No barrel `index.ts` files.

---

## Meta Spatial Editor

Meta Spatial Editor is a spatial composition tool for IWSDK. Import, organize, and transform your assets into visual compositions and export them into IWSDK projects.

### mse-agent

mse-agent is the Meta Spatial Editor command-line tool for creating and modifying 3D scenes programmatically. Run `mse-agent readme` for the full command reference.

**mse-agent Location:**

- Mac: `/Applications/Meta Spatial Editor.app/Contents/MacOS/mse-agent`
- Windows: `C:\Program Files\Meta Spatial Editor\V*\Resources\mse-agent` (use the latest version folder)
- Linux: `<package-root>/mse-agent` (mse-agent is located at the root of the downloaded package)

**Before You Start**

- Check Meta Spatial Editor is installed — verify the mse-agent path exists
- If Meta Spatial Editor is not installed, download it from:
  - Mac: https://developers.meta.com/horizon/downloads/package/meta-spatial-editor-for-mac/
  - Windows: https://developers.meta.com/horizon/downloads/package/meta-spatial-editor-for-windows/
  - Linux (headless CLI only): https://developers.meta.com/horizon/downloads/package/meta-spatial-editor-cli-for-linux/
- Launch Meta Spatial Editor with the project scene (Run below commands from the project root directory):
  - **Important:** The editor is a long-running GUI process. You must launch it in the background so it does not block your current process. On Mac, open already returns immediately. On Windows, use start to spawn a separate process. On Linux, the Meta Spatial Editor runs in headless mode. Always wait a few seconds after launching before running mse-agent ping to confirm the editor is ready.
  - Mac: `open -a "/Applications/Meta Spatial Editor.app" "metaspatial/Main.metaspatial"`
  - Windows: `cmd /c start /B "" "C:\Program Files\Meta Spatial Editor\V*\MetaSpatialEditor.exe" "metaspatial/Main.metaspatial"`
  - Linux: `<package-root>/MetaSpatialEditorCLI serve -p app/scenes/Main.metaspatial &>/dev/null &`
- Verify connection: `mse-agent ping`
- Run `mse-agent readme` for the full command reference.

### Rules

- **Use Meta Spatial Editor (mse-agent) when entities are static and primarily define scene composition or layout.** Scenes are visually inspectable in Spatial Editor, which makes review and iteration faster than runtime-only entity creation.

- **Use TypeScript/JavaScript runtime entity creation when entities must be created dynamically.** Spatial Editor scenes are static — if entity creation depends on runtime data, variable counts, or entities that appear and disappear based on state, runtime code is the better fit since you cannot know what to author ahead of time.

- **Use a hybrid approach when a scene has both static and dynamic aspects.** Authoring the static entities in Spatial Editor keeps it visually inspectable and lets designers iterate on composition independently, while keeping runtime-driven content in TypeScript/JavaScript lets engineers focus on behavior and logic in code.

---

## Critical Best Practices

### Browser 3D With First-Class XR

IWSDK is a 3D web framework with first-class XR support. XR can be disabled for a browser-only app with `World.create(container, { xr: false })`, but the runtime still creates `world.player` as the local player/XR origin and keeps `world.camera` under it.

- For first-person browser apps that may later enter XR, move `world.player` for locomotion or WASD-style movement. Treat `world.camera` as the viewer/head under that rig.
- For orbit, editor, product, cinematic, or third-person views, it is fine to keep `world.player` at the origin and drive `world.camera` however the app needs.
- `world.camera.position` is local to `world.player`. Use `world.camera.getWorldPosition(tempVector)` when logic needs the actual viewer position.
- Configure the initial browser view with `render.camera` in `World.create`; do not add camera category presets unless the app explicitly needs one.

Browser pointer input is enabled through `input.canvasPointerEvents` by default. Add `Interactable`/`RayInteractable` and react to `Hovered`/`Pressed` for objects that should work with both mouse/touch canvas input and XR rays. XR-specific input lives at `world.input.xr`; use `world.input.keyboard` and `world.input.browserGamepads` for low-level browser controls. Reusable systems should prefer `world.input.actions` for intent such as `locomotion.move` or `locomotion.jump`; opt into browser locomotion bindings with `features.locomotion.browserControls`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felixtrz/brushspace](https://github.com/felixtrz/brushspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
