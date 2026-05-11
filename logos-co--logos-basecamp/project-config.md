---
trigger: always_on
description: A Qt/QML desktop application with a plugin-based architecture. It uses Nix for builds and has an MCP-based QML inspector for UI automation.
---

# Logos Basecamp

A Qt/QML desktop application with a plugin-based architecture. It uses Nix for builds and has an MCP-based QML inspector for UI automation.

## Building & Running

```bash
# Build the app
nix build

# Run in dev mode (hot-reload QML, no disk cache)
./run-dev.sh

# Build + run directly
nix build && ./result/bin/LogosBasecamp
```

## Testing

```bash
# Smoke test (validates app starts without QML errors)
nix build .#smoke-test -L

# Build test framework (one-time, rebuilds when logos-qt-mcp changes)
nix build .#logos-qt-mcp -o result-mcp

# UI integration tests (app must be running first)
node tests/ui-tests.mjs

# UI integration tests headless (CI mode)
node tests/ui-tests.mjs --ci ./result/bin/LogosBasecamp

# Hermetic CI test via Nix
nix build .#integration-test -L
```

## App Structure

- **Sidebar** (left): Contains app plugin icons (top/middle) and system buttons at the bottom (Dashboard, Modules, Settings)
- **Plugins** appear as sidebar icons: `counter`, `counter_qml`, `package_manager_ui`, `webview_app`
- Plugins are loaded from `~/Library/Application Support/Logos/LogosBasecampDev/plugins/`
- Main UI is in `src/qml/`, with panels in `src/qml/panels/`

## C++ Architecture

The backend is split into four classes with a unidirectional dependency graph:

```
MainUIBackend (facade, QML-facing — owns the other three as Qt children)
    │
    ├─► CoreModuleManager    (wraps logos_core_* C API, stats polling)
    │       ▲
    │       │ (uses for all C API calls)
    ├─► UIPluginManager       (UI plugin widgets, app launcher, unload cascade)
    │       ▲
    │       │ (queries for installType / missing-deps / dependents;
    │       │  provides intersectWithLoaded / teardownUiPluginWidget)
    └─► PackageCoordinator    (package_manager IPC, install/uninstall/upgrade
                               orchestration, install & uninstall-cascade dialogs)
```

### MainUIBackend (`src/MainUIBackend.h/.cpp`)
Thin QML-facing facade. Holds only navigation state (`m_currentActiveSectionIndex`, `m_sections`). Every QML-visible slot/signal is a one-line delegation into one of the three managers. The `coreModules()` Q_PROPERTY is the one exception — it composes data from multiple managers (known list + stats from CoreModuleManager, installType from PackageCoordinator). The `cancelPendingAction(name)` slot fans out to both UIPluginManager and PackageCoordinator so the un-involved one no-ops.

### CoreModuleManager (`src/CoreModuleManager.h/.cpp`)
Single owner of the `logos_core_*` C API. Provides thin wrappers: `knownModules()`, `loadedModules()`, `loadModule()`, `unloadModule()`, `unloadModuleWithDependents()`, plus a stats timer that periodically queries `logos_core_get_module_stats`. Nothing else in the app calls the C API directly.

### UIPluginManager (`src/UIPluginManager.h/.cpp`)
Owns UI plugin widget lifecycle in-process: PluginLoader wiring, widget teardown, app launcher state, UI-plugin metadata cache (`m_uiPluginMetadata`) used for load dispatch. Runs the local *unload* cascade (no package_manager involvement). Queries PackageCoordinator for installType / missing-deps / dependents via accessor methods. Exposes `intersectWithLoaded(names)` + `teardownUiPluginWidget(name)` for PackageCoordinator to call during uninstall cascade.

- **Load/unload**: `loadUiModule`, `unloadUiModule`, `loadCoreModule`, `unloadCoreModule` — pre-flight dependency checks, then delegates to CoreModuleManager
- **Unload cascade**: `confirmUnloadCascade`, `cancelUnloadCascade` — single-slot `m_pendingUnload` drives the QML dialog
- **App launcher**: `activateApp`, `onAppLauncherClicked`, `onPluginWindowClosed`, `setCurrentVisibleApp`

### PackageCoordinator (`src/PackageCoordinator.h/.cpp`)
Owns every interaction with the `package_manager` LogosAPI module. (Named `PackageCoordinator` rather than `PackageManager` to avoid colliding with the SDK-generated `PackageManager` proxy class.) Event subscriptions, install/uninstall/upgrade IPC, the install-confirmation dialog, the uninstall-cascade dialog, plus the package-state caches (`m_installTypeByModule`, `m_missingDepsByModule`, `m_dependentsByModule`). Holds the gated-cascade pending slot for uninstall/upgrade ops.

- **Install from LGX**: `installPluginFromPath` → `inspectPackageAsync` → shows install-confirm dialog (fresh install or upgrade) → `confirmInstall()`/`cancelInstall()`
- **Gated uninstall/upgrade**: Subscribes to `package_manager` module's `beforeUninstall`/`beforeUpgrade` events, acks within 3s, shows cascade dialog, then confirms/cancels back to the module
- **Cascade confirmation**: `confirmUninstallCascade`, `cancelPendingAction` — drives cascade unload via CoreModuleManager + UIPluginManager, then hands back to the module
- **Metadata refresh**: `refresh()` triggers the full `getInstalledUiPlugins` + `getInstalledPackages` + per-package `resolveFlatDependencies/Dependents` chain; pushes UI metadata to UIPluginManager via `uiPluginsFetched` signal

### Construction & Destruction Order

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [logos-co/logos-basecamp](https://github.com/logos-co/logos-basecamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
