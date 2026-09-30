---
trigger: always_on
description: This is the generic app builder/compiler. Scope is catalog data, not a separate installer. Keep the legacy CLI compatible.
---

# Makepad Builder

This is the generic app builder/compiler. Scope is catalog data, not a separate installer. Keep the legacy CLI compatible.

## Architecture and validation

- Platform dependency installation, network requests, extraction and source checkout belong on the persistent setup worker. The UI only submits bounded commands and receives progress/results.
- One folder layout on every platform: the installation folder people see holds only the entry point (`makepad-builder.bat` and `makepad-builder.exe` on Windows, the `makepad` script on macOS/Linux) and the built apps (`<binary>.exe`; the `<binary>` commands and `<Title>.app` bundles); everything else the Builder keeps (records, sources, toolchains, caches, target, logs, agent files, Unix `<binary>.bin`) is in its `builder/` subfolder (`STATE_DIR`; inside the Rust Builder `root` is that subfolder and `home_of(root)` the installation folder). Both Builders move an earlier flat folder's entries into `builder/` on start (`migrate_legacy_layout`, `migrate_layout`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makepad/makepad](https://github.com/makepad/makepad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
