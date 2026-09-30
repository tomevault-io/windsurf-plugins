---
trigger: always_on
description: <!-- Parent: ../../AGENTS.md -->
---

<!-- Parent: ../../AGENTS.md -->
<!-- Generated: 2026-09-26 | Updated: 2026-09-27 -->

# obsidian-plugin

## Purpose

The Obsidian plugin: a `TextFileView` that puts `<erd-editor>` in a tab for `.erd` / `.vuerd` files, and for `.erd.json` / `.vuerd.json` through a wrapped `WorkspaceLeaf.openFile`. Each vault window also serves the document hub `agent-hub` specifies, over `agent-hub-host`, so a coding agent running `@dineug/erd-editor-mcp` edits the diagrams open in that window live, as it does in VS Code; the Coding agents setting turns it off. The theme (appearance, gray and accent color) is a per-vault setting the editor's own theme builder changes too, as in VS Code. `dist/` (`main.js`, `manifest.json`, `styles.css`) is the whole plugin folder. It is released from `dineug/erd-editor-obsidian-plugin`, which carries this repository as a submodule, keeps a copy of `manifest.json` at its root for Obsidian's version check, and attaches `dist/` to a GitHub release tagged with the manifest version.

## Key Files

| File | Description |
| --- | --- |
| `src/main.ts` | The plugin: view and extension registration, the `openFile` wrapper, the export callback, the create command, the registry from load on, the theme (`ThemeHost`, `setTheme`, `applyTheme` over every ERD leaf, the `css-change` handler), the one `quit` and one `pagehide` handler, which write every ERD tab's unsaved value (`saveBeforeExit`) and then release the hub, the hub (`startHub`, none in a Flatpak), the vault adapter the hub opens and creates files through, the settings tab |
| `src/ErdView.ts` | The tab: load, the replica worker that serializes every save, the live relay between tabs of one file, the seeding of a tab opened beside others, `saveBeforeExit` (the synchronous write of the unsaved value on quit and pagehide, a quit task only while a write is under way), the read-only fallback, the theme builder's `changePresetTheme`, the `scope` that keeps Obsidian's hotkeys off the editor's shortcuts; implements `HubTab` |
| `src/settings.ts` | `PluginSettings` (`data.json`), `DEFAULT_SETTINGS`, `readSettings` / `readTheme` (unknown values fall back), `resolveTheme` (auto against Obsidian's light or dark), `themeFromBuilder`, the `ThemeHost` a tab themes through |
| `src/tabSave.ts` | `viewData`, `currentValue`, `hasUnsavedValue`: what a tab hands Obsidian to save, as pure functions of `TabSaveState`; `exitSave`, whether the window going writes that value now, leaves it to a quit task or leaves the file as it is; `seedValue`, the text a waiting tab loads |
| `src/keys.ts` | `scopeKeysOf` / `toScopeKey`: the editor's tinykeys shortcuts as the modifiers and key `Scope.register` takes, each press once |
| `src/icon.ts` | `ERD_ICON` / `ERD_ICON_SVG`: the icon of the tab and of the New ERD menu item, the logo's two tables and their link in lines |
| `src/loadErdEditor.ts` | Requires `@dineug/erd-editor` once per window |
| `src/hub/registry.ts` | `DocumentRegistry`: the tabs of each file, the writer, readiness, relays, the content mirror, the quiet state, joins, `seedWhenQuiet`, the active document (`setActive`, `keepActive`), renames, the lock's documents, `shutdown` |
| `src/hub/handlers.ts` | `createDocumentHandler`: `listDocuments`, `openDocument`, `join`, `applyActions`, `leave`, `save`, `disconnect` over the registry and the vault |
| `src/hub/host.ts` | `HubSwitch` (the Coding agents setting with listeners), `createObsidianHost` (ide `obsidian`, the vault folder), `pidSandbox` (Flatpak, where the window starts no hub) |
| `src/hub/runtime.ts` | `createHubRuntime`: one `ManagedRuntime` of the registry's layers and `documentHubLayer`; `start`, `dispose`, `releaseSync` |
| `src/hub/lifecycle.ts` | `HubLifecycle`: start after an earlier instance of the window closed, stop in order (`documentClosed`, drain, dispose), `releaseSync` |
| `src/hub/types.ts` | `HubTab`, `HubVault`, `VaultFile`, `CreateOutcome` — what `main.ts` and `ErdView.ts` implement |
| `src/hub/imports.test.ts` | Holds `src/hub/` to no runtime import of `obsidian` and effect to its named entries |
| `src/__test-utils__/hub.ts` | `createHubHarness` (a registry and handler over a memory disk and a vault double), `FakeTab`, connection doubles, `servePeer` (the shipping `serveConnection` on a real socket) |
| `vite.config.ts` | CommonJS library build into `dist/`, `inlineUrlWorkers` with the shared `base64InlineWorkers`, `pluginFiles`, `noBrowserExternal`, `run.tasks.build` and `run.tasks.test` |
| `vitest.config.ts` | Node environment; coverage over `src/hub/**`, `src/keys.ts`, `src/settings.ts` and `src/tabSave.ts`, perFile 80% on all four metrics |
| `manifest.json` / `versions.json` | Obsidian's plugin metadata; the release repository copies both to its root |
| `styles.css` | The tab layout and the containment override |
| `e2e/smoke.mjs` | `pnpm --filter @dineug/erd-editor-obsidian-plugin smoke` (see Testing Requirements) |
| `e2e/mcp.mjs` | The smoke's MCP client: JSON-RPC lines over the real server's stdio |

## For AI Agents

### Working In This Directory


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dineug/erd-editor](https://github.com/dineug/erd-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
