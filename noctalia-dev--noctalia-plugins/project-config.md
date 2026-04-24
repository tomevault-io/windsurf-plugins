---
trigger: always_on
description: Guidelines for AI tools contributing to the Noctalia Plugins repository. **Study the official plugins before writing code** — especially `hello-world` (minimal reference) and `timer` (complex example with shared state). Official plugins have `"official": true` in their manifest.
---

# AI-Assisted Development Guidelines

Guidelines for AI tools contributing to the Noctalia Plugins repository. **Study the official plugins before writing code** — especially `hello-world` (minimal reference) and `timer` (complex example with shared state). Official plugins have `"official": true` in their manifest.

## Plugin API

Every plugin component receives a `pluginApi` property. This is the core interface:

```qml
Item {
  property var pluginApi: null

  // Settings access pattern — always use this fallback chain
  property var cfg: pluginApi?.pluginSettings || ({})
  property var defaults: pluginApi?.manifest?.metadata?.defaultSettings || ({})
  property string message: cfg.message ?? defaults.message ?? "fallback"
}
```

**Key pluginApi members:**
- `pluginSettings` — mutable settings object, persisted via `saveSettings()`
- `manifest` — read-only manifest.json data
- `pluginId`, `pluginDir` — plugin identity and path
- `mainInstance` — reference to Main.qml instance (for shared state)
- `tr(key, interpolations)` — translate a key, e.g. `pluginApi?.tr("widget.label")`
- `trp(key, count, singular, plural)` — plural translation
- `openPanel(screen, widget)`, `togglePanel(screen, widget)` — panel control
- `saveSettings()` — persist `pluginSettings` to disk
- `withCurrentScreen(callback)` — get current screen in IPC handlers
- `panelOpenScreen` — the screen where this plugin's panel is open

## Entry Points

Only include the entry points your plugin uses. Available types:

| Entry Point | File | Purpose |
|---|---|---|
| `main` | Main.qml | Shared state, IPC handlers |
| `barWidget` | BarWidget.qml | Bar widget |
| `panel` | Panel.qml | Overlay panel |
| `controlCenterWidget` | ControlCenterWidget.qml | Control center button |
| `settings` | Settings.qml | Plugin settings UI |
| `desktopWidget` | DesktopWidget.qml | Draggable desktop widget |
| `desktopWidgetSettings` | DesktopWidgetSettings.qml | Desktop widget settings |
| `launcherProvider` | LauncherProvider.qml | Launcher search provider |

## Component Patterns

### Main.qml — Shared State

```qml
import QtQuick
import Quickshell
import Quickshell.Io
import qs.Commons

Item {
  id: root
  property var pluginApi: null

  // Shared state accessible from other components via pluginApi.mainInstance
  property bool isActive: false

  // IPC handler for CLI control (qs ipc call plugin:my-plugin commandName)
  IpcHandler {
    target: "plugin:my-plugin"

    function toggle() {
      if (pluginApi) {
        pluginApi.withCurrentScreen(screen => {
          pluginApi.togglePanel(screen);
        });
      }
    }
  }
}
```

### BarWidget.qml

```qml
import Quickshell
import qs.Commons
import qs.Services.UI
import qs.Widgets

Item {
  id: root

  // Injected properties
  property var pluginApi: null
  property ShellScreen screen
  property string widgetId: ""
  property string section: ""
  property int sectionWidgetIndex: -1
  property int sectionWidgetsCount: 0

  // Settings
  property var cfg: pluginApi?.pluginSettings || ({})
  property var defaults: pluginApi?.manifest?.metadata?.defaultSettings || ({})

  // Bar layout awareness
  readonly property string barPosition: Settings.getBarPositionForScreen(screen?.name)
  readonly property bool isVertical: barPosition === "left" || barPosition === "right"
  readonly property real capsuleHeight: Style.getCapsuleHeightForScreen(screen?.name)

  implicitWidth: isVertical ? capsuleHeight : contentWidth
  implicitHeight: isVertical ? contentHeight : capsuleHeight

  // Context menu (right-click)
  NPopupContextMenu {
    id: contextMenu
    model: [
      { "label": pluginApi?.tr("menu.settings"), "action": "settings", "icon": "settings" }
    ]
    onTriggered: action => {
      contextMenu.close();
      PanelService.closeContextMenu(screen);
      if (action === "settings") {
        BarService.openPluginSettings(screen, pluginApi.manifest);
      }
    }
  }

  MouseArea {
    anchors.fill: parent
    acceptedButtons: Qt.LeftButton | Qt.RightButton
    onClicked: mouse => {
      if (mouse.button === Qt.LeftButton) {
        if (pluginApi) pluginApi.togglePanel(root.screen, root);
      } else if (mouse.button === Qt.RightButton) {
        PanelService.showContextMenu(contextMenu, root, screen);
      }
    }
  }
}
```

### Panel.qml

```qml
import QtQuick
import QtQuick.Layouts
import qs.Commons
import qs.Services.UI
import qs.Widgets

Item {
  id: root
  property var pluginApi: null

  // Required for background rendering
  readonly property var geometryPlaceholder: panelContainer

  // Panel dimensions (always scale with uiScaleRatio)
  property real contentPreferredWidth: 400 * Style.uiScaleRatio
  property real contentPreferredHeight: 500 * Style.uiScaleRatio

  // Enable panel attach/detach UI
  readonly property bool allowAttach: true

  anchors.fill: parent

  Rectangle {
    id: panelContainer
    anchors.fill: parent
    color: "transparent"

    ColumnLayout {
      anchors.fill: parent
      anchors.margins: Style.marginL
      spacing: Style.marginL


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noctalia-dev/noctalia-plugins](https://github.com/noctalia-dev/noctalia-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
