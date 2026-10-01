---
trigger: always_on
description: Omastrator is a vector illustration app for Linux, modelled on Adobe Illustrator
---

# Omastrator: notes for agents

Omastrator is a vector illustration app for Linux, modelled on Adobe Illustrator
and made first for Omarchy. It is C++20 with Qt 6 Widgets (Qt 6.4 at minimum),
built with CMake and tested with Qt Test.

## Layout

Each folder builds as its own static library:

- `src/Document`, `src/Rendering` → `oma_core`. The model (`VectorDocument`,
  `VectorPath`, `Paint`), `EditorSession` (every edit, selection, history and
  view state; it emits `changed()` and `documentChanged()`), `DocumentHistory`,
  `DocumentCodec` (the JSON shared by `.omai` files and the clipboard),
  `PathOperations` (shapes, the boolean operations, offset, simplify),
  `ImageTrace`, and `VectorRenderer`, which the canvas and every export draw
  through.
- `src/IO` → `oma_io`. `ProjectStore` (`.omai`), `SvgImporter` (vendored
  nanosvg in `third_party/`), `SvgExporter`, `DocumentExporter` (PDF, PNG,
  JPEG), `ImageImporter` and `ScreenExport` (Export for Screens: artboards and
  export assets, a batch of scales and formats). Errors are thrown as
  `FileError`.
- `src/Cloud` → `oma_cloud`. Cloud storage through rclone
  (docs/CLOUD-STORAGE.md): `CloudStorage` runs it, `CloudLocation` is
  `remote:path` plus the cache, `CloudUploader` uploads in the background with
  conflict checks, and `CloudProviders` is the Connect list. Tests use the fake
  in `tests/Cloud/FakeRclone.cpp`; `CloudRcloneTests` runs the real rclone on a
  throwaway config and skips without it.
- `src/Agent` → `oma_agent`. The agent socket, CLI and MCP bridge
  (docs/AI-DESIGN.md), plus the desktop-wide commands in docs/OS-SUITE.md:
  `Cli` dispatches every GUI-less command, `Island` keeps the island's mode
  file and `omastrator island …`, `StatusStream` is `omastrator status
  --follow`, `Capture` runs hyprpicker, slurp, grim and wl-paste, `Setup` is
  `omastrator setup`, `Dictation` is push-to-talk (normalising, the grammar,
  Heard), `Vocabulary` is dictation's word list, `Hyprland` reads windows,
  monitors and the pointer from Hyprland's socket and runs dispatchers, and
  `DesignCli` is `omastrator design`, `desk` and `daemon`.
- `shell/` → the omarchy-shell plugins, QML: `omastrator.island` (the island),
  `omastrator.ai` (the tray light) and `omastrator-ui` (what they share).
  Setup copies them to `~/.config/omarchy/plugins/`. To try a change without
  touching the user's shell, run a throwaway `quickshell -p` config that loads
  the plugin, with `OMASTRATOR_SOCKET` and `OMASTRATOR_RUNTIME_DIR` pointed at a
  temporary folder.
- `src/Live` → `oma_live`. Live web editing (docs/OS-SUITE.md): an in-tree
  WebSocket client and the DevTools Protocol (`WebSocket`, `Cdp`), Chromium in
  Omastrator's own profile (`Browser`), dev servers and a static server,
  `ProjectRegistry`, `TokenSet` snapping, `LiveSession`, `EditSets` (edits to
  sites that aren't yours, kept per origin), write-back
  (`WriteBack`, `AgentWork`), and Deploy (`Deploy`, `DeployJob`, `History`,
  with GitHub through `gh`). Browser View's Chromium (docs/BROWSER-VIEW.md) is
  `BrowserPool` (its own thread, tabs, idle stop, the cap) and `Breakpoints`
  (the widths a site's stylesheets name). The page overlay
  is `overlay.js`, compiled in through `cmake/OverlayScript.h.in`. Headless
  tests run the fixtures in `tests/Live/fixtures` and skip without Chromium.
- `src/Anywhere` → `oma_anywhere`. Design mode everywhere (docs/ANYWHERE.md):
  `DesignMode` (on and off with the island's Design mode, hover, Alt distances),
  `DesktopSource` (Hyprland, AT-SPI and grim; tests use
  `tests/Anywhere/FakeDesktop.h`), `Inspect` (the web inspector script, the
  AT-SPI helper, distances), `Overlays` (`overlays.omai`, a layer per surface),
  `Desk` (frames), `Bar` (actions and suggestions), `Lift` (a surface's UI as
  vectors: `LiftScript.h` walks the DOM, `Lift+Screen` reads the AT-SPI tree or
  traces, `LiftJob` runs it in the background, `LiftDiff` maps changed lifted
  page vectors back to page edits; tests fake the tree through
  `FakeDesktop::trees` or `OMASTRATOR_ATSPI_TREE`) and `AnywhereSettings`
  (onboarding, destinations, in `anywhere.json`). The app side is
  `src/UI/DesignController` behind the `design` method. The overlay itself is
  `shell/omastrator.island/Overlay.qml`; its decisions are in
  `OverlayLogic.js`, which `ShellPluginTests` runs in a `QJSEngine`.
- `src/System` → `oma_system`. Design systems (docs/DESIGN-SYSTEMS.md):
  `TokenFiles` (W3C tokens.json, Tailwind v4 and v3, CSS variables),
  `ProjectCode`, `Library` (the global library), `SiteExtract`,
  `OmarchyThemes` and `SyncPlan`; phase 4's `DesktopLook` (Omarchy's gaps,
  borders, bar, font, wallpaper and colours), `AppStyle` (GTK CSS, qt6ct and
  Qt stylesheets) and `ConfigBackup` (backups and Revert), with the app side in
  `UI/DesignController+Look.cpp` and `UI/DesktopLookPanel`; their tests build a
  fake Omarchy desktop in a temporary HOME (`tests/System/DesktopFixtures.h`).
  Every push or pull, and every write to the desktop's config, is a `SyncPlan` that only
  `UI/SyncConfirmDialog` can confirm; tests answer it with
  `SyncConfirmDialog::setResponder`. The model is `Document/DesignTokens`,
  `Document/Components` and `EditorSession+System.cpp`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iretonsean/Omastrator](https://github.com/iretonsean/Omastrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
