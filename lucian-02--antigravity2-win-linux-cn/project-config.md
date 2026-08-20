---
trigger: always_on
description: These instructions apply to every AI agent and contributor working in this repository. The goal is to deliver natural, complete, and maintainable Simplified Chinese localization without breaking Antigravity behavior, user content, technical identifiers, or renderer performance.
---

# Antigravity Chinese Localization — Project Instructions

These instructions apply to every AI agent and contributor working in this repository. The goal is to deliver natural, complete, and maintainable Simplified Chinese localization without breaking Antigravity behavior, user content, technical identifiers, or renderer performance.

## 1. Current target and sources of truth

- The current localization target is **Antigravity v2.8.1**. The current version dictionary is `dicts/v2.8.1.json`; the version, date, support statement, and dictionary name documented in `README.md` must describe the same target.
- Treat the repository's actual files and executable behavior as authoritative. If this document, `README.md`, a dictionary filename, or an engine comment disagrees with the current client or implementation, report the mismatch and update only the files needed for the requested task.
- A version update must replace or deliberately migrate the previous version dictionary rather than leave multiple version dictionaries active by accident. The engine loads **every** `dicts/*.json` file, so retaining both old and new version files changes the effective dictionary. Before a version bump, compare against the official client source, audit collisions, update all current-version references, and do not change tags or release metadata unless explicitly requested.
- `README.md` is the user-facing description of supported platforms, installation, restoration, and current coverage. Keep technical claims there consistent with `install.sh`, `uninstall.sh`, the Windows batch launchers, and `localization_engine.js`; do not claim tooling, tests, paths, or compatibility that the repository does not provide.

## 2. Repository structure and ownership

- JSON files in `dicts/` are renderer-facing Simplified Chinese dictionaries. Before changing one, use `rg` across all dictionaries to find existing keys and translations. Reuse established wording and avoid duplicate, case-only, or conflicting entries.
- `localization_engine.js` has two responsibilities:
  1. build and inject the renderer translation engine, including dynamic text and React-fragment rules; and
  2. unpack/repack `app.asar` and patch packaged Electron surfaces such as `preload.js`, `menu.js`, `tray.js`, `loadingOverlay.js`, and updater-related files.
- Put fixed renderer text in the most appropriate `dicts/*.json` file. Fixed native menu, tray, loading, or updater text belongs in the corresponding narrowly scoped injection block in `localization_engine.js`, because those surfaces do not consume the renderer dictionary.
- `install.sh` and `双击安装中文汉化.bat` are the supported installation entry points. `uninstall.sh` and `双击卸载还原官方英文.bat` are the supported restoration entry points. `localization_engine.js` is side-effecting and is not a development test command.
- `.gitattributes` protects Windows batch files from Git text conversion. Treat the shipped `.bat` files as encoded release artifacts: do not reformat or rewrite them with ordinary UTF-8 tooling. When a batch-file change is explicitly requested, use the repository's `convert_to_gbk.ps1` workflow and verify the resulting encoding and line endings.
- Change only dictionaries, engine rules, installer scripts, documentation, or version references directly relevant to the requested outcome. Do not reorder large dictionaries, reformat unrelated files, add dependencies, or change release metadata speculatively.

## 3. Dictionary loading and matching semantics

- `loadDictionary()` reads all JSON dictionaries in lexicographic filename order. Keys are normalized by collapsing whitespace, trimming, and normalizing curly quotes. If two files contain the same normalized key, the later-loaded file silently overrides the earlier value.
- The renderer map also has a case-insensitive fallback. Therefore, keys that differ only by case are conflicts even when JSON itself accepts them. Audit normalized duplicates, case-insensitive duplicates, and source-equals-translation no-op entries before handoff.
- Invalid JSON is skipped by the loader rather than surfaced to the client. Parsing every dictionary independently is mandatory; never assume that a successful install proves every dictionary was loaded.
- Dictionary keys must match actual client source text after the engine's normalization. Do not add screenshot-specific counts, speculative variants, or broad single-word substitutions just to make a visible example translate.
- Exact dictionary matching is preferred for short labels and complete fixed sentences. The engine may use case-insensitive fallback and long-entry substring replacement for entries longer than 20 characters, so long keys still need enough context to be safe inside ordinary UI text.
- Fragment-only keys such as a suffix, article, or grammar connector are allowed only when the client is confirmed to render that exact fragment as an independent UI node and the key cannot affect unrelated content. Otherwise, handle the complete dynamic sentence in the engine.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lucian-02/antigravity2-win-linux-cn](https://github.com/Lucian-02/antigravity2-win-linux-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
