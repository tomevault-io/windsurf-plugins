---
trigger: always_on
description: > Before adding anything that crosses between C# and JavaScript, read
---

# OneJS v3: Agent Guide

> Before adding anything that crosses between C# and JavaScript, read
> [DESIGN.md](DESIGN.md). It states what belongs on each side, the four rules a
> wrapper has to satisfy, and why an interpreter on native makes it a portability
> question rather than a performance one.

Condensed facts for AI agents working with OneJS. Full documentation: https://onejs.com/docs

OneJS runs React 19 + TypeScript UIs inside Unity. TSX compiles with esbuild into a single bundle that the `JSRunner` component executes through QuickJS (or the browser's own JS engine on WebGL), rendering through UI Toolkit. No webview, no DOM.

## Requirements

- Unity 6000.3+ (Unity 6.3)
- Node.js 18+ on the dev machine

## Install

Package Manager: `+` > Add package from git URL:

```
https://github.com/Singtaa/OneJS.git
```

Or clone into Assets: `git clone https://github.com/Singtaa/OneJS.git`. Package id: `com.singtaa.onejs`. No other Unity packages or scoped registries required.

## Project setup

1. Add the `JSRunner` component to a GameObject in a saved scene.
2. Click **Initialize Project** in its inspector. This creates, next to the scene:

```
{SceneDir}/{SceneName}/{GameObjectName}/
├── ~/                      # TS/TSX source (the ~ suffix keeps Unity from importing it)
│   ├── index.tsx           # entry point
│   ├── package.json, tsconfig.json, esbuild.config.mjs
│   └── styles/main.uss, types/global.d.ts, AGENTS.md, .gitignore
├── PanelSettings.asset     # project marker - the one mandatory JSRunner field
├── UIDocument.uxml
├── app.js.txt              # built bundle (esbuild writes ../app.js.txt)
└── app.js.map.txt
```

then runs `npm install` and `npm run build` automatically.

Alternative: skip the button and just enter Play mode. On first play, `JSRunnerAutoWatch` auto-creates and assigns PanelSettings, scaffolds missing files, runs `npm install` + `npm run build` in the background, and starts the watcher. The first run takes a moment while packages install.

Key model:

- **Panel Settings is the project marker.** The folder containing the assigned PanelSettings asset defines the project; the bundle is always loaded from `{that folder}/app.js.txt`. A folder is considered valid when it contains `~/` or `app.js.txt`.
- **Do not add a UIDocument.** JSRunner adds and wires one at runtime (and an EventSystem if missing). On Unity 6.5+, do **not** substitute the new `PanelRenderer` either: it never attaches a panel outside Play mode, so edit-mode preview would silently render nothing. OneJS stays on `UIDocument`, which is not `[Obsolete]`. See `Runtime/README.md`.
- **Edit-mode preview**: JSRunner renders the UI in the Game view without Play mode whenever Panel Settings is valid, `app.js.txt` exists on disk, and the UIDocument root is ready. Ticks at 30 Hz. Ticking is gated by the Scene view's **OneJS overlay** update modes: in the default Auto mode only the selected runner (or, with nothing selected, the one closest to the Scene view camera) ticks, so an unselected runner's timers freeze while its hot reload keeps working. Select the runner or switch the overlay's Scene mode to Camera when driving previews headlessly.
- Scaffolding never overwrites existing files.

Headless (no inspector click): the three methods the button calls are public on `JSRunner`, `PopulateDefaultFiles()`, `EnsureProjectFolderAndAssets(true)`, `EnsureProjectSetup()`, callable from a small editor script (which `-executeMethod` can invoke); then run `npm install && npm run build` in `~/`.

## Build and live reload

All commands run inside `~/`:

- `npm run build`: one-shot bundle
- `npm run watch`: rebuild on save
- `npm run typecheck`: `tsc --noEmit`

Inside the editor, the esbuild watcher is **managed automatically** (`JSRunnerAutoWatch` / `NodeWatcherManager`): it starts when edit-mode preview begins (if `node_modules` exists) and on entering Play mode (running `npm install`/`build` first if needed), and stops on Play mode exit. Running `npm run watch` manually also works and is the path for working outside the editor.

JSRunner itself watches the **built bundle** (`app.js.txt`), not source files: an MD5 content poll every 0.5 s plus a FileSystemWatcher. So the loop is simply: save a source file, the watcher rebuilds the bundle, Unity hot-reloads, in both edit-mode preview and Play mode.

Reload is a **hard reload** (fresh JS context, all JS state lost): `onStop()` (if playing) → Janitor destroys JS-created GameObjects → React teardown (`useEffect` cleanups DO run) → new context, globals re-injected → bundle re-runs → `onPlay()` (if playing).

Player builds: `JSRunnerBuildProcessor` embeds the bundle as a TextAsset automatically. The bundle never goes through StreamingAssets.

## Authoring

The entry file ends with a render call:

```tsx
import { render, View, Text } from "onejs-react"

function App() {
    return <View style={{ flexGrow: 1 }}><Text text="Hello" /></View>
}

render(<App />, __root)

export function onPlay() {}   // Play mode start + after hot reload while playing
export function onStop() {}   // Play mode exit + before hot reload while playing
```

- `__root` (root VisualElement) and `__isPlaying` are ambient globals (declared in `types/global.d.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Singtaa/OneJS](https://github.com/Singtaa/OneJS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
