---
trigger: always_on
description: > Compatible with **GitHub Copilot** and **Cursor** agents.
---

# Copilot / Cursor instructions (Scrypted Advanced Notifier)

> Compatible with **GitHub Copilot** and **Cursor** agents.

**Language:** Use **English only** for UI strings, comments, commit messages, and documentation (plugin and frontend).

## Big picture
- This repo is a Scrypted plugin. The main entrypoint is [src/main.ts](src/main.ts) and it implements multiple Scrypted interfaces (Settings/DeviceProvider/MixinProvider/HttpRequestHandler/VideoClips/PushHandler).
- Most “real” device behavior lives in mixins:
  - Camera logic in [src/cameraMixin.ts](src/cameraMixin.ts)
  - Notifier logic in [src/notifierMixin.ts](src/notifierMixin.ts)
  - Sensors in [src/sensorMixin.ts](src/sensorMixin.ts) (if you change sensor flows, update MQTT discovery accordingly)
- MQTT + Home Assistant autodiscovery are centralized in [src/mqtt-utils.ts](src/mqtt-utils.ts). If you add/remove a published value, update both discovery + state publishing.
- Shared types/constants/helpers are in [src/utils.ts](src/utils.ts) (rule types, activation sources, IDs, limits like `MAX_RPC_OBJECTS_PER_CAMERA`, helper utilities).

## Key flows to follow
- **Settings pattern:** plugin + mixins use `StorageSettingsDict` + `StorageSettings` and often rebuild settings dynamically via `convertSettingsToStorageSettings` (see [src/notifierMixin.ts](src/notifierMixin.ts) and [src/cameraMixin.ts](src/cameraMixin.ts)). When adding a setting, ensure:
  - it has a stable key in the relevant `...SettingKey` union
  - `refreshSettings()` hides/updates dependent settings correctly
- **MQTT entities:** topics, discovery IDs, and entity naming conventions are owned by `idPrefix` and helpers in [src/mqtt-utils.ts](src/mqtt-utils.ts). Prefer extending existing entity builders over inventing new topic shapes.
- **Event storage:** detection events are stored as per-day JSON DB files via `node-json-db` in [src/db.ts](src/db.ts). If you change event schema (`DbDetectionEvent`), update writers/readers and any cleanup/retention logic.
- **Images/media:** snapshots/clips/GIFs are generated and post-processed across camera mixin + drawing utilities. Image filtering uses ffmpeg-based helpers (e.g. `ffmpegFilterImageBuffer`) and post-processing options in [src/utils.ts](src/utils.ts) / [src/drawingUtils.ts](src/drawingUtils.ts).

## Repo-specific conventions
- Device IDs / native IDs are centralized in [src/utils.ts](src/utils.ts) (`NOTIFIER_NATIVE_ID`, `CAMERA_NATIVE_ID`, etc). Reuse these constants.
- The plugin relies on sibling local packages (not published deps), e.g. `../../scrypted-apocaliss-base/...` and `../../scrypted-frigate-bridge/...` imports. Avoid refactors that break those paths unless you also update the mono-repo layout.
- Mixins store runtime state in plugin-owned maps (e.g. `currentCameraMixinsMap`, `currentNotifierMixinsMap` in [src/main.ts](src/main.ts)); when creating/releasing devices, keep those maps consistent.

## Dev workflows (what to run)
- Build bundle: `npm run build` (runs `scrypted-webpack`). See [package.json](package.json).
- Deploy + debug from VS Code: run the task `scrypted: deploy+debug` (it runs `npm run scrypted-vscode-launch`).
- Production build used for publish: `npm run prepublishOnly`.

## Editing tips (to avoid regressions)
- Prefer small, targeted changes in the relevant layer:
  - rule evaluation/notification behavior → [src/cameraMixin.ts](src/cameraMixin.ts)
  - notifier payload manipulation/translations/AI toggles → [src/notifierMixin.ts](src/notifierMixin.ts)
  - MQTT entity add/remove → [src/mqtt-utils.ts](src/mqtt-utils.ts)
- Keep defaults + “immediate” settings behavior consistent with existing settings (many toggles are `immediate: true`).
- Watch for load/size constraints: this plugin enforces soft/hard limits for RPC objects in [src/utils.ts](src/utils.ts).

## Quick checklists

### When you add/change an MQTT entity
- Start in [src/mqtt-utils.ts](src/mqtt-utils.ts): extend existing entity builders and keep `idPrefix` naming consistent.
- Update **both**: Home Assistant autodiscovery config (config topics) and state publishing (state/image topics). Don’t add “publish-only” values.
- If the entity is per-camera/per-rule, ensure the camera mixin publishes it (see [src/cameraMixin.ts](src/cameraMixin.ts)).
- If you change entity IDs/names, expect users may need to clear retained discovery/state topics; the plugin also exposes a “Reset all MQTT topics” action (see `mqttResetAllTopics` in [src/main.ts](src/main.ts)).
- **Interface-conditional entities** (e.g. Battery, Charger, Online, Sleep): define the entity in `getBasicMqttEntities()`, then guard inclusion with `device.interfaces.includes(ScryptedInterface.XXX)` in discovery (`getCameraMqttEntitiesForDiscovery`), initial state (`buildCameraInitialEntityStates`), publishing (`publishCameraValues`), and topic-clear (`resetAllPluginMqttTopicsAndRediscover`).
- **Plugin-level diagnostic entities** (e.g. memory, RPC, NVR storage health): define in `getBasicMqttEntities()`, add to `setupPluginAutodiscovery()`, and publish in `publishPluginValues()`. Use `entityCategory: ‘diagnostic’`. For conditional plugin entities, pass a flag (e.g. `hasNvrStoragePaths`) to control discovery inclusion.

### When you add/change a setting or rule option

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apocaliss92/scrypted-advanced-notifier](https://github.com/apocaliss92/scrypted-advanced-notifier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
