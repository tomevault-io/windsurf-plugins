---
trigger: always_on
description: PlasmaZones: window tiling + zone management for KDE Plasma. Qt6, KF6, Kirigami, C++20, Wayland-only.
---

# PlasmaZones — Claude Code Configuration
# KDE/Plasma Window Tiling — Qt6/C++20/QML/Kirigami

## Project
PlasmaZones: window tiling + zone management for KDE Plasma. Qt6, KF6, Kirigami, C++20, Wayland-only.

## Behavioral Rules (Always Enforced)
- NEVER question or doubt what the user says they did (installed, restarted, tested, etc.) — trust them and focus on the code
- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary; prefer editing existing files
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested
- NEVER save working files, text/mds, or tests to the root folder
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files
- ALWAYS run tests after making code changes
- ALWAYS verify build succeeds before committing
- NEVER run `cmake --install` or `sudo` — the user handles installation
- NEVER use temporary workarounds, TODOs, "for now" hacks, or deferred fixes — solve the root cause properly the first time

## File Organization
- NEVER save to root folder — use the directories below
- Use `/src` for source code files
- Use `/tests` for test files
- Use `/docs` for documentation and markdown files
- Use `/config` for configuration files
- Use `/scripts` for utility scripts
- Use `/examples` for example code

## License
- SPDX headers on ALL files: `// SPDX-FileCopyrightText: 2026 fuddlesworth` / `// SPDX-License-Identifier: GPL-3.0-or-later`
- `#pragma once` for C++ headers

## C++ Style

### Naming
- Classes: `PascalCase` — Functions: `camelCase` — Members: `m_camelCase`
- Struct POD fields: `camelCase` (no prefix) — Constants: `PascalCase` (class) / `UPPER_SNAKE` (global)
- Signals: past tense (`layoutChanged`) — Slots: action verb (`saveLayout`)

### Core Rules
- C++20, `namespace PlasmaZones`, `explicit` single-param constructors, `override` on virtuals
- `Q_OBJECT`, `Q_EMIT`, `Q_SIGNALS:`, `Q_SLOTS:`, `Q_PROPERTY` with READ/WRITE/NOTIFY
- Only emit signals when value actually changes
- Parent-based ownership for QObjects; `std::unique_ptr`/`QPointer` otherwise; never manual delete
- Forward declare in headers; group includes: own header → project → KDE → Qt
- `PLASMAZONES_EXPORT` on public API classes
- Keep files under 800 lines
- Input validation at system boundaries

### Qt6 String Literals (CRITICAL)
- `QLatin1String()` for JSON keys and string comparisons
- `QStringLiteral()` for constants, MIME types, paths
- NEVER use raw `"string"` with QString/QJsonObject (deleted constructor in Qt6)

### QUuid Convention
- `toString()` (with braces) everywhere — EXCEPT filesystem paths use `WithoutBraces`

## QML Style
- Qt Quick 6, Kirigami, QtQuick.Controls/Layouts
- Components/files: `PascalCase.qml` — IDs/props/functions: `camelCase`
- Prefer bindings over JS assignments; typed properties over `var`; `required property` for mandatory props
- Use `Kirigami.Theme` for colors, `Kirigami.Units` for spacing — never hardcode
- Zone IDs (QUuid), never indices — `Accessible.name` on interactive elements

## Architecture
- Service-oriented: `ILayoutService`, `ZoneManager`, `SnappingService`; DI via constructor
- Business logic in C++, UI in QML; controllers bridge via `Q_PROPERTY`
- Zone IDs everywhere, never indices
- JSON persistence in `~/.local/share/plasmazones/layouts/` with relative geometry (0.0–1.0)
- Wayland only (custom layer-shell QPA plugin for overlays); XWayland windows handled within Wayland session

## i18n
- C++: `PzI18n::tr()` — NEVER `KLocalizedString`/`i18n()`/`i18nc()` in C++
- QML: `i18n()` / `i18nc()` (via `PzLocalizedContext`)
- Extract: `cmake --build build --target update-ts`

## Settings

### Architecture
- `ISettings` interface → `Settings` class → `IConfigBackend` (pluggable, default: JSON → `~/.config/plasmazones/config.json`)
- `ConfigDefaults` for all default values; `.kcfg` schema is KCM-only
- Editor settings: separate, in `EditorController` (separate process)

### Adding a Setting
1. `configdefaults.h` — static default accessor + `xxxKey()` accessor for the config key string
2. `interfaces.h` — signal in ISettings
3. `settings.h` — Q_PROPERTY + getter + setter + member
4. `settings.cpp` — setter (check changed, emit), load/save/reset using `ConfigDefaults::xxx()`

### Config Key Strings
- ALL config group names and key strings MUST use `ConfigDefaults::` accessors — never inline `QStringLiteral("...")`
- Group accessors: `ConfigDefaults::snappingBehaviorGroup()`, key accessors: `ConfigDefaults::enabledKey()`
- v2 groups use dot-paths mirroring the UI hierarchy (e.g. `"Snapping.Behavior.ZoneSpan"`)
- Key accessors are generic (e.g. `enabledKey()`, `triggersKey()`) — the group context disambiguates

### No Ad-Hoc Backwards Compatibility
- NEVER add migration code for individual renamed keys or deprecated settings within the same schema version
- If a setting is renamed or restructured within a version, just use the new key — old values are silently dropped
- Users get the default value if their config doesn't have the current key; this is acceptable
- NEVER write empty strings to "clear" obsolete keys on save
- NEVER read from a fallback/legacy group when the primary group is empty

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fuddlesworth/PlasmaZones](https://github.com/fuddlesworth/PlasmaZones) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
