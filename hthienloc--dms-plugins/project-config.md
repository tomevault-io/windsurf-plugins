---
trigger: always_on
description: This repository is a personal collection of custom and community plugins for [DankMaterialShell (DMS)](https://github.com/AvengeMedia/DankMaterialShell), maintained by [@hthienloc](https://github.com/hthienloc). These guidelines help human contributors and AI agents build, maintain, and contribute plugins safely and consistently.
---

# Agent Guidelines — dms-plugins

This repository is a personal collection of custom and community plugins for [DankMaterialShell (DMS)](https://github.com/AvengeMedia/DankMaterialShell), maintained by [@hthienloc](https://github.com/hthienloc). These guidelines help human contributors and AI agents build, maintain, and contribute plugins safely and consistently.

---

## 1. Project Overview & Architecture

### Monorepo Structure
Each subdirectory in this repository represents an independent plugin:
```
dms-plugins/
├── <pluginName>/             # Individual self-contained plugin
│   ├── plugin.json           # Required: Plugin manifest & metadata
│   ├── <MainComponent>.qml   # Required: Plugin entrypoint
│   ├── <Plugin>Settings.qml  # Optional: Settings card in DMS Settings
│   ├── shared/               # Optional: Shared DMS utility components
│   ├── docs/                 # Optional: Specifications & architecture notes
│   ├── translations/         # Optional: Localized UI strings (.json)
│   └── README.md             # Required: Plugin documentation & shortcuts
├── shared/                   # Canonical source for shared UI components
├── scripts/                  # Repository maintenance & tooling scripts
└── README.md                 # Root catalog of all plugins
```

### Plugin Types & Type Selection Criteria
Every plugin declares its surface type in `plugin.json`. Choose the simplest, most appropriate type based strictly on the plugin's primary responsibility:

| Type | Base Component | Role & Primary Surface | When to Use | Examples in repo |
| :--- | :--- | :--- | :--- | :--- |
| `widget` | `PluginComponent` | DankBar pill + popout card / Control Center | Persistent bar presence needed for quick status readout, toggling state, or interacting via popout | `caffeine`, `timer`, `ipIndicator`, `hydrate`, `breathing`, `lutrisLauncher`, `bongoCat`, `ambientSound`, `handMirror`, `hiddenBar`, `mediaDownloader`, `ocrScanner` |
| `daemon` | `PluginComponent` | Background service or standalone on-demand modal/overlay | Headless background event/timer listener, or on-demand modal/overlay triggered via shortcut/IPC without cluttering the bar | `emojiPicker` |
| `launcher` | `Item` | DMS Launcher search result provider | Actionable search results, conversions, or query integrations inside DMS Launcher (`trigger` based) | `kaomojiPicker` |
| `desktop` | `DesktopPluginComponent` | Floating desktop widget | Persistent or draggable widget placed directly onto the desktop workspace | `activateLinux` |
| `composite` | Multiple entry components | Multi-surface integration | Complex plugins where distinct surfaces (e.g. daemon + Control Center toggle / bar widget) are strictly interdependent | `screenkey`, `typingSounds`, `takeABreak`, `quickCapture`, `desktopWidgetToggle`, `niriDS` |

### Type Selection Principles: Avoid Over-Engineering
- **Single Responsibility First:** Match the type strictly to what the plugin actually does. If a plugin is an on-demand modal picker triggered by hotkey (like `emojiPicker`), declare it as `daemon`; do not add a bar widget or launcher search unless the core UX demands it.
- **Do Not Default to `composite`:** `composite` introduces multiple entry points and significantly complicates state management, testing, and memory footprint. Never use `composite` when a single surface (`daemon` or `widget`) suffices.
- **Keep the Shell Clean:** Avoid adding bar widgets for tools used occasionally. Reserve the bar for real-time monitoring and frequent toggles. Prefer `daemon` with IPC toggle shortcuts for on-demand tools.

---

## 2. Agent Scope & Blast Radius Rules

- **Plugin Isolation:**
  - Every pull request, commit, and change must target exactly one plugin folder (`<pluginName>/`).
  - Do not make cross-cutting edits across multiple plugins in the same change unless explicitly instructed (e.g., repository-wide migration or dependency upgrade).
- **Protected Files & Scope Boundaries:**
  - **Translations (`<pluginName>/translations/`):** Do not modify translation JSON files manually unless explicitly instructed. Keep strings in `I18n.trFor()` calls stable.
  - **Manifests (`plugin.json`):** Preserve required fields (`id`, `name`, `version`, `entryPoint`, `type`, `capabilities`). Version bumps should match the nature of changes.
  - **Symlink Awareness:** Testing is conducted against local DMS symlinks (`~/.config/DankMaterialShell/plugins/<pluginName>`). Never create circular symlinks or alter the symlink targets outside this repository.

---

## 3. QML & DMS Code Standards

### Pragmas & Modern QML
- Use `pragma ComponentBehavior: Bound` in modern QML components when appropriate for component binding safety.
- Import standard DMS modules:
  - `qs.Common` / `qs.Widgets` for standard UI widgets and styling.
  - `qs.Services` for compositor, clipboard, toast, and system services (`CompositorService`, `ToastService`, `DMSService`).
  - `qs.Modules.Plugins` for `PluginComponent` and `PluginService`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hthienloc/dms-plugins](https://github.com/hthienloc/dms-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
