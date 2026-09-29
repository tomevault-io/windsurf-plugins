---
trigger: always_on
description: A click-drag window tiler for KDE Plasma 6 / KWin on Wayland. Hold a global
---

# VibeTiles

A click-drag window tiler for KDE Plasma 6 / KWin on Wayland. Hold a global
shortcut (default Meta+Alt+D), a fullscreen (or compact) grid overlay
appears, drag a rectangle across grid cells, release, and the
previously-active window snaps to that region.

Ships as a **declarative KWin script** (`kwinscript/`) — no compiler, no
Qt6/KF6 dev headers, just symlink one directory into
`~/.local/share/kwin/scripts/` and enable it. An earlier standalone Qt6/KF6
daemon was deleted in `baad1e7`; some code comments still cite its
`main.cpp` line numbers as the origin of a ported algorithm (historical
attributions only, that file is gone).

## Architecture

Everything runs inside `kwin_wayland` via Plasma 6's declarative KWin script
API (`X-Plasma-API: declarativescript`) — a full `QQmlEngine` with
privileged, synchronous access to `Workspace.*`. No D-Bus, no daemon.

- **`kwinscript/metadata.json`** — KPackage metadata. `KPlugin.Id` /
  `X-KDE-PluginKeyword` is both the kwinrc config-group key
  (`[Script-<id>]`) and the cache key for KWin's compiled-QML cache — see
  "Deploy / reload".
- **`kwinscript/contents/ui/main.qml`** — the entire overlay: grid,
  drag/snap, compact mode, title bar, multi-monitor picker,
  overlap-resize/relocate on commit, hot corners, drag-triggered
  activation, auto-trigger-on-drag, linked resize, expand-to-fill. Root is
  `PlasmaCore.Dialog` (a plain `QtQuick Window` gets silently swallowed by
  the script host, confirmed live).
- **`kwinscript/contents/ui/components/Shortcuts.qml`** — `ShortcutHandler`s
  for the two global shortcuts, user-rebindable from System Settings →
  Shortcuts.
- **`kwinscript/contents/config/main.xml`** — kcfg schema, auto-bound into
  `~/.config/kwinrc`.
- **`kwinscript/contents/ui/config.ui`** — Qt Widgets settings form (Qt
  Designer XML), rendered by KWin's built-in script-config dialog
  (`kcm_kwin4_genericscripted`) — no compiled KCM needed.

## Deploy / reload

- **`./install.sh`** — first-time setup: symlinks `kwinscript/` into
  `~/.local/share/kwin/scripts/<id>`, enables it, reconfigures KWin.
  Idempotent.
- **`./bump.sh`** — run after **every** `main.qml` edit. KWin caches
  compiled QML per plugin ID, so editing in place and reconfiguring is not
  enough. Bumps `metadata.json` to the next `vibetiles<N>`, migrates every
  kwinrc key forward, disables + unloads the old ID, verifies the new one
  loads. Check `KPlugin.Id` in `metadata.json` for the current live value.
  The live id comes from the **symlink name**, which can drift from the id in
  `metadata.json` (a fresh clone over an existing install does this). The
  rewrite therefore substitutes any `vibetiles<N>` it finds, and hard-fails if
  the file doesn't declare the new id afterwards — the symptom of that drift is
  the config dialog erroring with "could not locate package metadata".
- **`./build.sh`** — produces `vibetiles.kwinscript`, a release bundle
  under the canonical (non-numeric) id `vibetiles`, for "Install from
  File...". Doesn't touch the numbered dev install. **Don't have both
  enabled at once** — they'd register the same-named `ShortcutHandler` and
  race for the Meta+Alt+D grab. Disable the dev id first.
- **`config.ui` / `main.xml` changes** take effect immediately — just
  reopen "Configure...". `main.qml` changes need `./bump.sh`.
- **`reconfigure()` alone is not reliable for picking up a freshly-created
  plugin id** — confirmed live: enabling a brand-new `vibetiles<N>` and
  calling `org.kde.KWin.reconfigure` left `isScriptLoaded()` false and the
  previous generation kept running unchanged, silently (no error, no
  journal entry — the symptom to recognise is "edited main.qml, ran
  bump.sh, but the behavior is unchanged"). Both scripts now call
  `org.kde.kwin.Scripting.loadDeclarativeScript(path, pluginId)` explicitly
  after enabling, which loads it immediately; it's idempotent (returns
  `-1`, no duplicate `ShortcutHandler` registration) if the id is already
  loaded, so it's safe to run on every bump/install. If `bump.sh`'s
  own `isScriptLoaded` check ever comes back false again, load it by hand:
  `qdbus-qt6 org.kde.KWin /Scripting org.kde.kwin.Scripting.loadDeclarativeScript
  ~/.local/share/kwin/scripts/<id>/contents/ui/main.qml <id>`.
- **Old generations keep running after `unloadScript`, and only a KWin restart
  evicts them** — confirmed live: after unloading and even *deleting the package*
  of a generation, its instance kept handling native drags and drawing its own
  overlay, reading its own (now-orphaned) `[Script-<oldId>]` kwinrc section. The
  newest generation owns the global shortcut grab, so the symptom is a split
  personality: the shortcut drives the new code while drag-triggered paths, hot
  corners and the auto-picker still come from an old one — with the old one's
  settings, which look "impossible" against the live config section. Diagnose by
  counting instances (`qdbus-qt6 org.kde.KWin | grep '^/Scripting/Script'`)
  against installed packages (`ls ~/.local/share/kwin/scripts/`); more instances
  than packages means stale generations. `isScriptLoaded` is useless here — it
  tracks the kwinrc *enabled flag*, not the runtime, and returns true for ids
  whose package is gone. Clean up with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ivan-Malinovski/vibetiles](https://github.com/Ivan-Malinovski/vibetiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
